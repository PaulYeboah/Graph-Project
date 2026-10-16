# S4 – Shared Notation Sheet
**Project:** Time-dependent delivery routing (CesiCDP, ADEME call) – **Problem name:** TD-CVRP-TW
*Single reference for all team members: every notebook section, the ILP and the code must use exactly these symbols.*

## 1. Conventions
- Sets: calligraphic or capital letters ($V, A, C, N, \mathcal T, \mathcal A$). Parameters: lowercase/Latin ($q_i, s_i, d_a$). Decision variables: $x, y, f, g$.
- Node `0` is always the depot. Customers are `1..n`.
- Time unit: **minute**. Distance unit: **km**. Speed unit: **km/min**. Demand unit: **parcel units** (or kg, to be fixed per instance).
- Continuous time is written $t$ (minutes); discretized time is an integer index $t\in\{0,\dots,H\}$ meaning the instant $t\delta$. Always say which one is used.
- Python variable names are given in the last column of each table so code and maths stay aligned.

## 2. Road network and traffic
| Symbol | Meaning | Python |
|---|---|---|
| $G=(V,A)$ | directed road network | `G` |
| $V$, $A$ | nodes (intersections, customer locations, depot) and arcs (road segments) | `V`, `A` |
| $a=(u,v)\in A$ | arc from $u$ to $v$ | `a` |
| $d_a>0$ | length of arc $a$ (km) | `length[a]` |
| $T$ | planning horizon end (min); horizon is $[0,T]$ | `T` |
| $K$ | number of time slots | `K` |
| $\beta_0=0<\beta_1<\dots<\beta_K=T$ | slot breakpoints (min) | `breakpoints` |
| $s\in\{1,\dots,K\}$ | slot index (slot $s$ = $[\beta_{s-1},\beta_s)$) | `s` |
| $v_a^{s}>0$ | speed on arc $a$ during slot $s$ (km/min) | `speed[a][s]` |
| $\tau_a(t)$ | travel time on arc $a$ when entering at time $t$ (min) | `tau_arc(a,t)` |
| $\alpha_a(t)=t+\tau_a(t)$ | arrival time on arc $a$ when entering at $t$ | `arrival_arc(a,t)` |
| FIFO | $t\mapsto\alpha_a(t)$ strictly increasing | – |

## 3. Customers, depot and fleet
| Symbol | Meaning | Python |
|---|---|---|
| $0$ | depot | `0` |
| $C\subset V\setminus\{0\}$, $n=\lvert C\rvert$ | set and number of customers to serve | `C`, `n` |
| $N=C\cup\{0\}$ | decision nodes | `N` |
| $q_i>0$ ($q_0=0$) | demand of customer $i$ | `demand[i]` |
| $s_i\ge0$ ($s_0=0$) | service duration at $i$ (min) | `service[i]` |
| $[a_i,b_i]\subseteq[0,T]$ | time window for the **start** of service at $i$ | `tw[i]` |
| $k$ | maximum number of trucks | `k` |
| $Q$ | capacity of each truck (identical), $q_i\le Q$ | `Q` |

## 4. Customer graph (reduced graph)
| Symbol | Meaning | Python |
|---|---|---|
| $H=(N,E)$ | complete directed graph on $N$ | `H` |
| $\tau_{ij}(t)$ | minimum travel time from $i$ to $j$ when leaving at $t$ (time-dependent shortest path in $G$) | `tau(i,j,t)` |
| $\alpha_{ij}(t)=t+\tau_{ij}(t)$ | earliest arrival at $j$ | `arrival(i,j,t)` |

## 5. Solution (route-based notation)
| Symbol | Meaning |
|---|---|
| $r\le k$ | number of routes used |
| $R_\ell=(0,i^\ell_1,\dots,i^\ell_{m_\ell},0)$ | route $\ell$, with $m_\ell$ customers |
| $t_0^\ell$ | departure time of route $\ell$ from the depot (decision) |
| $u_p^\ell$ | start of service of the $p$-th customer of route $\ell$ |
| $u^\ell_{m_\ell+1}$ | return time to the depot |
| $D(R_\ell)=u^\ell_{m_\ell+1}-t_0^\ell$ | duration of route $\ell$ |
| **Objective** | $\min\sum_{\ell=1}^r D(R_\ell)$ |

## 6. ILP (time-expanded formulation)
| Symbol | Meaning |
|---|---|
| $\delta$ | time step (min) |
| $H=\lfloor T/\delta\rfloor$ | last time index (**rename to** $H_{\max}$ **if confusion with graph** $H$; see note below) |
| $\mathcal T=\{0,\dots,H\}$ | time indices |
| $\bar a_i,\bar b_i,\bar s_i$ | window and service time in steps ($\lceil a_i/\delta\rceil$, $\lfloor b_i/\delta\rfloor$, $\lceil s_i/\delta\rceil$) |
| $\theta_{ij}(t)=\bar s_i+\lceil\tau_{ij}((t+\bar s_i)\delta)/\delta\rceil$ | steps between start of service at $i$ (step $t$) and arrival at $j$ |
| $\mathcal A$ | set of valid time-expanded arcs $(i,j,t)$ |
| $x_{ij}^t\in\{0,1\}$ | a truck starts service at $i$ at step $t$ then goes to $j$ |
| $y_i^t\in\{0,1\}$ | a truck waits at $i$ from step $t$ to $t+1$ |
| $f_{ij}^t\ge0$ | load on board on arc $x_{ij}^t$ |
| $g_i^t\ge0$ | load on board while waiting at $i$ during $[t,t+1]$ |

> **Note on collisions:** the symbol $H$ is used both for the customer graph and the last time index. Decision for the team: **use $H=(N,E)$ for the graph and $L=\lfloor T/\delta\rfloor$ for the last time index** in the final version (the current notebook still uses $H$ for both; fix in the next revision). Likewise $s$ is the slot index *and* $s_i$ the service time; slots are always written as a superscript ($v_a^{s}$), service times always carry a customer index.

## 7. Complexity vocabulary
| Symbol | Meaning |
|---|---|
| $\mathrm{P},\mathrm{NP}$ | classical complexity classes |
| $A\le_p B$ | polynomial-time reduction from $A$ to $B$ |
| HC | Hamiltonian Cycle |
| TSP, CVRP, VRPTW | travelling salesman, capacitated VRP, VRP with time windows |
| TD-CVRP | basic version (no time windows, no service times) |
| TD-CVRP-TW | full problem with time windows and service times |
| TD-CVRP-TW-D | decision version (is there a solution of duration $\le B$?) |
| $B$ | duration bound of the decision version |

## 8. Instance sizes (for the statistical study later)
| Symbol | Meaning |
|---|---|
| $n$ | number of customers (up to several thousand) |
| $\lvert V\rvert,\lvert A\rvert$ | size of the road network |
| $k,Q$ | fleet size and capacity |
| $\rho=\sum_i q_i/(kQ)$ | fleet load ratio (tightness of capacity), proposed instance parameter |
