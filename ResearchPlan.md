# Research Plan: JSSP-Based Autonomous Intersection Management

## 1. Research Motivation

Autonomous Intersection Management (AIM) can replace fixed traffic signals with a scheduling mechanism that coordinates connected and autonomous vehicles through an intersection. The key safety problem is conflict resolution: two vehicles must not occupy the same conflict area at the same time. The key efficiency problem is scheduling: among all safe crossing orders, choose the one that minimizes delay and improves traffic flow.

This project follows the idea of Huang et al. (DATE 2023), which transforms graph-based intersection management into a Job-Shop Scheduling Problem (JSSP) with additional constraints. The extension proposed here is to consider heterogeneous vehicles and trajectory data in a receding-horizon scheduling framework. In particular, buses, trucks, emergency/service vehicles, and ordinary passenger cars can have different priority weights because their delay has different social, operational, or safety impact.

The research question is:

> How can a single autonomous intersection be modeled as a constrained JSSP/resource allocation problem, and how can reinforcement learning or data-driven scheduling improve weighted traffic delay while preserving collision-free and deadlock-free operation?

![Priority-aware data-driven JSSP for AIM concept map](Pic/PriorityAwareJSSPAIMMindMap.png)

## 2. Problem Formulation

### 2.1 Intersection as Resources

The intersection is divided into non-overlapping conflict zones. Each conflict zone is treated as a reusable resource that can be occupied by at most one vehicle at a time.

Let the intersection be:

```text
I = (Z, T)
```

where:

- `Z = {z_1, z_2, ..., z_m}` is the set of conflict zones.
- `T` is the set of feasible trajectories through the intersection.
- Each trajectory `t_i in T` is an ordered sequence of conflict zones, such as straight, left turn, or right turn.

This resource view is useful because it can represent different intersection shapes and different levels of spatial granularity. A coarse model has fewer resources and lower computation cost. A fine-grained model has more zones and can express more detailed vehicle interactions.

### 2.2 Vehicles as Jobs

Each incoming vehicle is modeled as a job:

```text
x_i = (r_i, t_i, class_i, w_i)
```

where:

- `r_i` is the earliest arrival or release time.
- `t_i` is the predicted trajectory through the conflict zones.
- `class_i` is the vehicle type, such as car, bus, truck, or emergency/service vehicle.
- `w_i` is the priority weight used in the objective.

Example priority interpretation:

| Vehicle class | Example weight | Interpretation |
| --- | ---: | --- |
| Passenger car | 1.0 | Standard delay cost |
| Truck | 1.5 | Higher influence due to size, acceleration, and blocking impact |
| Bus | 2.0 | Higher social cost because many passengers are affected |
| Emergency/service vehicle | 3.0 or higher | Strong priority, still constrained by safety |

The exact weights should be calibrated experimentally. The important principle is that priority changes the optimization objective, not the safety constraints.

### 2.3 JSSP Mapping

The AIM problem can be mapped to JSSP as follows:

| AIM concept | JSSP concept |
| --- | --- |
| Vehicle | Job |
| Conflict zone | Machine/resource |
| Vehicle occupying a zone | Operation |
| Predicted zone traversal time | Processing time |
| Vehicle trajectory order | Job operation order |
| Earliest arrival time | Release time |
| Conflict-zone capacity | Machine capacity |
| Crossing order | Schedule/dispatching policy |

For vehicle `x_i`, if the predicted trajectory is:

```text
z_{i,1} -> z_{i,2} -> ... -> z_{i,n_i}
```

then the corresponding JSSP job is:

```text
o_{i,1} -> o_{i,2} -> ... -> o_{i,n_i}
```

where operation `o_{i,j}` means vehicle `x_i` occupies conflict zone `z_{i,j}` for processing time `p_{i,j}`.

### 2.4 Required Constraints

The scheduling model must include the following constraints:

- Trajectory order: a vehicle must pass conflict zones according to its planned trajectory.
- One-vehicle-per-zone safety: a conflict zone cannot be occupied by two vehicles at the same time.
- Same-lane FIFO/no-overtaking: vehicles from the same incoming lane should preserve their physical order.
- Arrival/release time: a vehicle cannot be scheduled before it reaches the intersection control area.
- Blocking: a vehicle cannot disappear between conflict zones; if it is waiting for the next zone, it may still block the previous zone.
- Deadlock avoidance: a schedule must not create circular waiting, such as multiple vehicles each occupying a zone while waiting for another zone.
- Priority consistency: high-priority vehicles receive larger delay penalties, but no priority rule may violate collision-free or deadlock-free constraints.

### 2.5 Objective

The DATE 2023 paper minimizes total delay rather than only makespan. This project extends the objective to weighted total delay:

