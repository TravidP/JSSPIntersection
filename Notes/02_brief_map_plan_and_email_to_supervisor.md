# Brief Map Plan and Email Draft for Supervisor

## 1. Short common-agreement map

### Research direction

**Priority-aware data-driven scheduling for autonomous intersection management.**

### Main idea

Divide the intersection into conflict zones. Treat each conflict zone as a limited resource. Treat each vehicle as a job. Treat the vehicle trajectory as a sequence of conflict-zone operations. Then solve the vehicle passing-order problem as a constrained Job-Shop Scheduling Problem.

![Priority-aware data-driven JSSP for AIM concept map](../Pic/PriorityAwareJSSPAIMMindMap.png)

### Extension beyond the base paper

The base paper models autonomous intersection management as constrained JSSP and trains a PPO-based reinforcement learning scheduler. My extension is to consider **heterogeneous vehicles and vehicle priority**.

Examples:

- Bus: higher priority because it carries many passengers.
- Truck: different priority because it is larger and may occupy conflict zones longer.
- Emergency vehicle: hard or very high priority.
- Ordinary car: normal priority.

### Proposed framework

1. **Intersection representation**
   - Cut the intersection into conflict zones.
   - Each conflict zone can be occupied by at most one vehicle at a time.

2. **Vehicle representation**
   - Each vehicle has type, priority, arrival time, speed, route, and trajectory.
   - Each trajectory is converted into a sequence of conflict zones.

3. **Scheduling formulation**
   - Vehicle = job.
   - Conflict zone = machine/resource.
   - Passing a conflict zone = operation.
   - Passing time = processing time.
   - Passing order = scheduling decision.

4. **Main constraints**
   - Collision-free conflict-zone occupation.
   - Vehicle trajectory order.
   - Same-lane no-overtaking.
   - Arrival-time constraint.
   - Blocking constraint.
   - Deadlock constraint.

5. **Priority-aware objective**
   - Instead of minimizing only total delay, minimize weighted delay:

   \[
   \min \sum_i w_i D_i
   \]

   where \(w_i\) is the vehicle priority and \(D_i\) is the delay of vehicle \(i\).

6. **Data-driven prediction**
   - Use trajectory data to estimate arrival time and conflict-zone occupation time.
   - This makes the scheduling model more realistic than using fixed assumptions.

7. **Receding-horizon scheduling**
   - At each time step, predict vehicles arriving in the next short horizon.
   - Build a temporary JSSP/resource-allocation problem.
   - Schedule vehicles.
   - Execute the first part of the decision.
   - Update and reschedule as new trajectory data arrives.

8. **Possible methods**
   - FCFS baseline.
   - Greedy scheduling baseline.
   - Constrained optimization / CP-SAT baseline.
   - PPO/RL scheduler.
   - Hybrid method: greedy at low density and RL at high density.

9. **Evaluation metrics**
   - Average delay.
   - Weighted delay.
   - Bus delay.
   - Truck delay.
   - Emergency vehicle delay.
   - Throughput.
   - Fairness.
   - Collision/deadlock count.
   - Computation time.

### Suggested first-step focus

The first step should be:

**Build a priority-aware constrained JSSP formulation for one autonomous intersection and compare it with equal-priority scheduling.**

After this is clear, reinforcement learning, data-driven prediction, and edge/federated learning can be added as extensions.

---

## 2. Email draft

**Subject:** Research idea: Priority-aware JSSP/RL framework for autonomous intersection management

Dear Professor [Name],

Following our recent discussion, I would like to summarize a possible research direction for autonomous intersection management.

The main idea is to model the intersection as a scheduling and resource-allocation problem. We can divide the intersection into several conflict zones and treat each conflict zone as a limited resource. Each vehicle is modeled as a job, and its trajectory through the intersection is represented as a sequence of conflict zones. In this way, the vehicle passing-order problem can be formulated as a constrained Job-Shop Scheduling Problem.

Based on the paper on reinforcement-learning-based JSSP for intelligent intersection management, I would like to extend the formulation by considering heterogeneous vehicles and priority-aware scheduling. For example, buses, trucks, emergency vehicles, and ordinary cars may have different priorities and different practical impacts. Therefore, instead of minimizing only total delay, we can design a weighted objective that considers vehicle priority, delay, fairness, and safety.

The planned framework is:

1. Build a conflict-zone graph of the intersection.
2. Convert each vehicle trajectory into a sequence of conflict-zone operations.
3. Formulate the problem as a priority-aware constrained JSSP.
4. Use trajectory data to predict arrival time and conflict-zone occupation time.
5. Apply receding-horizon scheduling so that the system can update decisions continuously.
6. Compare several methods, such as FCFS, greedy scheduling, optimization-based scheduling, and reinforcement learning.

The expected contribution could be a data-driven and priority-aware autonomous intersection manager that is more practical for heterogeneous traffic, especially when buses or trucks have greater influence on traffic efficiency.

For the first stage, I think it may be better to focus on a single intersection and first build the priority-aware JSSP formulation. After that, we can add the reinforcement learning scheduler and trajectory-data-driven prediction. The edge computing or federated learning part could be considered as a future extension.

I would appreciate your feedback on whether this direction is suitable. In particular, I would like to discuss whether the first contribution should focus on the priority-aware JSSP formulation, the reinforcement learning method, or the data-driven trajectory prediction component.

Best regards,

[Your Name]

---

## 3. Very short version for meeting discussion

**Idea:** Model autonomous intersection management as a priority-aware JSSP problem.

**Why:** Current JSSP/RL methods usually minimize total delay, but real traffic has heterogeneous vehicles. Buses, trucks, and emergency vehicles should not be treated exactly the same as ordinary cars.

**How:** Divide the intersection into conflict zones, convert vehicle trajectories into sequences of conflict-zone operations, and schedule the passing order using weighted delay and safety constraints.

**Method:** Start with constrained JSSP and greedy/optimization baselines, then add PPO/RL and receding-horizon prediction.

**Novelty:** Heterogeneous vehicle priority + weighted-delay objective + data-driven conflict-zone prediction + receding-horizon scheduling.

**First experiment:** One intersection, mixed vehicle types, compare equal-priority scheduling with priority-aware scheduling.
