# Literature Review Report: Priority-Aware Data-Driven JSSP for Autonomous Intersection Management

Generated from the local `Notes/` folder and a targeted literature search.  
Date: 2026-06-03

## 1. Executive Positioning

Your current research idea is strong and defensible if it is framed as:

> A priority-aware constrained Job-Shop Scheduling Problem (JSSP) formulation for autonomous intersection management (AIM), solved online in a receding-horizon workflow, with reinforcement learning used as a scalable scheduling-policy learner rather than as the only contribution.

The clearest novelty is not simply "using PPO for traffic." The stronger contribution is to combine four elements that are usually studied separately:

1. Conflict-zone/resource modeling of an intersection.
2. JSSP-style scheduling with intersection-specific constraints such as blocking, same-lane order, release times, and deadlock avoidance.
3. Heterogeneous vehicle priority using weighted delay, person-delay, value-of-time, or emergency/service priority.
4. Online data-driven scheduling using predicted arrival times, zone occupation times, and receding-horizon re-planning.

This positioning gives you a good bridge between the DATE 2023 JSSP-RL paper, classical reservation-based AIM, optimization-based CAV intersection scheduling, and transport-priority literature such as transit signal priority and emergency vehicle priority.

**Chinese summary:**  
你的研究最强的定位不是“把 PPO 用到交通里”，而是“把交叉口冲突区建模、约束 JSSP、异构车辆优先级、轨迹预测和滚动时域调度”组合起来。这样 novelty 更清楚，也更容易和现有 AIM、MILP、RL、公交优先和紧急车辆优先文献区分开。

## 2. Review of Your Local Notes

The local notes already contain a coherent research direction:

- `PaperAndMeetingIdea.md` clearly explains the mapping from AIM to JSSP: vehicle = job, conflict zone = machine/resource, zone traversal = operation, traversal time = processing time, and passing order = schedule.
- `01_detailed_research_design_priority_aware_jssp_aim.md` already has the right technical backbone: weighted delay objective, release-time constraints, conflict-zone capacity, lane FIFO, blocking, deadlock prevention, action masking, PPO/GNN as a candidate policy, and practical baselines.
- `02_brief_map_plan_and_email_to_supervisor.md` and `SupervisorEmailMap.md` are good for supervisor communication because they present the idea at the correct abstraction level.
- `ResearchPlan.md` is already close to a first thesis proposal skeleton.

Recommended tightening:

1. Separate the high-level scheduling problem from the low-level vehicle trajectory control problem. The first paper or thesis stage can focus on scheduling/order/time-window allocation; detailed MPC trajectory tracking can be a controlled assumption or later extension.
2. Define priority weights with a transport interpretation. For buses, use person-delay or passenger occupancy; for trucks, use longer occupancy/headway or operational impact; for emergency vehicles, decide whether the priority is a hard constraint or a very high weight.
3. Make fairness explicit. Weighted delay can starve ordinary cars under high-priority demand, so add maximum-wait constraints or a fairness penalty.
4. Treat exact optimization as a benchmark, not the main real-time method. MILP/CP-SAT can validate small instances; greedy, FCFS, priority-greedy, and RL can handle online streams.
5. Keep edge computing and federated learning as future work unless the supervisor specifically asks for distributed deployment.

**Chinese summary:**  
你的 Notes 已经很完整，尤其是 JSSP 映射、约束、weighted delay、receding horizon 和 baseline 都有了。下一步需要把贡献收窄：第一阶段重点做“priority-aware constrained JSSP + online scheduling”，低层轨迹控制、edge/federated learning 先放到 future work。还要补强权重解释和公平性约束。

## 3. Literature Map

### 3.1 Reservation-Based AIM Foundations

The foundational AIM literature starts with reservation-based intersection control. Dresner and Stone proposed a multiagent reservation approach in which vehicles and intersections communicate to allocate space-time reservations. Their 2008 JAIR paper is especially important because it shows that autonomous intersections can outperform traffic lights/stop signs in simulation and already discusses mixed traffic and emergency-vehicle priority.

For your project, this literature establishes the big picture: an intersection can be treated as a scarce space-time resource allocator. However, early reservation systems often use request/accept protocols or FCFS-style ordering, which can be inefficient under dense traffic. This creates room for optimization, scheduling, and learning-based dispatching.

