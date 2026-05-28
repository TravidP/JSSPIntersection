# Paper and Meeting Idea Summary

## Purpose of This Note

This note organizes the research idea discussed with the supervisor. The core direction is to use Job-Shop Scheduling Problem (JSSP) or related resource allocation methods to model autonomous intersection management. The plan is based mainly on the DATE 2023 paper "Reinforcement-Learning-Based Job-Shop Scheduling for Intelligent Intersection Management" and extends it toward heterogeneous vehicles, weighted priority, trajectory data, and receding-horizon prediction.

![Priority-aware data-driven JSSP for AIM concept map](../Pic/PriorityAwareJSSPAIMMindMap.png)

## 1. Source Paper: Main Idea

Huang et al. propose a centralized intersection manager for connected and autonomous vehicles. Their key idea is:

1. Divide the intersection into conflict zones.
2. Represent each vehicle trajectory as an ordered sequence of conflict zones.
3. Transform the intersection scheduling problem into a constrained JSSP.
4. Model the scheduling process as an MDP.
5. Train a reinforcement learning agent with PPO.
6. Use a grouping strategy so that a trained scheduler can handle a stream of vehicles.

In this formulation, an intersection is not controlled by fixed signal phases. Instead, the manager decides the passing order of vehicles through shared conflict zones. If each conflict zone is occupied by at most one vehicle at a time, and if the schedule avoids deadlock, then the crossing can be collision-free and efficient.

![JSSP traffic sketch](../Pic/RLJSSPPrePaper.png)

## 2. Mapping AIM to JSSP

The paper's most important modeling step is the mapping between autonomous intersection management and JSSP:

| Intersection management | JSSP |
| --- | --- |
| Vehicle | Job |
| Conflict zone | Machine/resource |
| Vehicle crossing one zone | Operation |
| Zone traversal time | Processing time |
| Vehicle route through the intersection | Operation sequence |
| Arrival time | Release time |
| Vehicle crossing order | Schedule |

For example, if a car goes through three conflict zones, this becomes a job with three ordered operations. Each operation requires one resource: the corresponding conflict zone.

This is a useful abstraction because the intersection manager can be viewed as a scheduling system. The goal is to decide which operation or vehicle should be served next while maintaining safety.

## 3. Constraints Beyond Standard JSSP

The source paper emphasizes that AIM is not exactly a standard JSSP. Several additional constraints are needed:

- Blocking constraint: a vehicle cannot be removed from the intersection while waiting for the next zone. It may continue blocking its current zone.
- Deadlock constraint: the schedule must avoid circular waiting among vehicles.
- Arrival-time constraint: vehicles cannot be scheduled before they physically arrive.
- Same-lane order constraint: vehicles from the same lane should not overtake each other.
- Conflict-zone capacity constraint: each conflict zone can be occupied by at most one vehicle at a time.

These constraints are essential because the method is controlling physical vehicles, not abstract manufacturing jobs.

## 4. Supervisor Discussion: Research Direction

The discussion can be summarized as the following research idea:

> Use constrained JSSP, queuing, or resource allocation models to describe autonomous intersection management. Divide the intersection into resources/conflict zones. Use vehicle trajectory data and prediction to build a scheduling problem over a receding horizon. Use reinforcement learning or other data-driven methods to optimize the conflict-zone allocation, while considering heterogeneous vehicle priorities such as buses, trucks, and emergency vehicles.

Key points from the board discussion:

- Traffic management can be studied as data-driven optimization.
- The intersection can be divided into small zones or resources.
- Vehicle trajectories define which resources are needed and in what order.
- Scheduling the crossing is similar to JSSP, queuing, and constrained allocation.
- Reinforcement learning can learn dispatching/scheduling policies.
- Edge processing or edge computing could support real-time scheduling near the intersection.
- Federated learning may be a future extension if multiple intersections or privacy-sensitive data are involved.
- Receding-horizon prediction is important because vehicles arrive continuously and their trajectories are uncertain.
- Heterogeneous vehicles should be included because buses and trucks have different practical effects from passenger cars.

## 5. Proposed Extension: Heterogeneous Vehicles

The original paper mainly focuses on scheduling vehicles efficiently and safely. The proposed extension is to make the scheduler aware of heterogeneous vehicle classes.

Each vehicle has:

```text
x_i = (r_i, t_i, class_i, w_i)
```

where `w_i` is a priority weight. The objective becomes weighted total delay:

```text
minimize sum_i w_i * delay_i
```

This means that one second of bus delay can count more than one second of passenger-car delay, because a bus carries more passengers and has larger practical influence. A truck can also receive a higher weight because it may accelerate slowly, occupy more space, and affect downstream flow more strongly.

This approach is safer than hard priority rules. A bus or truck can be preferred in the objective, but it still cannot violate conflict-zone, blocking, or deadlock constraints.

## 6. Receding-Horizon Predictive Scheduling

The scheduling problem should be solved repeatedly over a moving time horizon:

1. Detect incoming vehicles and collect trajectory information.
2. Predict each vehicle's arrival time and intended movement.
3. Build a constrained-JSSP instance for vehicles inside the current horizon.
4. Compute the crossing schedule.
5. Execute only the first part of the schedule.
6. Update predictions and re-solve when new vehicles enter the control area.

This makes the method suitable for continuous traffic streams. It also reduces the risk of making long-term decisions based on uncertain predictions.

## 7. Candidate Methods

The main learning method is reinforcement learning:

- State: graph/JSSP representation, vehicle features, resource status, operation completion estimates.
- Action: choose the next feasible operation or vehicle to schedule.
- Reward: negative weighted delay or reduction in weighted total delay.
- Policy: PPO with a graph neural network is the first candidate because it follows the source paper.

Other related methods should be compared:

- FCFS reservation scheduling.
- Greedy scheduling.
- Priority-weighted greedy scheduling.
- Deadlock-aware greedy scheduling.
- Exact optimization for small instances.
- Queueing-based models for analytical comparison.

## 8. Research Novelty

The potential novelty is:

- Combine graph-based AIM and constrained JSSP with explicit vehicle heterogeneity.
- Use weighted delay to model the practical influence of buses, trucks, and service vehicles.
- Use trajectory data and prediction to generate online scheduling instances.
- Study whether RL is more useful under high traffic density and mixed vehicle classes.
- Compare learning-based scheduling with interpretable queueing and greedy baselines.

## 9. Short Version for Discussion

My proposed research direction is to model an autonomous intersection as a constrained scheduling system. The intersection is divided into conflict zones, and each vehicle trajectory becomes a sequence of resources to occupy. This naturally maps the problem to JSSP with additional physical constraints such as blocking, release times, lane order, and deadlock avoidance. Building on the DATE 2023 JSSP-RL paper, I want to extend the model to heterogeneous vehicles by optimizing weighted total delay, so buses, trucks, and emergency/service vehicles can have larger influence without violating safety. The system would use trajectory data and receding-horizon prediction to repeatedly build and solve scheduling instances, using RL/PPO or data-driven policies and comparing them with FCFS, greedy, priority-aware greedy, and exact small-instance optimization.

## Local Materials

- Source paper: [DATE 2023 PDF](../Doc/Reinforcement-learning-based_job-shop_scheduling_for_intelligent_intersection_management.pdf)
- Presentation: [DATE 2023 PPTX](../Doc/Reinforcement-Learning-Based%20Job-Shop%20Scheduling%20for%20Intelligent%20Intersection%20Management.pptx)
- Board discussion image: [JSSP traffic sketch](../Pic/JSSPTraffic.png)
- Full research plan: [ResearchPlan.md](../ResearchPlan.md)
