# Successive Convexification (SCvx) and Guaranteed Sequential Trajectory Optimization (GuSTO) SCP Algorithms

## SCOPE
This note is all about two specific **Sequential Convex Programming (SCP)** algorithms; **Successive Convexification (SCvx)** and **Guaranteed Sequential Trajectory Optimization (GuSTO)**. We will briefly introduce SCP before diving into the two algorithms discussing what 
they are used for and how to use them. This is a collection of notes taken from a paper written by researchers from the University of 
Washington's Autonomous Controls Laboratory (UW ACL), the developers of the two algorithms[1].

## Motivation and Introduction
Generating safe and reliable trajectories is crucial in the autonomous world through the computation of multidimensional discrete state and control signals within a set of constraints that satisfies a set of specifications while optimizing for a mission objective. This challenges engineers and researchers to develop algorithms that focus on the safety, performance, and trustworthiness of trajectory generation. Numerical Optimization presents a great solution for the generation and optimization of trajectories to meet objectives so it can be expressed as an Optimal Control Problem.

The optimization problem is solved through the following:
> 1. **Formulation:** specifications of how the functions to be minimized and constraints to be satisfied are expressed mathematically.
> 2. **Discretization:** approximation of the infinite-dimensional state and control signals by a finite-dimensional set of basis functions.
> 3. **Numerical Optimization:** iterative computation of an optimal solution of the discretized problem.

In terms of efficiency and reliability for numerical optimization, discretized convex problems are the way to go, which is the main motivation for **Convex Programming** or **Convex Optimization**. However, you may find that most problems in the real world are **nonlinear (nonconvex)**, which leads to the topic of **Convexification** and **SCP** for trajectory generation of nonlinear dynamic systems based on convex optimization. The main idea of SCP is iterative convex approximation as the name suggests. The general outline of an SCP algorithm is shown in **Fig. 1** where it starts with an initial trajectory guess that goes through an iteration scheme that optimizes the trajectory until it converges into a feasible solution at which the iteration is stopped. With SCP, safe and reliable trajectories can be generated for nonconvex functions.

![SCP Block Diagram](figs/SCP_diagram.png)
**Figure 1:** Block diagram of a typical SCP algorithm acquired from [1].

An algorithm hierarchy in **Fig. 2** shows where SCP lies. Typically, the convex solver in layer 2 (denoted by the circled numbers) of **Fig. 1** iteratively calls an **Interior Point Method (IPM)** algorithm from layer 3 in **Fig. 2** that solves a convex problem with linear equality and convex inequality constraints as a sequence of linear equality-constrained problems and thus, IPM iteratively calls the algorithm in layer 2 in **Fig. 2**.

![SCP Block Diagram](figs/SCP_hierarchy.png)
**Figure 2:** Algorithm hierarchy also acquired from [1].

## Preliminaries
**Formulation**
The objective of SCP is to solve continuous-time-optimal control problems in the following form:

$$
\begin{align}
\min_{u, p} J(x, u, p), \tag{1a} \\
s.t. \quad \dot{x}(t) = f(t, x(t), u(t), p), \tag{1b} \\
(x(t), p) \in \mathcal{X}(t), \tag{1c} \\
(u(t), p) \in \mathcal{U}(t), \tag{1d} \\
s(t, x(t), u(t), p) \leq 0, \tag{1e} \\
g_{ic}(x(0), p) = 0, \tag{1f} \\
g_{tc}(x(1), p) = 0, \tag{1g}
\end{align}
$$

where $x(\cdot) \in \mathbb{R}^{n}$ is the state trajectory, $u(\cdot) \in \mathbb{R}^{m}$ is the control trajectory, and $p \in  \mathbb{R}^{d}$ is a vector of parameters. The function $f:\mathbb{R} \times \mathbb{R}^{n} \times \mathbb{R}^{m} \times \mathbb{R}^{d} \rightarrow \mathbb{R}^{n}$ represents the nonlinear dynamics assumed to be at least once continuously differentiable. Continuously differentiable functions $g_{ic}: \mathbb{R}^{n} \times \mathbb{R}^{d} \rightarrow \mathbb{R}^{n_{ic}}$ and $g_{tc}: \mathbb{R}^{n} \times \mathbb{R}^{d} \rightarrow \mathbb{R}^{n_{tc}}$ enforces the initial and terminal boundary conditions. The sets $\mathcal{X}(t)$ and $\mathcal{U}(t)$ represent the convex path constraints of state and control. The continuously differentiable function $s:\mathbb{R} \times \mathbb{R}^{n} \times \mathbb{R}^{m} \times \mathbb{R}^{d} \rightarrow \mathbb{R}^{n_{s}}$ represents the nonconvex path constraints. $\mathcal{X}(t)$ and $\mathcal{U}(t)$ are assumed to be compact, that is, closed and bounded so that the system cannot escape to infinity or apply impossible control inputs. Equation (1) is defined on the [0,1] time interval and the constraints must hold at each time instant where the initial and final time parameters in $p$, $t_{0}$ and $t_{f}$, can be substituted.

