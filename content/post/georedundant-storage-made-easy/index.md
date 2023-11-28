
+++
author = "Michael Collins"
title = "Ceph's Stretch Mode: Georedundant Storage Made Easy!"
date = "2023-11-28"
description = "Georedundant storage made easy, with Cephs little known 'Stretch Mode' feature."
tags = [
    "storage",
    "ceph",
    "georedundancy",
    "stretch mode",
    "failure domains",
    "data replication",
    "data redundancy",
    "data integrity"
]
+++


## Geo-Redundant Storage Made Easy

---

Georedundant storage is hard, companies pay a lot of money for serious guarantees that their data will not only be replicated across multiple locations but that it'll be available to them when they need it. This is a problem that has been solved by the likes of Amazon, Google, and Microsoft, but it's not a problem that has been solved for the rest of us. Until now.

Ceph is a distributed storage system that is designed to be highly available and fault-tolerant. It spreads data across multiple 'nodes' (hosts) and disks, so that if one node or disk fails, that data is still available. Configuring Ceph to make data redundant across one site can be a challenge, but this is made even more difficult when you want to replicate data across multiple sites while ensuring reliability. A multi-site Ceph setup requires running multiple Ceph clusters in an active-active or active-passive configuration, it is significantly more complex than a single-site setup.

This is where Ceph's "stretch mode" feature comes in. If you only want to distribute data across 2 sites and you want to avoid the complexity of a multi-site setup, stretch mode is the happy middle ground you're looking for.


## Limits of a Ceph Cluster

---

CRUSH picks OSDs at random to write data to, it doesn't guarantee that data will be replicated across multiple sites. This is a problem that is difficult to solve with standard CRUSH rules, as the documentation states:

https://docs.ceph.com/en/latest/rados/operations/stretch-mode/#stretch-cluster-issues

blockquote a

> "For example, in a scenario in which there are two data centers named Data Center A and Data Center B, and the CRUSH rule targets three replicas and places a replica in each data center with a min_size of 2, the PG might go active with two replicas in Data Center A and zero replicas in Data Center B. In a situation of this kind, the loss of Data Center A means that the data is lost and Ceph will not be able to operate on it. This situation is surprisingly difficult to avoid using only standard CRUSH rules."

blockquote b

[^1]: "For example, in a scenario in which there are two data centers named Data Center A and Data Center B, and the CRUSH rule targets three replicas and places a replica in each data center with a min_size of 2, the PG might go active with two replicas in Data Center A and zero replicas in Data Center B. In a situation of this kind, the loss of Data Center A means that the data is lost and Ceph will not be able to operate on it. This situation is surprisingly difficult to avoid using only standard CRUSH rules."

blockquote c

<blockquote>
  "For example, in a scenario in which there are two data centers named Data Center A and Data Center B, and the CRUSH rule targets three replicas and places a replica in each data center with a min_size of 2, the PG might go active with two replicas in Data Center A and zero replicas in Data Center B. In a situation of this kind, the loss of Data Center A means that the data is lost and Ceph will not be able to operate on it. This situation is surprisingly difficult to avoid using only standard CRUSH rules."
</blockquote>

highlight

=="For example, in a scenario in which there are two data centers named Data Center A and Data Center B, and the CRUSH rule targets three replicas and places a replica in each data center with a min_size of 2, the PG might go active with two replicas in Data Center A and zero replicas in Data Center B. In a situation of this kind, the loss of Data Center A means that the data is lost and Ceph will not be able to operate on it. This situation is surprisingly difficult to avoid using only standard CRUSH rules."==

italic

__"For example, in a scenario in which there are two data centers named Data Center A and Data Center B, and the CRUSH rule targets three replicas and places a replica in each data center with a min_size of 2, the PG might go active with two replicas in Data Center A and zero replicas in Data Center B. In a situation of this kind, the loss of Data Center A means that the data is lost and Ceph will not be able to operate on it. This situation is surprisingly difficult to avoid using only standard CRUSH rules."__


