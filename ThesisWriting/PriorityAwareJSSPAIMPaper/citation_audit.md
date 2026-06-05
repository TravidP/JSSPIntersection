# Citation and Technical Audit

Date: 2026-06-04

## Reference Policy

The paper uses only central references that were verified from local files or reliable online metadata. It does not preserve every reference from the notes. Some references from the literature-review note are useful for future reading, but they are not all needed in the first formulation paper.

## Verified and Used

- Dresner and Stone (2008): reservation-based AIM; DOI `10.1613/jair.2502`.
- Chen and Englund (2016): cooperative intersection management survey; DOI `10.1109/TITS.2015.2471812`.
- Rios-Torres and Malikopoulos (2017): CAV coordination survey; DOI `10.1109/TITS.2016.2600504`.
- Khayatian et al. (2020): CAV intersection management survey; DOI `10.1145/3407903`.
- Abbas-Turki et al. (2023): AIM scheduling/trajectory review; DOI `10.3390/s23031509`. Author list is shortened as `and others` in BibTeX to avoid inventing incomplete names.
- Lin et al. (2019): graph-based AIM scheduling and verification; DOI `10.1145/3358221`.
- Huang et al. (2023): DATE JSSP-RL paper; DOI `10.23919/DATE56975.2023.10137280`. Local PDF text confirms graph-based AIM to constrained JSSP, blocking/deadlock/arrival constraints, PPO, and grouping strategy.
- Ahn and Del Vecchio (2016): JSSP collision-avoidance verification; DOI `10.1145/2883817.2883830`.
- Levin and Rey (2017): conflict-point MILP and rolling horizon; DOI `10.1016/j.trc.2017.09.025`.
- Fayazi and Vahidi (2018): MILP optimal intersection scheduling; DOI `10.1109/TIV.2018.2843163`.
- Soleimaniamiri and Li (2019): heterogeneous CAV scheduling at conflict areas; TRB paper `19-04176`.
- Li et al. (2023): priority-based search for autonomous intersection coordination; DOI `10.1609/aaai.v37i10.26368`.
- Guan et al. (2020): PPO CAV coordination; DOI `10.1109/TVT.2020.3026111`. The author names are corrected to Yangang Ren and Laiquan Luo.
- Wu et al. (2019): DCL-AIM; DOI `10.1016/j.trc.2019.04.012`.
- Zhang et al. (2020): DRL for JSSP dispatching; NeurIPS 2020.
- Park et al. (2021): GNN/RL for JSSP; arXiv `2106.01086`.
- Lin et al. (2015): transit signal priority review; DOI `10.1179/1942787514Y.0000000044`.
- Hui Zhang et al. (2021): priority-based AIM scheme; DOI `10.3390/vehicles3030032`.
- Salzmann et al. (2020): Trajectron++; ECCV 2020.
- Teeti et al. (2022): AV intention and trajectory prediction survey; DOI `10.24963/ijcai.2022/785`.

## Omitted or Demoted

- Yusuf Saltan's Bilkent thesis is not used as a core citation in this paper because the current manuscript focuses on single-intersection scheduling, not strategic network-level incentives. It can be mentioned in future work if the project expands toward mechanism design or route-choice incentives.
- Guney and Raptis (2020) is useful background, but the current draft already has enough verified optimization-based intersection scheduling citations. It can be added later if the method section needs more motion-coordination context.
- Unverified or weakly central references from the notes were not copied into `references.bib`.

## Technical Claim Checks

- The paper does not claim completed experiments or measured improvements.
- The paper does not claim that RL guarantees safety. Safety is attributed to constraints, action masks, and verification checks.
- Truck heterogeneity is not treated only as priority weight. The draft states that trucks should also affect processing time and headway.
- Emergency priority is framed as hard priority only when safety allows.
- The paper states that standard JSSP is commonly makespan-oriented, while this traffic formulation uses weighted delay.
- The paper separates high-level scheduling from low-level trajectory tracking and leaves low-level control as future implementation detail.