The cost function (1a) is assumed to be in the form:

$$
\begin{align}
J(x, u, p) = \phi(x(1), p) + \int_{0}^{1} \Gamma(x(t), u(t), p)dt \tag{2}
\end{align}
$$

where the terminal cost $\phi: \mathbb{R}^{n} \times \mathbb{R}^{d} \rightarrow \mathbb{R}$ is assumed to be a convex function and the running cost $\Gamma: \mathbb{R}^{n} \times \mathbb{R}^{m} \times \mathbb{R}^{d} \rightarrow \mathbb{R}$ can be, in general, a nonconvex function.

SCP methods work by solving local convex approximations to (1), referred to as subproblems, which require an existing reference trajectory or reference solution. SCP updates this reference solution through each loop iteration of **Fig. 1**. The initial trajectory is the initial trajectory guess which can be infeasible with respect to the dynamics (1b) and the nonconvex constraints (1e)-(1g) but needs to be feasible with with respect to the convex constraints (1c) and (1d). A good rule of thumb for initialization is the straight-line interpolation method which works for a variety of problems and it is in the form:

$$
\begin{align}
\bar{x}(t) = (1-t) x_{ic} + t x_{tc}, \quad for \quad t \in [0,1] \tag{3}
\end{align}
$$

As for the initial input trajectory, inputs are chosen based on insight from the physics of the problem whenever possible and if not possible, set inputs to the smallest feasible value that satisfies (1d). The input trajectory is in a similar form to (3) which is:

$$
\begin{align}
\bar{u}(t) = (1-t) u_{ic} + t u_{tc}, \quad for \quad t \in [0,1] \tag{4}
\end{align}
$$

The initial guess for $\bar{p}$ can significantly impact the number of SCP iterations to obtain a solution. There is no ideal rule of thumb for initial guesses for $\bar{p}$, however, the run time of SCP is usually in the order of a few seconds or shorter so the user can experiment with different values and develop a good initialization strategy. SCvx and GuSTO will always converge on a solution but do not guarantee a feasible solution that satisfies (1) so it is important to provide a good guess to reduce time, increase solution optimality, and encourage converging to a feasible solution.

**Approximation / Linearization**
Most of the convexification of (1) is done through the first-order approximation at the reference trajectory by computing the following jacobian matrices:

$$
\begin{align}
A(t) \triangleq \nabla_{x}f(t, \bar{x}(t), \bar{u}(t), \bar{p}), \tag{5a} \\
B(t) \triangleq \nabla_{u}f(t, \bar{x}(t), \bar{u}(t), \bar{p}), \tag{5b} \\
F(t) \triangleq \nabla_{p}f(t, \bar{x}(t), \bar{u}(t), \bar{p}), \tag{5c} \\
r(t) = f(t, \bar{x}(t), \bar{u}(t), \bar{p}) - A\bar{x}(t) -B\bar{u}(t) - F\bar{p} \tag{5d} \\
C(t) \triangleq \nabla_{x}s(t, \bar{x}(t), \bar{u}(t), \bar{p}), \tag{5e} \\
D(t) \triangleq \nabla_{u}s(t, \bar{x}(t), \bar{u}(t), \bar{p}), \tag{5f} \\
G(t) \triangleq \nabla_{p}s(t, \bar{x}(t), \bar{u}(t), \bar{p}), \tag{5g} \\
r'(t) = s(t, \bar{x}(t), \bar{u}(t), \bar{p}) - C\bar{x}(t) -D\bar{u}(t) - G\bar{p} \tag{5h} \\
H_{0} \triangleq \nabla_{x}g_{ic}(\bar{x}(0), \bar{p}), \tag{5i} \\
K_{0} \triangleq \nabla_{p}g_{ic}(\bar{x}(0), \bar{p}), \tag{5j} \\
l_{0} \triangleq g_{ic}(\bar{x}(0), \bar{p}) - H_{0}\bar{x}(0) - K_{0}\bar{p}, \tag{5k} \\
H_{f} \triangleq \nabla_{x}g_{tc}(\bar{x}(1), \bar{p}), \tag{5l} \\
K_{f} \triangleq \nabla_{p}g_{tc}(\bar{x}(1), \bar{p}), \tag{5m} \\
l_{f} \triangleq g_{tc}(\bar{x}(1), \bar{p}) - H_{f}\bar{x}(1) - K_{f}\bar{p}. \tag{5n}
\end{align}
$$