```text
minimize sum_i w_i * (C_i - r_i - P_i)
```

where:

- `C_i` is the completion/leaving time of vehicle `x_i`.
- `r_i` is its earliest arrival time.
- `P_i = sum_j p_{i,j}` is the minimum physical traversal time without waiting.
- `C_i - r_i - P_i` is the delay caused by scheduling and conflicts.
- `w_i` is the vehicle priority weight.

This objective keeps the optimization practical: buses and trucks matter more, but the scheduler still searches for a system-level compromise rather than always forcing hard precedence.

## 3. Proposed Method

### 3.1 Data-Driven Receding-Horizon Workflow

The phrase "horizontal predictive" is interpreted here as receding-horizon predictive scheduling. At each decision cycle:

1. Collect vehicle states and trajectories from sensors, V2X messages, or simulation logs.
2. Predict each vehicle's arrival time, intended movement, and conflict-zone processing times.
3. Build a finite constrained-JSSP instance for the current horizon.
4. Solve or approximate the schedule using RL, heuristic baselines, or optimization baselines.
5. Execute only the near-term part of the schedule.
6. Update the horizon when new vehicles arrive or predictions change.

This avoids committing to a long schedule under uncertain future traffic and makes the method closer to real-time AIM operation.

### 3.2 Reinforcement Learning Formulation

Following the DATE 2023 paper and learning-to-dispatch JSSP work, the scheduling process can be modeled as a Markov Decision Process (MDP).

State:

- Current graph/JSSP representation of scheduled and unscheduled operations.
- Operation features: scheduled/not scheduled, earliest completion time, release time, vehicle class, priority weight, lane index, trajectory index, and conflict-zone index.
- Resource features: current occupancy, blocking status, and next available time.

Action:

- Select one currently feasible operation or vehicle dispatch decision.
- The action set is masked by safety, release-time, blocking, and deadlock constraints.

Transition:

- The selected operation receives its earliest feasible start time.
- Conflict-zone availability, vehicle position, and graph edges are updated.

Reward:

- Negative incremental weighted delay:

```text
r_t = - Delta(sum_i w_i * delay_i)
```

- Additional penalties may be used for excessive waiting, unstable schedules, or infeasible actions if the environment allows them. Infeasible actions should preferably be masked rather than punished after selection.

Policy model:

- A graph neural network policy is suitable because the state is naturally graph-structured.
- PPO is the first candidate because it is used in the source paper and is stable for discrete scheduling decisions.
- Offline or imitation learning can be considered later if enough trajectory/schedule data are collected.

### 3.3 Baselines

The research should compare the proposed RL scheduler against several interpretable baselines:

- FCFS: first-come-first-served reservation scheduling.
- Greedy delay minimization: select the feasible operation that gives the smallest immediate delay.
- Priority-weighted greedy: select the feasible operation with highest weighted urgency.
- iGreedy/deadlock-aware greedy: greedy scheduling with explicit deadlock checking, as used by Huang et al.
- Constraint programming or MILP: used for small instances to estimate optimality gaps, not for real-time control.

### 3.4 Evaluation Metrics

Primary metrics:

- Average vehicle delay.
- Weighted total delay.
- Delay by vehicle class.
- Throughput and intersection clearing time.
- Optimality gap on small instances where exact optimization is possible.

Safety and feasibility metrics:

- Number of collisions or resource conflicts, expected to be zero.
- Number of deadlocks, expected to be zero.
- Number of infeasible scheduling attempts.

Operational metrics:

- Runtime per scheduling horizon.
- Sensitivity to traffic density.
- Sensitivity to vehicle mix, such as high bus/truck ratio.
- Robustness to trajectory prediction error.
- Fairness tradeoff between high-priority and ordinary vehicles.

## 4. Related Work

- Dresner and Stone (2008) introduced reservation-based autonomous intersection management, where vehicles request time-space reservations from an intersection manager. This established an influential AIM framework and also considered mixed traffic and emergency vehicle priority.
- Lin et al. (2019) proposed graph-based modeling, scheduling, and verification for intelligent intersection management. Their model represents conflict zones and timing constraints with a graph and provides deadlock-freeness analysis.
- Huang et al. (2023) transformed graph-based intersection scheduling into a constrained JSSP and trained a centralized PPO scheduler. Their results show that RL is especially useful under high traffic density.
- Guan et al. (2020) studied centralized cooperation for connected and automated vehicles at intersections using PPO, showing the relevance of deep RL for unsignalized intersection coordination.
- Wu, Chen, and Zhu (2019) proposed DCL-AIM, a decentralized coordination learning approach for CAV intersection management using a multi-agent MDP view.
- Zhang et al. (2020) proposed learning dispatching rules for JSSP with graph neural networks and reinforcement learning. This is a key methodological reference for graph-based JSSP policies.
- Khayatian et al. (2020) surveyed intersection management for connected autonomous vehicles, providing broader context on reservation, scheduling, robustness, communication, and deployment challenges.
- The local Bilkent thesis material on non-monetary intersection incentives is also relevant as a later extension: priority or timestamp mechanisms at intersections can influence network-level route choice. For this project, that idea is kept as future work rather than the core single-intersection scheduler.

