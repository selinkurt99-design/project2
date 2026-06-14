# Architecture Overview — MPC Real-Time Loop

Compact flowchart for battery + ultracapacitor energy management MPC control loop.

```mermaid
flowchart TD
  B1(("START<br>k=0, Δt=10ms")) 
  B2["Data Acquisition<br>Read sensors, Kalman Filter<br>Estimate SoC, V_1"]
  B3["State Prediction<br>Horizon Np=20"]
  B4["Constraints<br>0.20≤SoC≤0.90<br>0.50≤SoC_sc≤1.00"]
  B5{Feasible?}
  B6["Relax<br>Constraints<br>Slack ε"]
  B7["Solve QP<br>u*=argmin J"]
  B8["Apply u(k)<br>Set I_b_ref, I_sc_ref"]
  B9["k=k+1<br>Wait 10ms"]

  B1 --> B2
  B2 --> B3
  B3 --> B4
  B4 --> B5
  B5 -- Yes --> B7
  B5 -- No --> B6
  B6 --> B7
  B7 --> B8
  B8 --> B9
  B9 -. Feedback .-> B2

  classDef states fill:#ECEFF1,stroke:#333,stroke-width:1px,font-size:11px
  classDef math fill:#DDEFFB,stroke:#333,stroke-width:1px,font-size:11px
  classDef decision fill:#FFF3D9,stroke:#333,stroke-width:1px,font-size:11px
  classDef opt fill:#E8F7E9,stroke:#333,stroke-width:1px,font-size:11px

  class B1,B2 states
  class B3,B4 math
  class B5,B6 decision
  class B7,B8,B9 opt
```

## Legend
- **Light Grey**: Perception & State Estimation
- **Light Blue**: Mathematical Operations
- **Light Amber**: Decision & Constraint Relaxation
- **Light Green**: Optimization & Actuation
- **Dashed line**: Feedback loop (10ms cycle)

## Key Components
| Block | Function |
|-------|----------|
| **B1** | Timer interrupt trigger (100 Hz) |
| **B2** | ADC sampling + Kalman filter observer |
| **B3** | State trajectory prediction (200 ms horizon) |
| **B4** | Physical constraint formulation |
| **B5** | QP feasibility check |
| **B6** | Soft constraint relaxation with slack variable |
| **B7** | Active-set QP solver (Nc=5) |
| **B8** | Receding horizon execution |
| **B9** | Horizon shift + wait for next interrupt |