and the linearized (1) becomes:

$$
\begin{align}
\min_{u,p} J(x, u, p), \tag{6a} \\
s.t. \quad \dot{x}(t) = A(t)x(t) + B(t)u(t) + F(t)p(t) + r(t), \tag{6b} \\
(x(t), p) \in \mathcal{X}(t), \tag{6c} \\
(u(t), p) \in \mathcal{U}(t), \tag{6d} \\
C(t)x(t) + D(t)u(t) + G(t)p + r'(t) \leq 0, \tag{6e} \\
H_{0}x(0) + K_{0}p + l_{0} = 0, \tag{6f} \\
H_{f}x(1) + K_{f}p + l_{f} = 0. \tag{6g} \\
\end{align}
$$

Notice that the cost is not convexified while everything else is convex in (6). The cost function will be convexified through SCvx and GuSTO algorithm to be discussed. The control trajectory $u(\cdot)$ is an infinite-dimensional vector space of continuous-time functions and needs to be discretized through temporal discretization and direct colocation so it can be solved numerically.

Linearization guarantees that constraints in (6) are convex but it introduces **Artificial Unboundedness** where the subproblem solution must be kept close to the linearization point and sometimes the convex cost can go to negative infinity.

Artificial Unboundedness can be solved by adding the following trust region constraint:

$$
\begin{align}
\delta x(t) = x(t) - \bar{x}(t), \\
\delta u(t) = u(t) - \bar{u}(t), \\
\delta p = p - \bar{p}, \\
\alpha_{x}\parallel \delta x(t) \parallel_{q} + \alpha_{u}\parallel \delta u(t) \parallel \leq \eta, \quad for \quad t \in [0,1], \tag{7}
\end{align}
$$

for some choice of $q \in {1,2,2^{+}, \inf}$ and constants $\alpha_{x}, \alpha_{u}, \alpha_{p} \in {0,1}$. Use $q = 2^{+}$ to denote the ability to impose the trust region as the quadratic two-norm squared. The trust region radius $\eta$ is a fixed scalar that is updated between SCP iterations.

Linearization also introduces **Artificial Infeasibility** where the linearized constraints produce no solution or a feasible solution is outside the trust region so the SCP cannot find a solution because the trust region is too small. There are two approaches to remedy this; Add an unconstrained **virtual controller** slack variable that is penalized or penalize constraint violations by augmenting the cost function (6a) with **soft penalty** terms.

This is the reason why SCP can produce an infeasible solution which is referred to as a "soft" failure because usually, a few virtual control terms are nonzero. A feasible solution can be converged through the tuning of the parameters and this leads to the creation of different SCP algorithms such as the SCvx and GuSTO algorithms.

## Main Body
**SCvx Algorithm**
The SCvx algorithm follows three things:
> 1. The terminal and running cost is assumed to be convex by offloading nonconvex terms into the constraints.
> 2. The constraint (7) is enforced as a hard constraint and $\eta$ is adjusted through the SCvx update rule.
> 3. Artificial infeasibility is handled using virtual control terms.

The virtual control terms to handle artificial infeasibility are added to (6b) and (6e-6g) as follows:

$$
\begin{align}
\dot{x} = A x + B u + F p + r + Ev, \tag{8a} \\
v_{s} \geq C x + D u + G p + r', \tag{8b} \\
0 = H_{0}x(0) + K_{0}p + l_{0} + v_{ic}, \tag{8c} \\
0 = H_{f}x(1) + K_{f}p + l_{f} + v_{tc}, \tag{8d}
\end{align}
$$

where $v(\cdot) \in R^{n_{v}}$, $v_{s}(\cdot) \in R^{n_{s}}$, $v_{ic} \in R^{n_{ic}}$, and $v_{tc} \in R^{n_{tc}}$ are the virtual control terms. The pair (A, E) is required to be controllable and a common choice for $E$ is $E = I_{n}$.