## 5. Expected Contributions

The planned contribution is not simply applying PPO to traffic. The goal is to formulate a practical and extensible scheduling framework:

- A constrained JSSP formulation for single-intersection AIM with heterogeneous vehicle priorities.
- A weighted-delay objective that captures practical differences between passenger cars, buses, trucks, and emergency/service vehicles.
- A receding-horizon data-driven workflow using trajectory prediction and online schedule updates.
- A comparison between RL, greedy, priority-aware greedy, and exact optimization on small instances.
- An analysis of when learning-based scheduling is useful, especially under high density and heterogeneous traffic.

## 6. Risks and Open Questions

- Weight calibration: the priority weights must be meaningful and defensible, not arbitrary.
- Prediction error: wrong trajectory or arrival-time predictions may reduce schedule quality.
- Scalability: exact optimization is useful for evaluation but may not scale to real-time horizons.
- Generalization: a trained RL policy may not transfer well across intersection layouts, traffic densities, or vehicle mixes.
- Fairness: high-priority vehicles should not cause unacceptable delay for ordinary vehicles.
- Safety assurance: RL should not be trusted alone for safety; feasibility masks and verification checks are needed.

## 7. Near-Term Work Plan

1. Define the conflict-zone map for one representative intersection.
2. Build a small simulator or data parser that produces vehicle arrivals, trajectories, classes, and processing times.
3. Implement FCFS, greedy, and priority-weighted greedy baselines.
4. Implement a constrained-JSSP environment with action masking.
5. Train a PPO or graph-policy scheduler on synthetic traffic scenarios.
6. Compare against exact optimization on small instances and against heuristics on larger horizons.
7. Analyze the effect of vehicle heterogeneity under low, medium, and high traffic densities.

## References

- Kurt Dresner and Peter Stone. "A Multiagent Approach to Autonomous Intersection Management." Journal of Artificial Intelligence Research, 31:591-656, 2008. https://www.cs.utexas.edu/~pstone/Papers/bib2html/b2hd-JAIR08-dresner.html
- Yi-Ting Lin, Hsiang Hsu, Shang-Chien Lin, Chung-Wei Lin, Iris Hui-Ru Jiang, and Changliu Liu. "Graph-Based Modeling, Scheduling, and Verification for Intersection Management of Intelligent Vehicles." ACM Transactions on Embedded Computing Systems, 18(5s), 2019. DOI: 10.1145/3358221
- Shao-Ching Huang, Kai-En Lin, Cheng-Yen Kuo, Li-Heng Lin, Muhammed O. Sayin, and Chung-Wei Lin. "Reinforcement-Learning-Based Job-Shop Scheduling for Intelligent Intersection Management." DATE 2023. DOI: 10.23919/DATE56975.2023.10137280
- Yang Guan, Yanjun Ren, Shengbo Eben Li, Qi Sun, Longhan Luo, and Keqiang Li. "Centralized Cooperation for Connected and Automated Vehicles at Intersections by Proximal Policy Optimization." IEEE Transactions on Vehicular Technology, 69(11), 2020. https://arxiv.org/abs/1912.08410
- Yuanyuan Wu, Haipeng Chen, and Feng Zhu. "DCL-AIM: Decentralized Coordination Learning of Autonomous Intersection Management for Connected and Automated Vehicles." Transportation Research Part C, 103:246-260, 2019. DOI: 10.1016/j.trc.2019.04.012
- Cong Zhang, Wen Song, Zhiguang Cao, Jie Zhang, Puay Siew Tan, and Chi Xu. "Learning to Dispatch for Job Shop Scheduling via Deep Reinforcement Learning." NeurIPS 2020. https://proceedings.neurips.cc/paper/2020/hash/11958dfee29b6709f48a9ba0387a2431-Abstract.html
- Mohammad Khayatian, Mohammadreza Mehrabian, Edward Andert, Rachel Dedinsky, Sarthake Choudhary, Yingyan Lou, and Aviral Shrivastava. "A Survey on Intersection Management of Connected Autonomous Vehicles." ACM Transactions on Cyber-Physical Systems, 4(4), 2020. DOI: 10.1145/3407903
