# Category 1 – Autonomous Exploration

## Paper 1: TARE: A Hierarchical Framework for Efficiently Exploring Complex 3D Environments

### Overview

The TARE framework was developed to improve autonomous robot exploration in large, complex, and previously unknown three-dimensional environments. It introduces a hierarchical planning architecture that separates exploration into local and global planning to improve efficiency while reducing computational cost.

### Key Contributions

- Uses a hierarchical planning framework with local and global exploration.
- Maintains a high-resolution map only around the robot.
- Uses a coarse representation for distant regions to reduce computation.
- Selects viewpoints that maximize new surface coverage while minimizing overlap.
- Combines A*, Travelling Salesman Problem (TSP), and spline-based smoothing to generate smooth and collision-free paths.
- Stores partially explored regions as subspaces and revisits them automatically.
- Eliminates the need for manually defined switching between local and global exploration.

### Results

- Achieved more complete exploration than existing methods.
- Improved exploration efficiency by approximately **80%**.
- Reduced computational cost while maintaining high exploration performance.

---

## Paper 2: Under-Canopy Navigation Using Aerial LiDAR Maps

### Overview

This paper presents a navigation framework that enables ground robots to navigate dense forest environments using aerial LiDAR maps collected by a drone before the mission. The aerial information helps overcome the limited sensing range of ground robots caused by vegetation and obstacles.

### Key Contributions

- Uses aerial LiDAR scans to generate a three-dimensional probabilistic occupancy map.
- Converts the occupancy map into a two-dimensional obstruction-risk map.
- Models uncertainty in the drone's trajectory using Monte Carlo simulation.
- Produces uncertainty-aware maps by averaging multiple occupancy predictions.
- Combines obstruction probability with travel distance using expected-cost and log-reachability formulations.
- Uses the D* Lite algorithm for path planning and dynamic route updates.

### Results

- Generated shorter and safer navigation paths.
- Reduced unnecessary replanning during navigation.
- Helped the robot avoid cluttered regions and dead ends.
- Improved navigation performance using aerial prior information.

---

## Conclusion

Both papers focus on improving autonomous robot exploration and navigation in challenging environments.

- **TARE** improves exploration efficiency through hierarchical planning and intelligent viewpoint selection.
- **Under-Canopy Navigation** improves path planning by combining aerial LiDAR maps with uncertainty-aware navigation.

Together, these approaches demonstrate that combining efficient planning strategies with environmental awareness can significantly enhance autonomous robot performance in complex and unknown environments.