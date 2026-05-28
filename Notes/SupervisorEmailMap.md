# Supervisor Email Map

## Suggested Email Subject

Research idea: JSSP/RL-based autonomous intersection management with heterogeneous vehicle priority

## Short Email Draft

Dear Professor,

Following our recent discussion, I tried to organize the research idea into a clearer direction. My current understanding is that we can model autonomous intersection management as a constrained scheduling/resource allocation problem. The intersection can be divided into conflict zones, and each vehicle trajectory can be represented as an ordered sequence of zones to occupy. This creates a natural connection to the Job-Shop Scheduling Problem (JSSP), where vehicles are jobs, conflict zones are machines/resources, and zone traversal events are operations.

The main paper I am using as the starting point is "Reinforcement-Learning-Based Job-Shop Scheduling for Intelligent Intersection Management" (DATE 2023). It transforms graph-based intersection scheduling into a constrained JSSP and trains a PPO-based reinforcement learning scheduler. I would like to extend this direction by considering heterogeneous vehicles and trajectory-based receding-horizon scheduling. In particular, buses, trucks, and emergency/service vehicles can be assigned higher delay weights because their delay has larger practical influence, while the safety constraints remain unchanged.

The initial research plan is:

- Divide the intersection into conflict zones/resources.
- Use trajectory data to estimate each vehicle's zone sequence, arrival time, and processing times.
- Build a constrained-JSSP instance over a receding horizon.
- Optimize weighted total delay with safety, same-lane order, blocking, release-time, and deadlock constraints.
- Compare RL/PPO or graph-based learned dispatching with FCFS, greedy, priority-aware greedy, deadlock-aware greedy, and exact optimization on small cases.

The expected contribution would be a data-driven scheduling framework for single-intersection AIM that explicitly handles heterogeneous vehicle priorities. Later extensions could include edge computing, federated learning across intersections, or network-level priority/timestamp incentive mechanisms.

Could we use this as the initial common research direction? I would also appreciate your feedback on whether the first step should focus more on the mathematical JSSP formulation, the simulation environment, or the RL scheduling policy.

Best regards,

[Your Name]

## One-Page Agreement Map

![Priority-aware data-driven JSSP for AIM concept map](../Pic/PriorityAwareJSSPAIMMindMap.png)

### Core Motivation

Autonomous intersections can be treated as scheduling systems rather than fixed signal systems. If the intersection manager knows the incoming vehicles and their planned trajectories, it can allocate conflict zones safely and efficiently.

### Starting Point

The DATE 2023 paper maps graph-based intersection management to constrained JSSP and uses PPO to learn a centralized scheduling policy. This project uses that idea as the foundation.

### Proposed Extension

Add heterogeneous vehicle priorities:

- Passenger cars: normal delay weight.
- Trucks: higher delay weight due to size, slow acceleration, and blocking impact.
- Buses: higher delay weight due to passenger capacity and social cost.
- Emergency/service vehicles: very high priority weight, while still respecting safety.

### Technical Idea

Model each vehicle as:

```text
x_i = (r_i, t_i, class_i, w_i)
```

Optimize:

```text
minimize sum_i w_i * delay_i
```

under:

- Conflict-zone capacity constraints.
- Vehicle trajectory order.
- Same-lane FIFO/no-overtaking.
- Release-time constraints.
- Blocking constraints.
- Deadlock avoidance.

### Method

Use a receding-horizon scheduler:

1. Collect trajectory and vehicle-class data.
2. Predict arrivals and zone processing times.
3. Build a finite constrained-JSSP instance.
4. Solve using RL/PPO, graph-based policy, or baselines.
5. Execute near-term decisions and replan.

### Experiments

Compare:

- FCFS.
- Greedy.
- Priority-weighted greedy.
- Deadlock-aware greedy.
- RL/PPO scheduler.
- Exact optimization for small cases.

Measure:

- Average delay.
- Weighted total delay.
- Delay by vehicle class.
- Throughput.
- Runtime.
- Deadlocks or infeasible actions.
- Robustness to prediction error.

### Questions for Supervisor

- Should the first milestone be a clean mathematical model or a simulation prototype?
- Are bus/truck priority weights a good starting point, or should emergency vehicles be included from the beginning?
- Should the first experiment use synthetic vehicle arrivals or existing trajectory data?
- Is edge computing/federated learning better kept as future work after the single-intersection scheduler is stable?
