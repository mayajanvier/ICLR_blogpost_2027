---
layout: distill
title: Learning dynamics under physical constraints
# TODO Emma : écrire un résumé de 2-3 phrases (sans maths ni liens)
description: When it comes to enforcing constraints on trajectories, methods that learn the dynamics of a physical system and generative methods have more in common than you could think. This blog highlights their similarities in terms of algorithms and study how physical constraints affect the trajectories they generate, on a 2D ODE whose trajectories spiral onto the unit circle.
date: 2027-04-30
future: true
htmlwidgets: true
hidden: false

# Anonymize when submitting
authors:
  - name: Anonymous

# must be the exact same name as your blogpost
bibliography:  2027-04-30-physics-constrained.bib 

toc:
  - name: Enforcing physical constraints, a task shared between two communities
    subsections:
      - name: Two communities learning to simulate dynamics…
      - name: ... enforcing physical constraints under a common framework
  - name: "Our playground: the spiral landing system and its constraints"
    subsections:
      - name: An ODE that spirals onto the circle
      - name: The two constraints
  - name: Simulating the spiral landing system
    subsections:
      - name: Without constraints
      - name: Guidance strategies during trajectory generation
  - name: Conclusion

_styles: |
  /* TODO markers (remove before submission) */
  .todo { color:#c2410c; background:rgba(234,88,12,.12); border:1px solid rgba(234,88,12,.45); border-radius:4px; padding:0 .35em; font-weight:600; font-size:.9em; }
  html[data-theme="dark"] .todo { color:#fb923c; background:rgba(251,146,60,.14); }
  /* Pink highlight box */
  .pinkbox { background:#fdf2f8; border:1px solid #f9a8d4; border-left:5px solid #ec4899; border-radius:6px; padding:.9em 1.2em; margin:1.2em 0; }
  .pinkbox ul { margin:.4em 0 0; }
  html[data-theme="dark"] .pinkbox { background:rgba(236,72,153,.12); border-color:rgba(236,72,153,.5); border-left-color:#ec4899; }
  .pk-grid { display:grid; grid-template-columns:1fr 1fr; gap:1.2rem; margin-top:.6rem; }
  @media (max-width:760px) { .pk-grid { grid-template-columns:1fr; } }
  .pk-grid > div + div { border-left:1px dashed #f9a8d4; padding-left:1.2rem; }
  @media (max-width:760px) { .pk-grid > div + div { border-left:none; padding-left:0; } }
  .pk-grid .pk-title { margin:0 0 .3rem; }
  .pk-grid ul { margin:0; padding-left:1.1rem; font-size:.92em; }

  /* algorithm box */
  .algo { margin:1.4rem 0; padding:.4rem 1.2rem; border-left:4px solid #2b7bba; background:rgba(43,123,186,.07); border-radius:0 10px 10px 0; }
  html[data-theme="dark"] .algo { background:rgba(90,162,220,.13); }
  .algo .algo-note { margin:.6rem 0 .5rem; padding-top:.55rem; border-top:1px dashed rgba(43,123,186,.35); font-size:.88em; opacity:.85; }

  /* family colours: green = dynamics-learning, purple = generative */
  d-article { --dyn:#2e9e6a; --dyn-bg:#eaf6ef; --gen:#8e5cc4; --gen-bg:#f3edfa; }
  html[data-theme="dark"] d-article { --dyn:#4fc08d; --dyn-bg:rgba(79,192,141,.12); --gen:#b48be0; --gen-bg:rgba(180,139,224,.13); }
  .fam { margin:1rem 0; padding:.6rem 1.1rem; border-left:4px solid; border-radius:0 10px 10px 0; }
  .fam p:last-child { margin-bottom:0; }
  .fam-dyn { border-color:var(--dyn); background:var(--dyn-bg); }
  .fam-gen { border-color:var(--gen); background:var(--gen-bg); }
  .fam-dyn strong:first-child { color:var(--dyn); }
  .fam-gen strong:first-child { color:var(--gen); }
  .fam-grid { display:grid; grid-template-columns:1fr 1fr; gap:1rem; margin:1rem 0; }
  @media (max-width:760px) { .fam-grid { grid-template-columns:1fr; } }
  .fam-grid .fam { margin:0; font-size:.9em; }
  .fam-grid .fam-title { margin:.2rem 0 .5rem; font-weight:700; font-size:1.05em; }
  .fam-dyn .fam-title { color:var(--dyn); }
  .fam-gen .fam-title { color:var(--gen); }
  .fam-grid .fam-defs { margin:.4rem 0; padding-left:1.1rem; }
  .fam-grid .fam-algo { margin:.6rem 0 .2rem; padding:.5rem .5rem .5rem 1.8rem; border-top:1px dashed currentColor; }
  .fam-dyn .fam-algo { border-color:var(--dyn); }
  .fam-gen .fam-algo { border-color:var(--gen); }
  d-article tr:has(.dyn) td { background:var(--dyn-bg); }
  d-article tr:has(.gen) td { background:var(--gen-bg); }
  d-article tr:has(.dyn) td:first-child { border-left:4px solid var(--dyn); }
  d-article tr:has(.gen) td:first-child { border-left:4px solid var(--gen); }
  /* inline method names in family colour */
  .c-dyn { color:var(--dyn); font-weight:600; }
  .c-gen { color:var(--gen); font-weight:600; }
  /* constraint colours: blue = along the trajectory, gold = landing angles */
  d-article { --traj:#2563eb; --traj-bg:rgba(37,99,235,.06); --land:#a16207; --land-bg:rgba(202,138,4,.08); }
  html[data-theme="dark"] d-article { --traj:#60a5fa; --traj-bg:rgba(96,165,250,.10); --land:#facc15; --land-bg:rgba(250,204,21,.09); }
  .c-traj { color:var(--traj); font-weight:600; }
  .c-land { color:var(--land); font-weight:600; }
  d-article details.dyn-toggle.traj { --tg:var(--traj); --tg-bg:var(--traj-bg); --tg-line:rgba(37,99,235,.28); }
  d-article details.dyn-toggle.land { --tg:var(--land); --tg-bg:var(--land-bg); --tg-line:rgba(202,138,4,.32); }

  /* collapsible callouts: .dyn-toggle (blue, default) and .dyn-toggle.warn (amber) */
  d-article { --tg:#2b7bba; --tg-bg:rgba(43,123,186,.06); --tg-line:rgba(43,123,186,.28); }
  d-article details.dyn-toggle.warn { --tg:#c2410c; --tg-bg:rgba(234,88,12,.06); --tg-line:rgba(234,88,12,.3); }
  html[data-theme="dark"] d-article { --tg:#5aa2dc; --tg-bg:rgba(90,162,220,.10); --tg-line:rgba(90,162,220,.3); }
  html[data-theme="dark"] d-article details.dyn-toggle.warn { --tg:#fb923c; --tg-bg:rgba(251,146,60,.10); --tg-line:rgba(251,146,60,.3); }

  d-article details.dyn-toggle {
    margin:1.6rem 0; background:var(--tg-bg);
    border:1px solid var(--tg-line); border-left:4px solid var(--tg);
    border-radius:0 10px 10px 0; overflow:hidden;
  }
  d-article details.dyn-toggle > summary {
    display:flex; align-items:center; gap:.8rem;
    padding:.75rem 1rem; cursor:pointer; list-style:none;
    font-size:.95rem; line-height:1.4; color:inherit;
  }
  d-article details.dyn-toggle > summary::-webkit-details-marker { display:none; }
  d-article details.dyn-toggle > summary:hover { background:var(--tg-line); }
  d-article details.dyn-toggle > summary strong { font-weight:500; }
  d-article details.dyn-toggle .dyn-badge {
    flex:0 0 auto; white-space:nowrap;
    padding:.2rem .6rem; border:1.5px solid var(--tg); border-radius:999px;
    color:var(--tg); background:transparent;
    font-size:.68rem; font-weight:700; letter-spacing:.06em; text-transform:uppercase;
  }
  d-article details.dyn-toggle .dyn-arrow {
    flex:0 0 auto; margin-left:auto; color:var(--tg);
    font-size:1rem; transition:transform .2s ease;
  }
  d-article details.dyn-toggle[open] .dyn-arrow { transform:rotate(45deg); }
  d-article details.dyn-toggle .dyn-content {
    padding:.4rem 1.1rem 1rem; border-top:1px solid var(--tg-line);
  }
  d-article details.dyn-toggle .dyn-content .fam ul { margin:0; padding-left:1.1rem; }

  d-article .table-responsive { overflow-x:auto; width:100%; }
---

## Enforcing physical constraints, a task shared between two communities
In the world of physics modelisation, data-driven approaches represent cheap alternatives to classical numerical modelling, ranging from **Neural Ordinary Differential Equations (NODE)**<d-cite key="chen2026neuralordinarydifferentialequations"></d-cite>, that learn the dynamics from trajectory points, to **generative methods**, that simulate trajectories based on a learned distribution of states.

As many physical systems present continuous constraints called **algebraic constraints** ,i.e., that is respected continuously along any trajectory (for example conservation laws) or **final constraints** that should be respected at the end of the simulation (for control purposes), many methods have been developped to enforce these constraints on the learnt dynamics <d-cite key="white2024stabilizedneuraldifferentialequations"></d-cite> <d-cite key="pal2025semiexplicitneuraldaeslearning"></d-cite> or distributions <d-cite key="chamon2025constrainedsamplingprimalduallangevin"></d-cite> <d-cite key="blanke2026strictlyconstrainedgenerativemodeling"></d-cite> <d-cite key="chung2024diffusionposteriorsamplinggeneral"></d-cite>, in the hope of stabilizing or improving the model's simulations .

Although dynamics-learning and generative methods to enforce physical constraints appear different at first, they do not fundamentally differ and can be understood under a common **prediction-correction** framework that we will present here. 

<div class="pinkbox">
  <strong>The aim of the blog is:</strong>
  <ul>
    <li>to gather dynamics-learning and generative approaches</li>
    <li>study the efficiency on selected representant of each, on a system presenting both algebraic and final constraints.</li>
  </ul>
</div>

### Two communities learning to simulate dynamics...

**Setting.** We consider the dynamical equation of a physical system:

$$\frac{dx}{dt} = f(x), \qquad x(t) \in \mathbb{R}^d,$$

here an Ordinary Differential Equation (ODE). The trajectories are generated by using a numerical solver $S$ (such as Euler forward, Runge-Kutta schemes) with a time step $\Delta t$, starting from the initial condition $x_0 = x(0)$:

$$x(t + \Delta t) = S\big(x(t), f, \Delta t\big).$$

**Constraints.** We suppose our system presents an **algebraic constraint**, i.e., a constraint that is respected continuously along any trajectory:

$$g(x) = 0, \qquad g : \mathbb{R}^d \to \mathbb{R}^m,$$

and we call $$\mathcal{C} = \{x \in \mathbb{R}^d : g(x) = 0\}$$ the constraint set or manifold of constraints .


**Two families of learned models.**

<div class="fam fam-dyn" markdown="1">

**Dynamics-learning methods** learn the vector field, $f_\theta \approx f$, and integrate it in time with the same solver, such as NODEs <d-cite key="chen2026neuralordinarydifferentialequations"></d-cite>: $\hat x(t + \Delta t) = S\big(\hat x(t), f_\theta, \Delta t\big)$.
</div>

<div class="fam fam-gen" markdown="1">

**Generative methods** learn to transport a simple prior $p_0$ (usually $\mathcal{N}(0, I)$) to the distribution of states conditioned on the past states. 
$f_\theta$ is then a velocity field (flow matching) or a score $\nabla \log p_{\theta}$ (diffusion). To solve the dynamics, the models has to learn a conditional distribution: either $p_\theta\big(x(t + \Delta t) \mid x(t)\big)$ and generate the trajectory autoregressively or $p_\theta\big(x(\Delta t), \dots, x(T) \mid x_0\big)$ and generate the whole trajectory at once.
</div>


### ... enforcing physical constraints under a common framework
Let's consider the task for learning how to simulate trajectories of an ODE under some algebraic constraint. By separating trajectory simulation into a prediction or **learned forward step** and a **correction/projection step**, constrained integration of the learned dynamics and constrained sampling from a generative model can both be understood under a common framework, that we describe in the algorithm below. 

<div class="algo" markdown="1">

**Algorithm: learned step + correction**

**Inputs:** A neural network $f_\theta$, a correction operator $\mathcal{P}_{k}$, step sizes $h_k$, and the initial distribution $p_0$. 

1. Sample $x^0 \sim p_0$.
2. For $k = 0, \dots, K-1$:
   - learned forward step: $\tilde x^{k+1} = x^k + h_k\, f_\theta(x^k, k)$ (here Euler, could be another numerical or learned solver)
   - correction: $x^{k+1} = \mathcal{P}_{k}\big(\tilde x^{k+1}\big)$
3. Return the whole sequence $x^0, \dots, x^K$ (dynamics) or only $x^K$ (generative).

<p class="algo-note"><em>Note:</em> the correction operator \(\mathcal{P}_k\) can be hand-designed (e.g. an exact projection onto \(\mathcal{C}\) or a gradient of the constraint) or learned by a neural network \(\mathcal{P}_{\psi,k}\) with parameters \(\psi\).</p>
</div>

To better understand how this algorithm operates for constrained dynamics-learning and constrained sampling, we detail the inputs for each family below, with one major difference: the simulated time that is either a <span class="c-dyn">physical time</span> or a <span class="c-gen"> sampling time </span>. 

<div class="fam-grid">

<div class="fam fam-dyn">
  <p class="fam-title">Constrained dynamics-learning </p>
  <p><strong>\(k\) indexes the physical time</strong>, meaning: \(x^k = x_{k \Delta t}\) with \(h_k = \Delta t\) is the solver time step. Every iterate is a physical state, so the constraint is enforced <em>along the trajectory</em>.</p>
  <ul class="fam-defs">
    <li>\(d\): state dimension</li>
    <li>\(f_\theta : \mathbb{R}^d \to \mathbb{R}^d,\; x \mapsto f_\theta(x)\)</li>
    <li>\(\mathcal{P}_{k} : \mathbb{R}^d \to \mathbb{R}^d\): correction operator of \(x\)</li>
    <li>\(p_0 = \mathcal{N}(0, I_d)\): initial conditions</li>
  </ul>
  <ol class="fam-algo">
    <li>Sample \(x^0 \sim p_0\).</li>
    <li>For \(k = 0, \dots, K-1\):<br>
      \(\tilde x^{k+1} = x^k + h_k\, f_\theta(x^k)\)<br>
      \(x^{k+1} = \mathcal{P}_{k}(\tilde x^{k+1})\)</li>
    <li>Return the trajectory \(x^0, \dots, x^K\).</li>
  </ol>
</div>

<div class="fam fam-gen">
  <p class="fam-title"> Constrained generative sampling</p>
  <p><strong>\(k\) indexes the sampling time</strong>, meaning: \(h_k\) is the sampling time step, from noise \(X^0\) to a sample trajectory \(X^K\). Intermediate states are not physical, so only the <em>final sample</em> must satisfy the constraint.</p>
  <ul class="fam-defs">
    <li>\(X = [x_1, \dots, x_N] \in \mathbb{R}^{d \times N}\): a whole trajectory</li>
    <li>\(f_\theta : \mathbb{R}^{d\times N} \to \mathbb{R}^{d\times N},\; X \mapsto f_\theta(X)\)</li>
    <li>\(\mathcal{P}^N_{k}(X) = [\mathcal{P}_{k}(x_1), \dots, \mathcal{P}_{k}(x_N)]\)</li>
    <li>\(p_0 = \mathcal{N}(0, I_{d\times N})\): noise</li>
  </ul> 
  <ol class="fam-algo">
    <li>Sample \(X^0 \sim p_0\).</li>
    <li>For \(k = 0, \dots, K-1\):<br>
      \(\tilde X^{k+1} = X^k + h_k\, f_\theta(X^k, k)\)<br>
      \(X^{k+1} = \mathcal{P}^N_{k}(\tilde X^{k+1})\)</li>
    <li>Return only \(X^K\).</li>
  </ol>
</div>


</div>

We now have a general view of how both families can enforce physical constraints when simulating trajectories. In particular, both rely on a correction operator $$\mathcal{P}_k$$ that it is applied many times to enforce the constraint. There are actually various ways to apply it, depending on a schedule indexed by $$k$$. By specifying this schedule of $P_{k}$, we retrieve existing methods in the litterature:  

  <div class="pk-grid">
    <div>
      <p class="pk-title c-dyn"><span class="c-dyn">Dynamics-learning</span></p>
      <ul>
        <li><strong>Projected NODE (PNODE)</strong> <d-cite key="pal2025semiexplicitneuraldaeslearning"></d-cite>: \(\mathcal{P}_k = \mathcal{P}_{\mathcal{C}} \, \forall k \leq K \), the exact projection onto \(\mathcal{C}\) (closed form or solved iteratively). </li>
        <li><strong>Stabilized NODE (SNODE)</strong> <d-cite key="white2024stabilizedneuraldifferentialequations"></d-cite>: the correction operator directly modifies \(f_\theta\) into \(f_\theta - \gamma G^{+}g\)where $G^+$ is the Moore-Penrose pseudoinverse of the Jacobian of $g$. When the solver is Euler, it translates into the schedule \(\mathcal{P}_k(\tilde x) = \tilde x - \gamma h_k G^{+}(\tilde x)g(\tilde x)\). As described in <d-cite key="pal2025semiexplicitneuraldaeslearning"></d-cite>, it corresponds to a single step of the full projection \(\mathcal{P}_\mathcal{C}\). </li>
      </ul>
    </div>
    <div>
      <p class="pk-title c-gen"><span class="c-gen"> Generative methods </span></p>
      <ul>
        <li><strong>Final state projection (TODO mettre refs):</strong> \(\mathcal{P}_k = \mathrm{Id}\) for \(k &lt; K\), and \(\mathcal{P}_K=\mathcal{P}_{\mathcal{C}}\), i.e., no correction on the intermediate noisy states, only on the final state.</li>
        <li> <strong> Primal Dual Constrained Sampling</strong> <d-cite key="chamon2025constrainedsamplingprimalduallangevin"></d-cite> : similarly to SNODE, the projection is guided via a gradient step towards \(\mathcal{C}\), i.e., \(\mathcal{P}_k(\tilde x) = \tilde x - \alpha_k \nabla g(\tilde x)\).</li>
      </ul>
    </div>
  </div>

<br>
More methods can be classified into this prediction-correction framework. If you are curious, we detail some state-of-the-art (SOTA) approches in the following table.

<details class="dyn-toggle">
  <summary>
    <span class="dyn-badge">Taxonomy</span>
    <strong> Identification of prediction and correction step of SOTA methods</strong>
    <span class="dyn-arrow">＋</span>
  </summary>

  <div class="dyn-content" markdown="1">

<span class="todo">TODO écrire les cellules DPS, daps. inclure precisement la correction. Vérifier ce que j'ai écrit pour constrained sampling, j'ai lu les papiers à moitié.</span>

We report this analysis for various state-of-the-art methods in the following table. This taxonomy is non exhaustive, but gives examples for both families.

<div class="table-responsive" markdown="1">

| Method | Loop index $$k$$ | $$f_\theta$$ learns | Correction $$\mathcal{P}_{k}$$ | Constraint enforced |
|---|---|---|---|---|
| <span class="dyn"></span>Neural ODE (baseline) | Physical time | Vector field $$f$$ | None | – |
| <span class="dyn"></span> SNODE <d-cite key="white2024stabilizedneuraldifferentialequations"></d-cite> | Physical time | Vector field $$f$$ | $$\mathcal{P}_k(x)=x-\gamma h_k G^{+}(x)g(x)$$ | $$\mathcal{C}$$ is asymptotically stable. |
| <span class="dyn"></span>PNODE <d-cite key="pal2025semiexplicitneuraldaeslearning"></d-cite> | Physical time | Vector field $$f$$ | $$\mathcal{P}_k= \mathcal{P}_{\mathcal{C}}$$, projection onto $$\mathcal{C}$$ using Newton's method with a fixed Jacobian.| Full projection onto $$\mathcal{C}$$ at each step. |
| <span class="gen"></span>GenCast <d-cite key="price2024gencastdiffusionbasedensembleforecasting"></d-cite>, Autoregressive conditional diffusion <d-cite key="kohl2024benchmarkingautoregressiveconditionaldiffusion"></d-cite> | Physical time + inner sampling loop | Conditional score of $$p(x_{t+\Delta_t}\mid x_t)$$ | None | – |
| <span class="gen"></span>Constrained Sampling (primal-dual) <d-cite key="chamon2025constrainedsamplingprimalduallangevin"></d-cite> | Sampling time | Full trajectory distribution | Langevin sampling step with penalty $$\lambda^\top g$$, solved using a primal-dual formulation. | Generated distribution satisfies the constraint in expectation, $$\mathbb{E}[g(x)]=0$$. |
| <span class="gen"></span>Constrained Sampling (split) <d-cite key="blanke2026strictlyconstrainedgenerativemodeling"></d-cite> | Sampling time | Score | Variable splitting: alternates a sampling step and a projection onto $$\mathcal{C}$$. | Final sample (exact constraint?) |
| <span class="gen"></span>DPS <d-cite key="chung2024diffusionposteriorsamplinggeneral"></d-cite> | | | | |
| <span class="gen"></span>DAPS <d-cite key="Zhang_2025"></d-cite> | | | | |

</div>

  </div>
</details>


 In the rest of this blog, we investigate a representant of each family: <span class="c-dyn">SNODE</span> and <span class="c-gen">Primal Dual Constrained Sampling</span>. We will test both on a 2D ODE whose trajectories spiral onto the unit circle. Let's discover our playground in the next section...

## Our playground: the spiral landing system and its constraints

We choose a simple 2D system with both type of constraints in order to easily visualize and understand the impact of the methods on the predicted trajectories.

### An ODE that spirals onto the circle

The equation: for $z = (z_1, z_2) \in \mathbb{R}^2$ (here the state is $x = z$, with $d = 2$):

$$
\dot z = \Bigl(1 - \frac{1}{\lVert z \rVert}\Bigr) M z,
\qquad M = \begin{pmatrix} -1 & -\alpha \\ \alpha & -1 \end{pmatrix},
$$

where $\alpha \in \mathbb{R}$ controls the rotation ($\alpha = 2$ throughout).

In polar coordinates $z = r(\cos\theta, \sin\theta)$, the system decouples:

$$
\begin{cases}
\dfrac{dr}{dt} = -(r - 1) \\[6pt]
\dfrac{d\theta}{dt} = \alpha\,\dfrac{r - 1}{r}
\end{cases}
$$

The radius relaxes to 1 independently of $\theta$, and the rotation stops as the point lands. The closed-form solution is

$$
r(t) = 1 + (r_0 - 1)\,e^{-t},
\qquad
\theta(t) = \theta_0 + \alpha \log r_0 - \alpha \log\bigl(1 + (r_0 - 1)e^{-t}\bigr).
$$


<span class="todo">Reduce the size</span>
{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/rockets_gt.gif" class="img-fluid rounded" caption="This system ressembles rockets landing onto the Earth, right ?" %}
### The two constraints


- <span class="c-traj">Constraint along the trajectory (algebraic).</span> We have $\frac{d\theta}{dr}=\frac{-\alpha}{r}$, meaning $I(\theta, r)=\theta-\alpha \ln(r)$ is constant along a trajectory. In carthesian coordinates, it writes $I(x,y)=\arctan(y/x) + \alpha \ln(\sqrt{x^2+y^2})$ and we have the algebraic constraint $g(x,y)=I(x,y)-I(x_0,y_0)=0$.

- <span class="c-land">Final constraint: landing angles (attractor).</span> Every trajectory lands on the unit circle: $g(z) = \lVert z \rVert^2 - 1 = 0$ as $t \to \infty$. A start at $(r_0, \theta_0)$ lands at angle $\theta_\infty = \theta_0 + \alpha \log r_0$.

With three classes of initial angle $\theta_0 \in \{0.3\pi, 0.8\pi, 1.5\pi\}$ and $r_0 \sim \mathrm{LogNormal}(0, \sigma^2)$, $\sigma = 0.15$, the landing angles are exactly Gaussian, $\theta_\infty \sim \mathcal N(\theta_0, (\alpha\sigma)^2)$:

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/landing_angles_gt.png" class="img-fluid rounded" caption="Final states: landing points on the unit circle (left) and their angle distribution per class (right)." %}

## Simulating the spiral landing system

It's finally time to test dynamics-learning and generative approaches in the simulation of the spiral landing system ! We first inspect baseline approaches without any constraints before testing their constrained counterparts. 

### Without constraints
--> EMMA
**Same start, three trajectories.** All models start from the same Gaussian initial points $z_0 \sim \mathcal N(0, I)$.
We compare a Neural ODE that learns the $f_\theta$ and a flow matching model that learns the transport between starting point and final state distirbution.
We show the trajectories generated by these models:
We first simulate our system without any constraint, using NODE for the dynamics-learning methods and flow matching for the generative methods. 

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/traj_gt_node_fmtraj.gif" class="img-fluid rounded" caption="Ground truth, neural ODE and flow matching trajectory simulation from the same initial points." %}

We can now visualize the simulation time distinction between the two methods: the ground truth (GT) and neural ODE trajectories advance the 2D state in the physical time (left and middle panels), while the trajectory flow matching (FM) iteratively refine a trajectory in the sample time (right panel).

Qualitatively and quantitatively, the NODE trajectories are accurately reproducing the true ones  with a Mean Squared Error (MSE) on the test set of $1.31e-2$, while the flow matching ones are a bit noisier with a MSE on the test set of $4.92e-2$. 

**But do these methods also respect the different constraints we defined ?** 

<details class="dyn-toggle traj">
  <summary>
    <span class="dyn-badge">Along the trajectory</span>
    <strong> Are the models satisfying the algebraic constraint?</strong>
    <span class="dyn-arrow">＋</span>
  </summary>
  <div class="dyn-content" markdown="1">

We can check if these methods conserve $I(x,y)$ along the trajectories. We plot $g(x_t,y_t)$ along the 100 test trajectories and for our three models, a value of 0 meaning that the constraint is respected.

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/constraint_gt_node_fmtraj.png" class="img-fluid rounded" caption="Evolution of the algebraic constraint along trajectories over time." %}

As expected, all GT trajectories respect the algebraic constraint for all timesteps. We observe that NODE is quite good at respecting the algebraic constraint as well, with $g(x_t,y_t)$ overall centered around 0 and with only some trajectories drifting away. The flow matching method, although overall centered around 0 too, is more noisy and less constrained with more amplitude in the error. 

_NB: Here, the constraint is computed on the final sampled trajectories for the flow matching strategy, so the $x$ axis is indeed the physical time as for the other panels._

  </div>
</details>

<details class="dyn-toggle land">
  <summary>
    <span class="dyn-badge">Landing angles</span>
    <strong> Is the distribution of landing angles respected ?</strong>
    <span class="dyn-arrow">＋</span>
  </summary>
  <div class="dyn-content" markdown="1">

We can also check if the final constraint is respected. Starting from three classes of initial angle $\theta_0 \in \{0.3\pi, 0.8\pi, 1.5\pi\}$ with $r_0 \sim \mathrm{LogNormal}(0, 0.15^2)$, the ground truth spreads each class into $\mathcal N(\theta_0, (\alpha\sigma)^2)$ on the circle. We compare the landing-angle distributions of the three models (1-Wasserstein distance to the ground truth).

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/landing_angles_gt_node_fmtraj.png" class="img-fluid rounded" caption="Angle distributions at the start (dashed lines: $\theta_0$) and at landing, per angle class. On the learned models' panels, the ground-truth landing histogram is outlined." %}

We observe that when no constraint is enforced during training, the dynamical learning does not respect the landing constraint: NODE produces a distribution centered at the true mean but over-representing some angles. On the other hand, the generative approach almost perfectly fit the ground truth landing angles distribution. 

It can be explained as the generative method is designed to transport distributions, while NODE is only tasked to learn a dynamic, with the smallest error getting amplified along a trajectory as you leave the attractor seen during training <d-cite key="list2024differentiabilityunrolledtrainingneural"></d-cite>. (je sais pas si c;est la best reference ni la bonne explication, a voir, ou alors overfitting ?)

  </div>
</details>

**Conclusion** We see that both approaches are complementary. On one hand, NODE is better as preserving the constraint along trajectories as it learns the true dynamic but does not preserve the initial Gaussian distribution into the landing angles and can over-represent certain landing angles (overfitting) ? On the other hand, the flow matching model succeeds in preserving the Gaussian distribution of final states, but does not preserve the algebraic constraint as tightly as NODE. 

These observations further motivate the usage of methods to enforce constraints, that we investigate in the next section. 

### Guidance strategies during trajectory generation
<span class="todo">-> MAYA et BlANDINE
GT, NODE, SNODE
FM traj, FM traj + constraint guidance</span>

In this section, we want to enforce $g(x_t,y_t)=0$ for every timestep of the predicted trajectory. Since projecting can become very costly in realistic settings, we rather focus on methods that "guide" the trajectory towards the constraint manifold $\mathcal{C}$: <span class="c-dyn">SNODE</span> and <span class="c-gen">Primal Dual Constrained Sampling</span>.

peut etre le mettre en toggle ici en mode rappel
<div class="fam fam-dyn" markdown="1">

<strong><span class="c-dyn">Reminder </span></strong> The idea of <span class="c-dyn">SNODE</span> is to directly modify the Neural ODE with a term that “nudges” back the trajectory to the manifold of algebraic constraint $$\mathcal{C}=\{x, g(x)=0\}$$: 
$$
\dot{\hat x} = f_\theta(\hat x) - \gamma S(\hat x )g(\hat x).
$$

The manifold $\mathcal{C}$ is asymptotically Lyapunov-stable in the SNODE under certain constraints for the stabilization matrix $S$, which they choose to be the Moore-Penrose pseudoinverse of the Jacobian of $g$ at $\hat x$, $G^+=J_g^T(J_g J_g^T)^{-1}$, with $J_g$ the Jacobian matrix of $g$. 

</div>

<details>
<summary><strong>A little training challenge</strong></summary>

The constraint has discontinuities due to the domain of definition of `arctan`, so we have to use `torch.remainder` to ensure it is zero modulo \(2\pi\): this is what we use during training. 
<br>
<br>
For testing, a more rigorous metric can be defined: we compute an unwrapped angle that is continuous and avoids the jumps due to `arctan`.
<br>
<br>
We train several models with \(\gamma \in \{0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 2.0\}\) and choose the model with the best final training loss: \(\gamma_{\mathrm{opt}}=0.4\).

</details>


<div class="fam fam-gen" markdown="1">
<strong><span class="c-gen">Reminder </span></strong> The idea of <span class="c-gen">Primal Dual</span>
</div>

<details>
<summary><strong>When should we apply the projection during generative sampling?</strong></summary>

- Closed form: when we have a closed form, it is more straightforward to apply it at the end.
- Gradient guidance: <span class="todo">TODO</span>

</details>

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/traj_gt_node_snode.gif" class="img-fluid rounded" caption="Ground truth, neural ODE and Stabilized NODE trajectories from the same initial points." %}

On test trajectories, NODE and SNODE perform similarly, with respective MSE of 1.63e-2 and 1.82e-2. 

**Evaluation.**

<details class="dyn-toggle traj">
  <summary>
    <span class="dyn-badge">Along the trajectory</span>
    <strong>Are the models satisfying the algebraic constraint?</strong>
    <span class="dyn-arrow">＋</span>
  </summary>
  <div class="dyn-content" markdown="1">

#### SNODE

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/constraint_gt_node_snode.png" class="img-fluid rounded" caption="Algebraic constraint along the trajectories for the ground truth, neural ODE and SNODE." %}

Looking at the algebraic constraint, SNODE seems to sometimes perturbate the initial trajectories, inducing some errors in the respect of the constraint for some. The $\gamma$ parameter is particularly sensitive to calibrate, as it requires 

$$\gamma \geq \frac{\Vert J_g(x)f_\theta(x)\Vert}{\lambda_0\Vert g(x)\Vert}$$
with $\lambda_0$ the smallest eigenvalue of $J_g$ the Jacobian of the constraint $g$.

Furthermore, SNODE is designed to be asymptotically stable for the manifold of constraint, meaning it is designed to prevent any divergences in the long term trajectories. Here the trajectories can be too short to observe the stabilization effect. This observation will be validated in section 5.  
<span class="todo">TODO: FM + constraint blandine</span>
  </div>
</details>

<details class="dyn-toggle land">
  <summary>
    <span class="dyn-badge">Landing angles</span>
    <strong>Is the distribution of landing angles respected ?</strong>
    <span class="dyn-arrow">＋</span>
  </summary>
  <div class="dyn-content" markdown="1">

#### SNODE
Although SNODE is a method to enforce the constraint along the whole trajectory, we can analyse its impact on the landing angle distribution. 

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/landing_angles_gt_node_snode.png" class="img-fluid rounded" caption="Angle distributions at the start (dashed lines: $\theta_0$) and at landing, per class. On the learned models' panels, the ground-truth landing histogram is outlined." %}

We observe that the model with the constraint is now very good at transporting the initial gaussian distribution of angles: even though SNODE can perturbate the trajectories, the manifold of the constraint remain asymptotically stable, that is to say the constraint is progressively pushed to $\mathcal{C}$. This is why the model is good when we look at the final angle, but can be less good during the trajectory (that are quite short here).


<span class="todo">TODO: FM + constraint blandine</span>
  </div>
</details>




## Conclusion

- PNODE: doing the full projection, although more expensive, could ensure the respect of the algebraic constraint more strictly than SNODE


We saw that these methods can be explained under the same framework but they are not originally designed for the same applications. 

Bridging those approaches requires to play with the methods by predicting windows instead of states for  generative modeling. 


In the end, integrating dynamics and sampling processes boils down to integrating equations. It's therefore not surprising that similar constraining methods have been developed in their respective litterature.

Our blog provides a framework to make a parallel between them.

Differences remain, depending on whether you need to generate distributions of trajectories, or need flexibility of the timestep of the trajectories. One may be better suited than the other.

Next: PDE-Refiner and Thermalizer ?
