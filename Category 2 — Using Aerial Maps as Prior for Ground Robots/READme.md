 # Literature Review: Aerial-Prior Ground Robot Navigation

Summary of two papers on using aerial (UAV) data to assist ground robot navigation 

---

## 1. Under-Canopy Navigation using Aerial Lidar Maps

**Authors:** Lucas Carvalho de Lima, Nicholas Lawrance, Kasra Khosoussi, Paulo Borges, Michael Brünig
**Affiliation:** University of Queensland / CSIRO Robotics, Data61
**Venue:** IEEE Robotics and Automation Letters (RA-L), 2024
**arXiv:** 2404.03911

### Problem

Ground robots navigating dense forests suffer from short-range, occluded onboard sensing, leading to inefficient paths and frequent dead-ends. Aerial lidar scanned above the canopy *before* deployment could provide global context, but two obstacles stand in the way:

1. **Canopy occlusion** — aerial lidar under-samples ground-level obstacles hidden by vegetation.
2. **Aerial pose uncertainty** — the aerial vehicle's own trajectory estimate (from SLAM/pose-graph optimization) is imperfect, and lidar hit-point error grows with range and angle. Standard occupancy mapping ignores this, producing falsely confident (and often wrong) maps.

### Method

The pipeline has four stages:

| Stage | What it does |
|---|---|
| **Ground filtering** | Cloth Simulation Filtering (CSF) inverts the point cloud, drapes a virtual "cloth" over it under simulated gravity, and uses the closest points to estimate a 0.25 m gridded ground-height map. Robust to canopy misclassification. |
| **Uncertainty-Aware (UA) occupancy mapping** | Instead of trusting a single aerial trajectory estimate, the system samples N=20 plausible trajectories from the SLAM pose posterior (Monte Carlo integration), builds an occupancy map (via OHM) per sample, and averages voxel occupancy probabilities across samples. This propagates localization uncertainty directly into map confidence. |
| **Obstruction scoring** | For each ground-level 2D cell, a weighted average of occupancy probability is computed over a vertical column of voxels (n=4 voxels, weights 1:2:2:2 from bottom up) to produce an obstruction risk score `b(s) ∈ [0,1]`. |
| **Global path planning** | D* Lite plans a minimum-cost path using one of two cost functions: **Expected Cost** (blends obstruction penalty with travel distance) or **Log-Reachability Cost** (maximizes probability of unobstructed traversal). D* Lite supports efficient replanning when the ground robot hits a previously unmapped obstacle. |

### Key Results

- **Simulation (3 synthetic forests):** UA-occupancy consistently beats standard occupancy mapping — lower KL-divergence vs. ground truth and higher ROC-AUC — across all voxel classes (free/occupied/uncertain). The advantage grows as aerial trajectory noise increases (SLAM-like drift vs. GPS-like accuracy).
- **Simulation (full navigation, 40 start/goal pairs):** Log-Reachability cost with prior aerial maps produced significantly shorter executed paths than a naïve (no-prior) baseline planner, especially under high-uncertainty (SLAM-like) aerial trajectories.
- **Real-world (QCAT forest, Brisbane, DJI M-300 + tracked robot):** In two field tests, the aerial-prior method found shorter, less cluttered paths (e.g., 69.6 m vs. 79.9 m for the naïve baseline in Test 1). In Test 2, the naïve local-only planner drove into a dead-end of dense undergrowth and failed to reach the goal; the aerial-prior method avoided it entirely.
- **Runtime:** ~37 min for 28 million lidar points on a 4-core i7 — feasible for offline pre-mission mapping, not yet real-time (GPU acceleration suggested as future work).

### Notable Limitations

- Purely offline mapping/planning; online execution is handed off to an existing local navigation stack.
- No ground-truth obstruction labels exist for real forest data, so real-world evaluation is qualitative.
- Assumes a single static aerial map; no mechanism to update it during ground operation.

---

## 2. Air-Ground Collaboration with SPOMP: Semantic Panoramic Online Mapping and Planning

**Authors:** Ian D. Miller, Fernando Cladera, Trey Smith, Camillo Jose Taylor, Vijay Kumar
**Affiliation:** GRASP Lab, University of Pennsylvania / NASA Ames Intelligent Robotics Group
**arXiv:** 2407.09902 (2024)

### Problem

Broader in scope than Paper 1: this is a **multi-robot** system (1 UAV + multiple UGVs) operating in **GPS-denied, intermittently-connected** outdoor environments. The central question is: what *shared representation* should robots communicate to collaborate effectively? The authors argue it must be:

- **Sufficient** (little performance loss vs. full internal state)
- **Efficient** (cheap to transmit)
- **Common** (usable across heterogeneous robots)
- **Interpretable** (understandable to humans, not just robots)

Their answer: **semantics** — human-meaningful class labels (road, grass, dirt, building, vegetation, vehicle) — because they are compact, transferable across viewpoints/platforms, and directly predictive of traversability.

### System Architecture (SPOMP)