The virtual control terms are paired with a penalty term $P:R^{n} \times R^{p} \rightarrow R_{+}$ where p is any appropriate integer so that they are only used when it is necessary to avoid subproblem infeasibility. A typical choice is:

$$
\begin{align}
P(y,z) = \parallel y \parallel_{1} + \parallel z \parallel_{1}, (9)
\end{align}
$$

where y and z are placeholder functions. The cost function (6a) is then augmented with penalty terms as:

$$
\begin{align}
J_{\lambda}(x,u,p,\hat{v}) \triangleq \phi_{\lambda}(x(1), p, v_{ic}, v_{tc}) + \int_{0}^{1} \Gamma_{\lambda}(x,u,p,Ev,v_{s})dt, \tag{10} \\
\phi_{\lambda}(x(1),p,v_{ic},v_{tc}) = \phi(x(1),p) + \lambda P(0,v_{ic}) + \lambda P(0,v_{tc}), \\
\Gamma_{\lambda}(x,u,p,Ev,v_{s}) = \Gamma(x,u,p) + \lambda P(Ev,v_{s}). \tag{11}.
\end{align}
$$

where $\hat{v}$ is short for $v,v_{s},v_{ic},v_{tc}$. The positive weight $\lambda \in R_{++}$ is selected by the user and must be sufficiently large (typically a power of 10).

So now the convex subproblem can be stated as:

$$
\begin{align}
\min_{u,p,\hat{v}} J_{\lambda}(x,u,p,\hat{v}), \tag{12a} \\
s.t. \quad \dot{x} = Ax + Bu + Fp + r + Ev, \tag{12b} \\
(x, p) \in \mathcal{X}, \quad (u, p) \in \mathcal{U}, \tag{12c} \\
Cx + Du + Gp + r' \leq v_{s}, \tag{12d} \\
H_{0}x(0) + K_{0}p + l_{0} + v_{ic} = 0, \tag{12e} \\
H_{f}x(1) + K_{f}p + l_{f} + v_{tc} = 0, \tag{12f} \\
\parallel \delta x \parallel_{q} + \parallel \delta u \parallel_q + \parallel \delta p \parallel_{q} \leq \eta. \tag{12g}
\end{align}
$$

We now apply temporal discretization to solve the problem numerically with temporal nodes $t_{k} \in [0,1] \quad for \quad k=1,...,N,$ and cast the subproblem with variables $x = \[x_{k}\]^{N}\_{k=1}$, $u = \[u{k}\]^{N}\_{k=1}$, $p$, $v = \[v_{k}\]^{N-1}\_{k=1}$, $v_{s} = \[v_{s,k}\]^{N}\_{k=1}$, $v_{ic}$, and $v_{tc}$. With zero order hold (ZOH) for the controls, we get the discretized problems:

$$
\begin{align}
\mathcal{L}_{\lambda}(x,u,p,\hat{v}) = \phi\_{\lambda}(x(1),p,v\_{ic},v\_{tc}) + trapz(\Gamma^{N}\_{\lambda}) \tag{13} \\
\Gamma^{N}\_{\lambda,k} = \Gamma\_{\lambda}(x_\{k},u_{k},p,E_{k}v_{k},v_{s,k}), \tag{14}
\end{align}
$$



**GuSTO Algorithm**
The GuSTO algorithm follows two things:
> 1. Artificial unboundedness is handled through the augmentation of the cost function with a soft penalty on the violation of (7) and $\alpha_{u} = 0$ since (1) is convex in the control variables.
> 2. Artificial infeasibility is handled through the augmentation of the cost function with a soft penalty on nonconvex path constraint violation and as the algorithm progresses, the penalty weight increases.

## Conclusion
You will notice that the Main Body only contains the key factors of how the SCvx and GuSTO works without much more detail and that is because I ran out of time to complete the note. The revised note shall include more detail information such as the addition and modification of constraints, the update rule for each algorithm, and the convergence guarantee for each.

## References
[1]: Malyuta, D., Reynolds, T. P., Szmuk, M., Lew, T., Bonalli, R., Pavone, M., and Açıkmeşe, B., “Convex Optimization for Trajectory
Generation: A Tutorial on Generating Dynamically Feasible Trajectories Reliably and Efficiently,” IEEE Control Systems,
Vol. 42, No. 5, 2022, pp. 40–113. doi:10.1109/mcs.2022.3187542, URL https://doi.org/10.1109/mcs.2022.3187542,
free preprint available at https://arxiv.org/abs/2106.09125
