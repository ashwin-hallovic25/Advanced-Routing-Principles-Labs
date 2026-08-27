# Week 3 — Single-Area OSPF Home Lab

A self-directed GNS3 home lab based on my university Week 3 **Single-Area OSPF** work.

The project focuses on configuring OSPF, verifying it, then changing selected parameters and observing what happens.

## 1. Topology

The lab uses three OSPF routers on a shared multiaccess Ethernet segment.

- **R1** — simulated external/Internet side using Loopback0
- **D1** — internal router
- **D2** — internal router
- **Shared OSPF network:** `10.10.0.0/29`
- **R1 Loopback0:** `192.168.1.1/26`

The shared Ethernet segment is used for OSPF neighbor formation and DR/BDR election.

![OSPF topology](Screenshots/topology.png)

## 2. Objectives

- Activate OSPF using different methods.
- Verify OSPF operation with `show` commands.
- Configure R1 to originate a default route.
- Observe the effect of mismatched Hello/Dead timers.
- Restore the timers and verify adjacency recovery.
- Change OSPF reference bandwidth and observe inconsistency.
- Investigate DR/BDR election.
- Change OSPF interface priority and observe the election outcome.

## 3. OSPF Activation Methods

### Method 1 — Network command with the appropriate wildcard

```text
router ospf 123
 network 10.10.8.0 0.0.0.255 area 0
```

### Method 2 — Network command with `0.0.0.0`

```text
router ospf 123
 network 10.10.9.1 0.0.0.0 area 0
```

### Method 3 — Activate OSPF directly on the interface

```text
interface GigabitEthernet0/0
 ip ospf 123 area 0
```

This was the primary method used in this home lab.

### Method 4 — Match every interface

```text
router ospf 123
 network 0.0.0.0 255.255.255.255 area 0
```

This is simple, but not recommended because it enables OSPF on all matching interfaces.

![OSPF activation methods](Screenshots/1.%20Enabling%20OSPF%20on%20interfaces%20through%20various%20methods.png)

## 4. OSPF Verification

### `show ip protocols`

Used to confirm the OSPF process, router ID, area and interfaces participating in OSPF.

![show ip protocols](Screenshots/2.%20OSPF%20Verification%20-%20show%20ip%20protocols.png)

### `show ip ospf interface brief`

Used to check OSPF-enabled interfaces, cost, interface state and neighbor count.

![show ip ospf interface brief](Screenshots/3.%20OSPF%20Verification%20-%20show%20ip%20ospf%20interface%20brief.png)

### `show ip route`

Used to verify routes learned through OSPF.

![OSPF routes before default route](Screenshots/4.%20OSPF%20Verification%20-%20show%20ip%20route.png)

## 5. Default Route Propagation

R1's Loopback0 represents the simulated Internet/external network.

A static default route was configured:

```text
ip route 0.0.0.0 0.0.0.0 Loopback0
```

Then R1 was configured to originate the default route into OSPF:

```text
router ospf 123
 default-information originate
```

![Configure default route](Screenshots/5.%20Configuring%20default%20route.png)

### Verification

The other router learned the default route as an OSPF external Type 2 route:

```text
O*E2 0.0.0.0/0
```

![Default route verification](Screenshots/6.%20Default%20Route%20Verification%20-%20show%20ip%20route.png)

## 6. Experiment — Hello and Dead Timers

The Hello and Dead timers on R1 were changed:

```text
interface GigabitEthernet0/0
 ip ospf hello-interval 5
 ip ospf dead-interval 20
```

![Hello and Dead timer change](Screenshots/7.%20Hello%20and%20Dead%20TImer%20-%20Manual%20Adjustment.png)

The neighboring routers were still using the default timers, so the timers became inconsistent and the OSPF adjacency was affected.

![Inconsistent timers](Screenshots/8.%20Incosistent%20Hello%20and%20Dead%20Interval%20Effect.png)

The timers were then restored:

```text
interface GigabitEthernet0/0
 ip ospf hello-interval 10
 ip ospf dead-interval 40
```

![Restored timers](Screenshots/9.%20Restoration%20of%20Hello%20and%20Dead%20Interval%20Effect.png)

**Observation:** OSPF neighbors on the same link must agree on the Hello and Dead intervals.

## 7. Experiment — OSPF Reference Bandwidth

The reference bandwidth was changed on R1:

```text
router ospf 123
 auto-cost reference-bandwidth 10000
```

![Reference bandwidth change](Screenshots/10.%20Reference%20Bandwidth%20-%20Manual%20Adjustment.png)

IOS warned that the reference bandwidth should be consistent across all routers.

![Reference bandwidth inconsistency](Screenshots/11.%20Reference%20Badwidth%20-%20Inconsistency%20Created.png)

### Observation

An inconsistent reference bandwidth can cause routers to calculate OSPF interface costs differently.

![Reference bandwidth effect](Screenshots/12.%20Reference%20Bandwidth%20-%20Inconsistency%20Effect.png)

## 8. Experiment — DR/BDR Election

Because the routers share the same Ethernet/multiaccess network, OSPF performs a DR/BDR election.

### Initial state

The initial output showed the DR/BDR roles.

![Default DR/BDR state](Screenshots/13.%20DR-BDR%20Default%20Status.png)

### Prevent D2 from becoming DR

D2's OSPF priority was changed to zero:

```text
interface GigabitEthernet0/0
 ip ospf priority 0
```

A priority of `0` makes the router ineligible to become DR or BDR.

![D2 priority zero](Screenshots/14.%20DR-BDR%20-%20Removing%20D2%20as%20DR.png)

### Make R1 preferred

R1's OSPF priority was changed to `255`:

```text
interface GigabitEthernet0/0
 ip ospf priority 255
```

![R1 DR election](Screenshots/15.%20DR%20-%20BDR%20-%20Making%20R1%20as%20DR.png)

**Observation:** OSPF interface priority influences DR/BDR election. A priority of `0` makes a router ineligible.

## 9. Main Verification Commands

```text
show ip protocols
show ip ospf
show ip ospf interface brief
show ip ospf interface GigabitEthernet0/0
show ip ospf neighbor
show ip route
show ip route ospf
show ip ospf database
```

Useful focused commands:

```text
show ip ospf interface GigabitEthernet0/0 | include Timer
show ip ospf | include Reference
```

## 10. Key Takeaways

- OSPF can be activated with network statements or directly on an interface.
- `default-information originate` advertises a default route through OSPF.
- Hello and Dead timers must match between OSPF neighbors.
- Reference bandwidth should be consistent across the OSPF domain.
- DR/BDR election occurs on a multiaccess OSPF network.
- OSPF interface priority affects DR/BDR election.
- Priority `0` prevents a router from becoming DR or BDR.


![OSPF activation methods](Screenshots/1.%20Enabling%20OSPF%20on%20interfaces%20through%20various%20methods.png)