**Aerial side:**
- **ASOOM** (Aerial Semantic Online Ortho-Mapping) — builds a semantic orthomap (RGB + class + elevation layers) online using ORB-SLAM3 odometry + GPS (the *only* GPS use in the whole system). Compressed ~10x for transmission.
- **Aerial planner** — a UAV state machine balances two competing goals: (1) explore to extend the map, (2) act as a communication relay by visiting UGVs to sync data via their distributed database, **MOCHA**. Time-shared between the two.

**Ground side:**
- **Odometry + segmentation** — modified LLOL (Low-Latency Odometry for spinning LiDARs) produces integrated depth panoramas approximately every meter of travel; RangeNet++ semantically segments them.
- **Cross-view localizer** — matches ground semantic panoramas against the aerial semantic orthomap for GPS-free localization in a shared coordinate frame.
- **Global planner** — builds a **sparse traversability graph** (not a dense occupancy grid) over the aerial map. Edge cost = −log(P(traversable)). A small **MLP** (1 hidden layer, size 10) is trained *online, on each robot, every 10 seconds* to predict traversability from color + semantic-class-distance + elevation, using the robot's own driving experience as labels — then **extrapolates that experience across the entire map** (e.g., "road" patches the robot hasn't visited yet inherit traversability from ones it has). Robots share this learned experience with each other, though only a robot's *own* observations can permanently mark an edge as untraversable (prevents bad data from other robots corrupting the graph).
- **Local planner** — plans directly on depth panoramas in real time for close-range obstacle avoidance.

### Key Results

- **Simulation:** Team-scaling experiments (up to several UGVs) on a 15-car-cluster search task; ablations show the learned semantic traversability model meaningfully improves mission performance over not having it.
- **Real-world:** Tested at two large sites — **Pennovation** (urban, ~cluttered patios, parking lots, active road construction) and **New Bolton Center** (rural) — with 3 Clearpath Jackal UGVs + 1 custom PX4 UAV. Totaled **~18 km of autonomous driving** across the fleet, demonstrating robustness under real, uncontrolled conditions (traffic, construction equipment, intermittent connectivity).
- System operates with **no assumption of constant communication or connectivity** — robots opportunistically sync via MOCHA whenever in range.

### Notable Limitations (stated by authors)

- Assumes the aerial orthomap is representative of the ground truth — breaks down under overhangs, bridges, or other areas the aerial view can't see.
- No mechanism for ground robots to update the aerial map, or for the system to revise previously-mapped (but now-changed) regions.
- Dynamic obstacles are not explicitly handled.
- Single-UAV, multi-UGV architecture may not generalize to all mission types.

---

## Side-by-Side Comparison

| | Under-Canopy Navigation (Lima et al.) | SPOMP (Miller et al.) |
|---|---|---|
| **Scope** | Single ground robot, offline planning | Multi-robot (1 UAV + N UGVs), online collaboration |
| **Aerial data** | Lidar point cloud → 3D occupancy map | Camera/lidar → semantic orthomap (RGB + class + elevation) |
| **Shared representation** | Probabilistic occupancy grid | Semantic labels (human-interpretable classes) |
| **Uncertainty handling** | Explicit — Monte Carlo over SLAM pose posterior | Implicit — learned traversability confidence via online MLP |
| **Planner** | D* Lite over dense grid, two cost functions (Expected Cost, Log-Reachability) | Graph search over sparse traversability graph, edge cost = −log P(traversable) |
| **Learning** | None — purely probabilistic/geometric | Online MLP, retrained every 10s from robot's own driving experience |
| **Communication model** | N/A (single robot, offline aerial map) | Opportunistic, intermittent (MOCHA distributed DB), no connectivity assumption |
| **Ground platform tested** | Dynamic Tracked Robot (DTR) | 3× Clearpath Jackal |
| **Real-world scale** | 2 test routes, QCAT forest | ~18 km across 2 large sites (urban + rural) |
| **Core insight** | Propagating aerial *pose* uncertainty into the map improves planning safety | Semantics are a better shared language between heterogeneous robots than raw geometry, and can be learned/extrapolated online |

## Relevance to Project Zero (SAFiR Lab)

Both papers are directly on-topic for resource-aware exploration using aerial priors on the Jackal:

- **Lima et al.** offers a concrete, well-validated recipe for turning an aerial prior into a ground-robot cost map and plan (D* Lite + obstruction scoring), plus a principled way to handle localization uncertainty in the aerial data — relevant if aerial priors are captured via drone/simulated overflight before Jackal deployment.
- **Miller et al.** is architecturally closer in hardware (Jackal UGVs) and offers a lighter-weight alternative to dense occupancy grids: a sparse semantic traversability graph with online-learned costs, which could be attractive given the Jackal's onboard compute (i5 CPU + RTX 3070) and the YOLO-based vision pipeline already in use — semantic class could feed a similar online traversability learner instead of/alongside geometric SLAM occupancy.

Both are strong candidates for the related-work section of the FYP/Mitacs writeup, and SPOMP in particular is a good source for framing the "why semantics + aerial priors" argument for a research narrative.