Key sources:

- [Dresner and Stone, 2008, A Multiagent Approach to Autonomous Intersection Management](https://www.cs.utexas.edu/~pstone/Papers/bib2html/b2hd-JAIR08-dresner.html)
- [Chen and Englund, 2016, Cooperative Intersection Management: A Survey](https://ri.diva-portal.org/smash/record.jsf?pid=diva2%3A1064135)
- [Khayatian et al., 2020, A Survey on Intersection Management of Connected Autonomous Vehicles](https://mpslab-asu.github.io/publications/papers/Khayatian2020TCPS.pdf)
- [Abbas-Turki et al., 2023, Autonomous Intersection Management: Optimal Trajectories and Efficient Scheduling](https://www.mdpi.com/1424-8220/23/3/1509)

**Chinese summary:**  
早期 AIM 文献主要是 reservation-based intersection management：车辆向交叉口管理器申请空间-时间资源。它证明了无信号交叉口的潜力，但 FCFS/request-accept 机制在高密度下可能效率不足。因此，你的 JSSP/RL/weighted scheduling 可以被解释为对 reservation AIM 的更强调度层。

### 3.2 Conflict-Zone and Graph-Based Scheduling

Graph-based AIM is the most direct conceptual parent of your work. Lin et al. model an intersection as a graph of conflict relations and provide formal verification for collision-freeness and deadlock-freeness. This is important because your JSSP formulation must remain physically meaningful: a learned policy is not acceptable unless infeasible actions are masked or rejected by safety/deadlock checks.

The DATE 2023 paper by Huang et al. is the direct base paper for your notes. It defines the intersection scheduling problem using a graph-based model, transforms it into constrained JSSP, models scheduling as an MDP, trains PPO, and uses grouping for vehicle streams. The key reported insight is that learning-based scheduling is especially useful under high traffic density.

Your extension should be described as:

> From graph-based constrained JSSP-RL for homogeneous vehicle delay minimization to priority-aware, heterogeneous, data-driven, receding-horizon constrained JSSP.

Key sources:

- [Lin et al., 2019, Graph-Based Modeling, Scheduling, and Verification for Intersection Management of Intelligent Vehicles](https://www.ri.cmu.edu/publications/graph-based-modeling-scheduling-and-verification-for-intersection-management-of-intelligent-vehicles/)
- [Huang et al., 2023, Reinforcement-Learning-Based Job-Shop Scheduling for Intelligent Intersection Management](https://scholars.lib.ntu.edu.tw/entities/publication/93cd76a3-9212-4103-8c26-e2716928f3e4)
- [Ahn and Del Vecchio, 2016, Semi-autonomous Intersection Collision Avoidance through Job-shop Scheduling](https://arxiv.org/abs/1510.07026)

**Chinese summary:**  
Graph-based AIM 和 DATE 2023 是你的直接根基。Lin et al. 提供冲突图和 deadlock-freeness 验证；Huang et al. 把 graph-based AIM 转成 constrained JSSP 并用 PPO。你的贡献可以写成：在这个框架上加入异构车辆优先级、weighted delay、数据驱动预测和滚动时域。

### 3.3 Optimization-Based Scheduling and Rolling Horizon

MILP and MIP formulations are important because they give exact or near-exact benchmarks. Levin and Rey propose a conflict-point formulation and a rolling-horizon algorithm for AIM reservations. Fayazi and Vahidi formulate autonomous vehicle intersection crossing as a MILP that assigns optimal intersection access times. Soleimaniamiri and Li explicitly consider heterogeneous vehicle headways and values of time at a general conflict area, which is especially relevant for your priority-weighted direction.

The lesson from this cluster is practical:

- Exact optimization is useful for small instances and validation.
- It becomes computationally expensive as the number of vehicles, lanes, and conflict points grows.
- Rolling horizon is a natural compromise because it solves a small current problem repeatedly.
- Heterogeneous headways and values of time support your claim that vehicle classes should not be treated equally.

Key sources:

- [Levin and Rey, 2017, Conflict-point formulation of intersection control for autonomous vehicles](https://experts.umn.edu/en/publications/conflict-point-formulation-of-intersection-control-for-autonomous/)
- [Fayazi and Vahidi, 2018, Mixed-Integer Linear Programming for Optimal Scheduling of Autonomous Vehicle Intersection Crossing](https://trid.trb.org/View/1539530)
- [Fayazi, Vahidi, and Luckow, 2017, Optimal scheduling of autonomous vehicle arrivals at intelligent intersections via MILP](https://dblp.org/rec/conf/amcc/FayaziVL17)
- [Soleimaniamiri and Li, 2019, Scheduling of Heterogeneous Connected Automated Vehicles at a General Conflict Area](https://trid.trb.org/view/1573076)
- [Li et al., 2023, Intersection Coordination with Priority-Based Search for Autonomous Vehicles](https://ojs.aaai.org/index.php/AAAI/article/view/26368)

**Chinese summary:**  
MILP/MIP 文献说明交叉口调度可以做成严格优化模型，但实时扩展性有限。因此你应该把 MILP/CP-SAT 用作小规模 benchmark，把 rolling horizon、heuristics 和 RL 用作在线调度。Soleimaniamiri and Li 的 heterogeneous headways 和 values of time 对你的优先级权重非常关键。

### 3.4 Reinforcement Learning for AIM

RL for AIM is already established, but the research is not yet settled. Guan et al. use PPO for centralized CAV cooperation and compare against MPC in simulation. Wu et al. propose DCL-AIM, a decentralized multi-agent learning approach that decomposes state information and adapts coordination needs. Huang et al. apply RL specifically to the graph-based intersection model transformed into JSSP.

For your project, RL should be justified by scalability and repeated online decisions, not by novelty alone. A good phrasing is:

> RL is used to learn a feasible dispatching policy for repeated constrained scheduling instances, while safety remains enforced by action masks, resource constraints, and deadlock checks.

This also protects the work from the common criticism that RL is hard to verify. The RL policy proposes scheduling actions; the environment verifies them.

Key sources:

- [Guan et al., 2020, Centralized Cooperation for Connected and Automated Vehicles at Intersections by Proximal Policy Optimization](https://arxiv.org/abs/1912.08410)
- [Wu, Chen, and Zhu, 2019, DCL-AIM](https://colab.ws/articles/10.1016/j.trc.2019.04.012)
- [Huang et al., 2023, JSSP-RL for Intelligent Intersection Management](https://scholars.lib.ntu.edu.tw/entities/publication/93cd76a3-9212-4103-8c26-e2716928f3e4)

**Chinese summary:**  
RL 在 AIM 里已经有人做，所以你的论文不要只说“我用 RL”。更好的说法是：RL 学习一个在线调度策略，但安全由 action masking、资源约束和 deadlock check 保证。这样 RL 是可扩展调度器，而不是唯一的安全来源。

### 3.5 RL/GNN for JSSP and Dispatching Rules

The JSSP learning literature supports your method choice. Zhang et al. learn dispatching rules for JSSP with a graph neural network and reinforcement learning. Park et al. also formulate JSSP scheduling as a sequential decision problem over a graph and train a GNN policy with PPO. These works support your plan to encode the conflict-zone/JSSP state as a graph, because both JSSP and conflict-zone AIM naturally have operation-resource precedence structures.

Important transfer idea:

- A vehicle's route through zones is a precedence chain.
- Shared zones create disjunctive resource conflicts.
- Same-lane FIFO adds extra precedence edges.
- The graph encoder can represent remaining operations, resource availability, vehicle priority, and predicted completion times.

The research gap is that generic JSSP papers usually optimize manufacturing objectives such as makespan or total completion time, not traffic-specific weighted delay with blocking, deadlock, lane order, and uncertain arrivals.

Key sources:

- [Zhang et al., 2020, Learning to Dispatch for Job Shop Scheduling via Deep Reinforcement Learning](https://papers.nips.cc/paper_files/paper/2020/hash/11958dfee29b6709f48a9ba0387a2431-Abstract.html)
- [Park et al., 2021, Learning to schedule job-shop problems: Representation and policy learning using GNN and RL](https://arxiv.org/abs/2106.01086)
- [Vepsalainen and Morton, 1987, Priority Rules for Job Shops with Weighted Tardiness Costs](https://pubsonline.informs.org/doi/10.1287/mnsc.33.8.1035)

**Chinese summary:**  
JSSP-GNN/RL 文献能支撑你使用图神经网络和 PPO。JSSP 本身有 precedence chain 和 machine conflict，和车辆路线/冲突区很像。但普通 JSSP 没有交通里的 blocking、deadlock、same-lane order、arrival uncertainty 和车辆优先级，所以你的 domain-specific formulation 仍然有贡献。

### 3.6 Heterogeneous Vehicle Priority

Your priority-aware contribution is promising because the literature already recognizes priority in several separate forms:

- Dresner and Stone discuss emergency-vehicle priority in reservation AIM.
- Transit signal priority gives buses special treatment because a bus carries many passengers and can improve person throughput.
- Priority-based AIM schemes consider emergency levels, vehicle size, predicted occupation time, and conflict points.
- Heterogeneous CAV scheduling work considers headways and values of time.

This supports a soft-priority objective:

\[
\min \sum_i w_i D_i
\]

where \(D_i = C_i - r_i - P_i\) and \(w_i\) can represent passenger occupancy, emergency level, value of time, vehicle type, or operational impact.

Recommended priority design:

| Vehicle class | Suggested modeling role | Suggested priority treatment |
| --- | --- | --- |
| Passenger car | Baseline road user | \(w_i = 1.0\) |
| Truck | Longer length, slower acceleration, larger blocking/headway | Higher processing time/headway, optional \(w_i > 1\) |
| Bus | High person-delay impact | Weight proportional to passenger occupancy or service class |
| Emergency/service vehicle | Safety-critical response | Hard priority if possible; otherwise very high weight plus fairness constraints |

Do not only increase weights for trucks. Trucks may be better modeled through physical processing time and safety headway, because their main difference is often size and dynamics rather than social priority. Buses are naturally weighted by person-delay. Emergency vehicles may require hard constraints.

Key sources:

- [Dresner and Stone, 2008](https://www.cs.utexas.edu/~pstone/Papers/bib2html/b2hd-JAIR08-dresner.html)
- [FTA Signal Priority overview](https://www.transit.dot.gov/research-innovation/signal-priority)
- [Lin et al., 2015, Transit signal priority control at signalized intersections: a comprehensive review](https://doi.org/10.1179/1942787514Y.0000000044)
- [A Priority-Based AIM Scheme for CAVs, 2021](https://www.mdpi.com/2624-8921/3/3/32)
- [Soleimaniamiri and Li, 2019](https://trid.trb.org/view/1573076)

**Chinese summary:**  
异构优先级是你的核心 novelty。公交优先可以用 person-delay 支撑，紧急车辆可以用 hard priority 支撑，卡车最好同时用更长 processing time/headway 来建模，而不只是加权。这样你的权重不是随便设的，而是有交通含义。

### 3.7 Trajectory Prediction and Receding-Horizon Scheduling

Trajectory prediction literature supports the data-driven part of your plan. Modern prediction models emphasize multi-agent interaction, maps, dynamics, and uncertainty. For a first implementation, you do not need a full deep prediction model. You can start with a kinematic baseline and then test robustness under noisy predicted arrival and processing times.

Suggested staged approach:

1. Perfect information: use known arrival time, movement, and processing time.
2. Kinematic prediction: estimate arrival time from position, speed, acceleration, lane, and intended movement.
3. Noisy prediction: add controlled noise to arrival/processing times and evaluate robustness.
4. Learning-based prediction: use LSTM/GNN/Trajectron-style methods only if data and time allow.

The receding-horizon structure is the right way to connect prediction with scheduling. At every cycle, build a finite constrained-JSSP instance, schedule the near-future vehicles, execute only the immediate decisions, and re-plan as new vehicles enter or predictions change.

Key sources:

- [Trajectron++, ECCV 2020](https://www.ecva.net/papers/eccv_2020/papers_ECCV/html/3094_ECCV_2020_paper.php)
- [Teeti et al., 2022, Vision-based Intention and Trajectory Prediction in Autonomous Vehicles: A Survey](https://www.ijcai.org/proceedings/2022/785)
- [Levin and Rey, 2017, rolling-horizon conflict-point AIM](https://experts.umn.edu/en/publications/conflict-point-formulation-of-intersection-control-for-autonomous/)
- [Khayatian et al., 2020, AIM survey](https://mpslab-asu.github.io/publications/papers/Khayatian2020TCPS.pdf)

**Chinese summary:**  
轨迹预测部分不要一开始做太复杂。第一版可以用 perfect information 和 kinematic prediction，再加入噪声测试鲁棒性。深度预测模型可以放到后续。滚动时域是连接预测和在线调度的关键：短窗口内求解，执行近端决策，然后不断更新。

## 4. Research Gap and Novelty Statement

A clean novelty statement for your proposal:

> Existing AIM work has studied reservation mechanisms, graph-based safety verification, MILP scheduling, and RL-based dispatching. However, fewer works combine graph/JSSP-style intersection scheduling with explicit heterogeneous vehicle priority, fairness-aware weighted delay, trajectory-data-driven processing-time prediction, and online receding-horizon execution. This project fills that gap by formulating a priority-aware constrained JSSP for CAV intersection management and evaluating RL/hybrid policies against interpretable scheduling and optimization baselines.

More precise contribution claims:

1. A priority-aware constrained JSSP formulation for single-intersection AIM.
2. A weighted-delay objective with defensible vehicle-class meanings: person-delay, value of time, emergency priority, and physical occupancy/headway.
3. A receding-horizon workflow that updates schedules from trajectory predictions.
4. A safety-aware learning environment with action masks for release time, conflict-zone capacity, trajectory order, lane order, blocking, and deadlock.
5. A comparison of FCFS, greedy, priority-greedy, deadlock-aware greedy, exact small-instance optimization, and PPO/GNN policy.

**Chinese summary:**  
你的 novelty 可以写成：现有 AIM 有 reservation、graph verification、MILP 和 RL，但很少把“约束 JSSP + 异构车辆优先级 + fairness-aware weighted delay + 轨迹预测 + rolling horizon”放在一个框架里系统评估。这个说法比单纯“用 RL”更有说服力。

## 5. Recommended Problem Formulation

Use a two-layer formulation.

### Layer 1: Scheduling Layer

Inputs:

- Vehicle \(i\): \(x_i = (r_i, t_i, class_i, w_i, l_i, v_i)\)
- Trajectory: \(T_i = (z_{i,1}, z_{i,2}, ..., z_{i,n_i})\)
- Processing time: \(p_{i,k}\)
- Safety headway: \(h_{ij}\)

Decision variables:

- Operation start time \(s_{i,k}\)
- Completion time \(c_{i,k} = s_{i,k} + p_{i,k}\)
- Binary ordering variable for conflicting operations

Core constraints:

- Release time: \(s_{i,1} \ge r_i\)
- Trajectory order: \(s_{i,k+1} \ge c_{i,k}\)
- Conflict-zone capacity with headway
- Same-lane FIFO/no-overtaking
- Blocking/no-disappearing vehicle behavior
- Deadlock avoidance by graph cycle checks or action masking

Objective:

\[
\min \sum_i w_i (C_i - r_i - P_i) + \lambda \sum_i \max(0, D_i - D_{\max})
\]

where \(P_i = \sum_k p_{i,k}\) and the second term discourages starvation.

### Layer 2: Motion/Execution Layer

The scheduling layer outputs target crossing order or time windows. Vehicles then track the assigned timing using a simple speed-planning rule or MPC. For the first research stage, this can be simplified: assume vehicles can safely track feasible assigned time windows.

**Chinese summary:**  
建议把模型分成两层：上层做调度，决定冲突区占用顺序和时间窗；下层做车辆速度/轨迹执行。第一阶段可以简化下层，重点证明上层 priority-aware constrained JSSP 的价值。

## 6. Baselines and Evaluation Plan

Recommended baselines:

| Baseline | Why it matters |
| --- | --- |
| Fixed signal | Traditional reference point |
| FCFS reservation | Standard AIM baseline |
| Greedy earliest completion | Fast and interpretable |
| Priority-weighted greedy | Tests whether simple priority rules are enough |
| Deadlock-aware greedy/iGreedy | Strong non-learning baseline |
| MILP/CP-SAT for small horizons | Measures optimality gap |
| Original PPO-JSSP without priority | Direct comparison to base paper |
| Proposed priority-aware PPO/GNN | Main learning method |
| Hybrid greedy/RL | Useful if RL only helps at high density |

Evaluation metrics:

- Average delay.
- Weighted total delay.
- Delay by vehicle class.
- Person-delay for buses.
- Emergency vehicle delay or deadline violations.
- Maximum delay and fairness index.
- Throughput.
- Queue length.
- Number of stops or braking events.
- Collision count: must be zero.
- Deadlock count: must be zero.
- Runtime per horizon.
- Robustness under prediction noise.
- Schedule stability between horizons.

Suggested hypotheses:

1. Priority-aware scheduling reduces weighted delay and bus/emergency delay compared with equal-priority scheduling.
2. Under low traffic density, simple greedy baselines may match or beat RL because conflicts are sparse.
3. Under high density and heterogeneous traffic, PPO/GNN or hybrid scheduling should outperform FCFS and simple greedy.
4. Fairness penalties prevent excessive delay for ordinary vehicles with only moderate loss in priority-service quality.
5. Receding-horizon scheduling with safety margins remains feasible under arrival-time and processing-time prediction errors.

**Chinese summary:**  
实验不要只比较 RL 和 FCFS。需要 fixed signal、FCFS、greedy、priority-greedy、deadlock-aware greedy、small-instance exact solver、原始 PPO-JSSP 和你的 priority-aware PPO/GNN。指标也要覆盖 average delay、weighted delay、class-specific delay、公平性、runtime、deadlock/collision 和 prediction noise robustness。

## 7. Literature Matrix

| Theme | Representative literature | Main contribution | Relevance to your project | Remaining gap |
| --- | --- | --- | --- | --- |
| Reservation AIM | Dresner and Stone (2008) | Space-time reservations, multiagent AIM, emergency priority | Establishes AIM as resource allocation | Less focused on JSSP, graph constraints, and learning |
| AIM surveys | Chen and Englund (2016); Khayatian et al. (2020); Abbas-Turki et al. (2023) | Taxonomy of cooperative/AIM methods, safety, scheduling, validation | Provides background and evaluation categories | Surveys do not provide your specific formulation |
| Graph AIM | Lin et al. (2019) | Conflict graph, scheduling, deadlock verification | Directly supports conflict-zone graph and action masks | No priority-aware weighted JSSP/RL |
| JSSP-RL AIM | Huang et al. (2023) | Converts graph AIM to constrained JSSP and trains PPO | Your direct base paper | Heterogeneous priorities and prediction are not central |
| Earlier JSSP intersection | Ahn and Del Vecchio (2016) | Collision verification via JSSP/MILP | Shows JSSP is natural for conflict points | Focused on supervision/verification, not priority online control |
| Conflict-point MILP | Levin and Rey (2017); Fayazi and Vahidi (2018) | Exact optimization and rolling horizon | Strong benchmark and scheduling formalism | Scalability and priority/fairness integration |
| Heterogeneous CAV scheduling | Soleimaniamiri and Li (2019) | Headways and values of time | Supports vehicle heterogeneity and weighted objective | Not graph/JSSP-RL focused |
| Priority/search AIM | Li et al. (2023) | Priority-based search and MAPF-style real-time coordination | Strong non-RL comparison | Different algorithmic framing; not weighted JSSP |
| RL for AIM | Guan et al. (2020); Wu et al. (2019) | PPO and multi-agent learning for intersection coordination | Supports RL as scalable policy | Needs hard safety/action-mask integration |
| RL/GNN for JSSP | Zhang et al. (2020); Park et al. (2021) | Learned dispatching over graph/JSSP states | Supports GNN/PPO scheduling model | Needs traffic-specific constraints and weighted delay |
| Vehicle priority | FTA; Lin et al. TSP review; Priority-Based AIM Scheme | Bus/emergency priority motivation | Supports weights and hard priority cases | Mostly signalized or rule-based, not JSSP |
| Prediction | Trajectron++; Teeti et al. survey | Multi-agent trajectory/intention prediction | Supports data-driven arrivals and processing times | Full prediction may be too broad for first stage |

**Chinese summary:**  
文献矩阵显示你的工作正好位于几个方向的交叉处：reservation AIM、graph verification、MILP scheduling、RL/JSSP、priority control 和 trajectory prediction。你的 gap 是把它们统一成一个 priority-aware constrained JSSP online scheduling framework。

## 8. Suggested Paper/Thesis Outline

1. Introduction
   - Motivation: signal-free CAV intersections, safety and efficiency.
   - Gap: existing JSSP/RL AIM does not fully handle heterogeneous vehicle priority and online prediction.
   - Contributions.

2. Related Work
   - Reservation-based AIM.
   - Graph/conflict-zone scheduling and verification.
   - Optimization/MILP/rolling horizon.
   - RL for AIM and RL/GNN for JSSP.
   - Priority-aware traffic control and trajectory prediction.

3. System Model
   - Intersection zones, vehicle classes, trajectory sequences, prediction inputs.

4. Priority-Aware Constrained JSSP Formulation
   - Variables, constraints, weighted-delay objective, fairness term.

5. Online Receding-Horizon Scheduler
   - Horizon construction, grouping, update cycle, prediction handling.

6. Algorithms
   - FCFS, greedy, priority-greedy, deadlock-aware greedy, small-instance exact solver, PPO/GNN scheduler.

7. Experiments
   - Scenario design, traffic density, vehicle mix, priority weights, prediction noise.

8. Results and Discussion
   - Efficiency, priority performance, fairness, runtime, robustness, when RL helps.

9. Conclusion and Future Work
   - Multi-intersection, edge deployment, federated learning, richer trajectory prediction.

**Chinese summary:**  
论文结构建议从 AIM 背景和 gap 开始，然后写系统模型、priority-aware constrained JSSP、rolling-horizon scheduler、算法和实验。edge/federated learning 最好放到 conclusion/future work，不要抢第一阶段主线。

## 9. Near-Term Work Plan

Recommended next steps:

1. Define one four-way intersection and conflict-zone graph.
2. Generate synthetic vehicle streams with car/bus/truck/emergency classes.
3. Implement schedule feasibility checks: release time, zone conflict, lane FIFO, blocking, and deadlock.
4. Implement FCFS, greedy, and priority-greedy first.
5. Implement small-instance exact optimization for validation.
6. Add receding-horizon grouping and noisy prediction experiments.
7. Add PPO/GNN only after baselines and feasibility checks are reliable.
8. Write the related-work section using the clusters in this report.

**Chinese summary:**  
先不要马上训练 RL。第一步应该是一个小交叉口、车辆生成器、约束检查器和几个 baseline。等 priority-aware JSSP 和 feasibility 都稳定后，再加入 PPO/GNN。这样风险更低，也更容易向导师展示阶段性成果。

## 10. Reference List

1. Kurt Dresner and Peter Stone. "A Multiagent Approach to Autonomous Intersection Management." Journal of Artificial Intelligence Research, 31:591-656, 2008. <https://www.cs.utexas.edu/~pstone/Papers/bib2html/b2hd-JAIR08-dresner.html>
2. Lei Chen and Cristofer Englund. "Cooperative Intersection Management: A Survey." IEEE Transactions on Intelligent Transportation Systems, 17(2):570-586, 2016. <https://ri.diva-portal.org/smash/record.jsf?pid=diva2%3A1064135>
3. Jackeline Rios-Torres and Andreas A. Malikopoulos. "A Survey on the Coordination of Connected and Automated Vehicles at Intersections and Merging at Highway On-Ramps." IEEE Transactions on Intelligent Transportation Systems, 18(5):1066-1077, 2017. <https://doi.org/10.1109/TITS.2016.2600504>
4. Mohammad Khayatian et al. "A Survey on Intersection Management of Connected Autonomous Vehicles." ACM Transactions on Cyber-Physical Systems, 2020. <https://mpslab-asu.github.io/publications/papers/Khayatian2020TCPS.pdf>
5. Abdeljalil Abbas-Turki et al. "Autonomous Intersection Management: Optimal Trajectories and Efficient Scheduling." Sensors, 23(3):1509, 2023. <https://www.mdpi.com/1424-8220/23/3/1509>
6. Yi-Ting Lin et al. "Graph-Based Modeling, Scheduling, and Verification for Intersection Management of Intelligent Vehicles." ACM Transactions on Embedded Computing Systems, 18(5), 2019. <https://www.ri.cmu.edu/publications/graph-based-modeling-scheduling-and-verification-for-intersection-management-of-intelligent-vehicles/>
7. Shao-Ching Huang et al. "Reinforcement-Learning-Based Job-Shop Scheduling for Intelligent Intersection Management." DATE 2023. DOI: 10.23919/DATE56975.2023.10137280. <https://scholars.lib.ntu.edu.tw/entities/publication/93cd76a3-9212-4103-8c26-e2716928f3e4>
8. Heejin Ahn and Domitilla Del Vecchio. "Semi-autonomous Intersection Collision Avoidance through Job-shop Scheduling." HSCC 2016 / arXiv. <https://arxiv.org/abs/1510.07026>
9. Michael W. Levin and David Rey. "Conflict-point formulation of intersection control for autonomous vehicles." Transportation Research Part C, 85:528-547, 2017. <https://experts.umn.edu/en/publications/conflict-point-formulation-of-intersection-control-for-autonomous/>
10. Seyed Alireza Fayazi and Ardalan Vahidi. "Mixed-Integer Linear Programming for Optimal Scheduling of Autonomous Vehicle Intersection Crossing." IEEE Transactions on Intelligent Vehicles, 3(3):287-299, 2018. <https://trid.trb.org/View/1539530>
11. Seyed Alireza Fayazi, Ardalan Vahidi, and Andre Luckow. "Optimal Scheduling of Autonomous Vehicle Arrivals at Intelligent Intersections via MILP." ACC 2017. <https://dblp.org/rec/conf/amcc/FayaziVL17>
12. Saeid Soleimaniamiri and Xiaopeng Li. "Scheduling of Heterogeneous Connected Automated Vehicles at a General Conflict Area." TRB Annual Meeting, 2019. <https://trid.trb.org/view/1573076>
13. Jiaoyang Li et al. "Intersection Coordination with Priority-Based Search for Autonomous Vehicles." AAAI 2023. <https://ojs.aaai.org/index.php/AAAI/article/view/26368>
14. Yang Guan et al. "Centralized Cooperation for Connected and Automated Vehicles at Intersections by Proximal Policy Optimization." IEEE Transactions on Vehicular Technology, 2020. <https://arxiv.org/abs/1912.08410>
15. Yuanyuan Wu, Haipeng Chen, and Feng Zhu. "DCL-AIM: Decentralized Coordination Learning of Autonomous Intersection Management for Connected and Automated Vehicles." Transportation Research Part C, 103:246-260, 2019. <https://colab.ws/articles/10.1016/j.trc.2019.04.012>
16. Cong Zhang et al. "Learning to Dispatch for Job Shop Scheduling via Deep Reinforcement Learning." NeurIPS 2020. <https://papers.nips.cc/paper_files/paper/2020/hash/11958dfee29b6709f48a9ba0387a2431-Abstract.html>
17. Junyoung Park et al. "Learning to Schedule Job-Shop Problems: Representation and Policy Learning Using Graph Neural Network and Reinforcement Learning." International Journal of Production Research, 2021. <https://arxiv.org/abs/2106.01086>
18. Ari P. J. Vepsalainen and Thomas E. Morton. "Priority Rules for Job Shops with Weighted Tardiness Costs." Management Science, 1987. <https://pubsonline.informs.org/doi/10.1287/mnsc.33.8.1035>
19. U.S. Federal Transit Administration. "Signal Priority." <https://www.transit.dot.gov/research-innovation/signal-priority>
20. Y. Lin, X. Yang, N. Zou, and M. Franz. "Transit signal priority control at signalized intersections: a comprehensive review." Transportation Letters, 2015. <https://doi.org/10.1179/1942787514Y.0000000044>
21. "A Priority-Based Autonomous Intersection Management Scheme for Connected Automated Vehicles." Vehicles, 2021. <https://www.mdpi.com/2624-8921/3/3/32>
22. Tim Salzmann et al. "Trajectron++: Dynamically-Feasible Trajectory Forecasting With Heterogeneous Data." ECCV 2020. <https://www.ecva.net/papers/eccv_2020/papers_ECCV/html/3094_ECCV_2020_paper.php>
23. Izzeddin Teeti et al. "Vision-based Intention and Trajectory Prediction in Autonomous Vehicles: A Survey." IJCAI 2022. <https://www.ijcai.org/proceedings/2022/785>

**Chinese summary:**  
参考文献覆盖了 AIM 基础、综述、graph-based verification、JSSP/RL、MILP/rolling horizon、异构车辆优先级、公交/紧急车辆优先和轨迹预测。建议后续写 related work 时按这些类别组织，而不是按年份简单罗列。
