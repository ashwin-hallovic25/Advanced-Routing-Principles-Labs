# EIGRP Path Selection and Metric Investigation

## Objective

Investigate how EIGRP selects routes and observe how changing interface
bandwidth affects the EIGRP metric and path selection.

## Topology

The lab uses four Cisco IOSv routers running EIGRP AS 100.

R1 is used as the main observation point.

![EIGRP Topology](screenshots/topology.png)

### Addressing

  Link / Interface   Network
  ------------------ ------------------
  R1--R2             `10.0.12.0/30`
  R1--R3             `10.0.13.0/30`
  R2--R4             `10.0.24.0/30`
  R3--R4             `10.0.34.0/30`
  R4 Loopback0       `192.168.4.1/24`

## Initial Configuration

EIGRP process/AS 100 was configured on all routers and the connected
networks were advertised.

R4's Loopback0 (`192.168.4.1/24`) was used as the destination network
for investigating EIGRP path selection.

## Baseline

Before making any changes, R1 learned the R4 Loopback network through
two paths:

-   R1 → R2 → R4
-   R1 → R3 → R4

Both paths had the same EIGRP metric, so both were installed as
successors.

### R1 Routing Table

![Baseline Routing Table](screenshots/baseline-routing-table.png)

The routing table shows two equal-cost paths to `192.168.4.0/24`, both
with an EIGRP metric of `131072`.

### EIGRP Neighbours

![R1 EIGRP Neighbours](screenshots/r1-neighbors.png)

R1 has EIGRP neighbours at `10.0.12.2` and `10.0.13.2`.

### EIGRP Topology Table

![Baseline EIGRP Topology](screenshots/baseline-eigrp-topology.png)

For `192.168.4.0/24`, R1 initially has **2 successors** with a feasible
distance of `131072`.

## Experiment --- Changing Bandwidth

The bandwidth of the R1--R2 link was changed to **100 Mbps**.

The purpose was to observe how changing the bandwidth affects the EIGRP
metric and route selection.

### Routing Table After the Change

![Routing Table After Bandwidth
Change](screenshots/bandwidth-routing-table.png)

After the bandwidth change, R1 installed only one route to
`192.168.4.0/24` through `10.0.13.2`.

The path through R2 was no longer equal-cost.

### EIGRP Topology After the Change

![EIGRP Topology After Bandwidth
Change](screenshots/bandwidth-eigrp-topology.png)

The EIGRP topology table now shows:

-   **1 successor** via `10.0.13.2`
-   The alternative path via `10.0.12.2` remains available with a higher
    feasible distance.

For the R4 Loopback network, the successor has a composite metric of
`131072`, while the alternative path has a metric of `154112`.

### R4 Loopback Investigation

![R4 Loopback EIGRP Topology](screenshots/loopback-eigrp-topology.png)

The detailed topology entry shows the different metrics and vector
values for the two available paths.

The preferred path has:

-   Minimum bandwidth: `1000000 Kbit`
-   Total delay: `5020 microseconds`
-   Composite metric: `131072`

The alternative path has:

-   Minimum bandwidth: `100000 Kbit`
-   Total delay: `5020 microseconds`
-   Composite metric: `154112`

## Observation

Changing the bandwidth of the R1--R2 link changed the EIGRP metric for
that path.

Initially, both paths to `192.168.4.0/24` had equal metrics and were
successors. After reducing the R1--R2 bandwidth, the R1--R3--R4 path
became the preferred successor, while the R1--R2--R4 path remained as an
alternative feasible path.

## Key Takeaways

-   EIGRP can install multiple equal-cost successor paths.
-   EIGRP uses its metric to compare available paths.
-   Interface bandwidth is one of the values used in EIGRP metric
    calculation.
-   Changing bandwidth can change the preferred path.
-   The EIGRP topology table can be used to investigate successors,
    feasible successors, feasible distance and reported distance.