## How Stretch Mode Works

---

The standard configuration for stretch mode is 2 datacenters with a roughly equal amount of OSDs in each. Each site holds 2 copies of the data with a total replication size of 4. New writes are written once to both sites before they are acknowledged by Ceph. A third site is used as a tiebreaker, which doesn't contain any OSDs. In the event that the network between the 2 sites is cut, the tiebreaker site will be used to determine which site is the primary site. When the network between the 2 sites is restored, the secondary site will be brought back online and the data will be resynced.

In the event that one datacenter is lost, the cluster will enter a special "degraded stretch mode" described here:

https://docs.ceph.com/en/latest/rados/operations/stretch-mode/#entering-stretch-mode

> "If all OSDs and monitors in one of the data centers become inaccessible at once, the surviving data center enters a “degraded stretch mode”. A warning will be issued, the min_size will be reduced to 1, and the cluster will be allowed to go active with the data in the single remaining site. The pool size does not change, so warnings will be generated that report that the pools are too small -- but a special stretch mode flag will prevent the OSDs from creating extra copies in the remaining data center. This means that the data center will keep only two copies, just as before."

![A preview of a cluster that's entered degraded stretch mode.](https://perthserverplus.com/images/degraded-stretch-mode.png#center)

_A cluster in degraded stretch mode that's lost one of its datacenters. It keeps serving data and allowing writes._


## The Benefits of Stretch Mode

---

- Cross-site data redundancy and high availability.
- It's easier to set up then a multi-site configuration, you don't need to run multiple Ceph clusters or configure zones, realms and endpoints.
- It's cheaper than a multi-site setup, as a multi-site setup requires running a duplicate amount of hardware in both locations to host 3 copies in each. While a stretch mode cluster only keeps 2 copies in each location.


## The Limitations of Stretch Mode

---

- It's only suitable for 2 sites, if you want to replicate data across more than 2 sites you'll need to set up a [multi-site cluster](https://docs.ceph.com/en/quincy/radosgw/multisite/).
- It's only suitable if your OSD devices are SSDs, as the latency of HDDs is too high to make this work.
- It's only suitable if your sites are close together, as the latency between sites needs to be low enough to make this work.
- Writes are slow, this is because Ceph needs to write an object to both sites before it can acknowledge the write.


Ceph also has a feature called "stretch mode" which allows you to replicate data across multiple datacenters without having to set up a full-blown multi-site cluster with multiple clusters and a lot of complexity.


## My Install Guide for Stretch Mode

I've published a guide for creating a stretch mode cluster, it can be used to help you examine if a stretch mode cluster is appropriate for your own use case:

https://github.com/PC-Admin/cephfs-stress-test/blob/main/stretch_mode_setup.md


## Testing Site Failure

I went to the extra effort of testing an abrupt site failure on this cluster, I did this by forcibly shutting off the VMs in datacenter a1. The filesystem became unresponsive for 33 seconds, and then it was back to normal.

I then tested a sudden isolation of the network between the 2 sites, where both could talk to the tiebreaker node but not to each other. I did this by quickly erecting firewall rules on the hosts in a1 and severing any existing connections. The filesystem became unresponsive for 47 seconds before returning to normal.

I've published the steps and results of these tests here:

https://github.com/PC-Admin/cephfs-stress-test/blob/main/stretch_mode_setup_tests.md

Stretch mode seems to be very resilient and allows clients to keep accessing their data pretty quickly in the event of a site failure.


## Conclusion

Stretch mode is a great feature that allows you to replicate data across multiple sites without having to set up a full-blown multi-site cluster with multiple clusters and a lot of added complexity. It'so cheaper than a multi-site setup, as a multi-site setup requires running a duplicate amount of hardware in both locations to host 3 copies in each. While a stretch mode cluster only keeps 2 copies in each location.

If you need assistance with your Ceph setup, or your interested in upgrading to a stretch mode cluster, please [contact me](https://perthserverplus.com/contact/) for a free consultation.
