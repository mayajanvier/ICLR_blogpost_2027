---
layout: distill
title: Learning dynamics under physical constraints
# TODO Emma : écrire un résumé de 2-3 phrases (sans maths ni liens)
description: We compare methods that learn the dynamics of a physical system with generative methods, and study how physical constraints affect the trajectories they generate, on a 2D ODE whose trajectories spiral onto the unit circle.
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
  - name: Introduction
    subsections:
      - name: A common framework
      - name: Taxonomy of the methods
  - name: System and constraints
    subsections:
      - name: An ODE that spirals onto the circle
      - name: The two constraints
  - name: Simulating the spiral landing system
    subsections:
      - name: NODE vs flow matching (no constraints)
      - name: "Landing on your feet: enforcing the final constraint"
      - name: "Following the plan: generating the right trajectories"

_styles: |
  /* TODO markers (remove before submission) */
  .todo { color:#c2410c; background:rgba(234,88,12,.12); border:1px solid rgba(234,88,12,.45); border-radius:4px; padding:0 .35em; font-weight:600; font-size:.9em; }
  html[data-theme="dark"] .todo { color:#fb923c; background:rgba(251,146,60,.14); }
  /* Pink highlight box */
  .pinkbox { background:#fdf2f8; border:1px solid #f9a8d4; border-left:5px solid #ec4899; border-radius:6px; padding:.9em 1.2em; margin:1.2em 0; }
  .pinkbox ul { margin:.4em 0 0; }
  html[data-theme="dark"] .pinkbox { background:rgba(236,72,153,.12); border-color:rgba(236,72,153,.5); border-left-color:#ec4899; }

  /* algorithm box */
  .algo { margin:1.4rem 0; padding:.4rem 1.2rem; border-left:4px solid #2b7bba; background:rgba(43,123,186,.07); border-radius:0 10px 10px 0; }
  html[data-theme="dark"] .algo { background:rgba(90,162,220,.13); }

  /* family colours: green = dynamics-learning, purple = generative */
  d-article { --dyn:#2e9e6a; --dyn-bg:#eaf6ef; --gen:#8e5cc4; --gen-bg:#f3edfa; }
  html[data-theme="dark"] d-article { --dyn:#4fc08d; --dyn-bg:rgba(79,192,141,.12); --gen:#b48be0; --gen-bg:rgba(180,139,224,.13); }
  .fam { margin:1rem 0; padding:.6rem 1.1rem; border-left:4px solid; border-radius:0 10px 10px 0; }
  .fam p:last-child { margin-bottom:0; }
  .fam-dyn { border-color:var(--dyn); background:var(--dyn-bg); }
  .fam-gen { border-color:var(--gen); background:var(--gen-bg); }
  .fam-dyn strong:first-child { color:var(--dyn); }
  .fam-gen strong:first-child { color:var(--gen); }
  d-article tr:has(.dyn) td { background:var(--dyn-bg); }
  d-article tr:has(.gen) td { background:var(--gen-bg); }
  d-article tr:has(.dyn) td:first-child { border-left:4px solid var(--dyn); }
  d-article tr:has(.gen) td:first-child { border-left:4px solid var(--gen); }
  /* inline method names in family colour */
  .c-dyn { color:var(--dyn); font-weight:600; }
  .c-gen { color:var(--gen); font-weight:600; }
---

<span class="todo">DISCLAIMER: toutes les métriques temporelles sont calculées sur $t_{gif}$ qu'il va falloir unifier entre SNODE/FM pour Blandine et pour pouvoir dire des choses sur nos méthodes</span>

## 1. Enforcing physical constraints, a task shared between two communities
In the world of physics modelisation, data-driven approaches represent cheap alternatives to classical numerical modelling, ranging from **Neural Ordinary Differential Equations (NODE)**, that learn the dynamics from trajectory points, to **generative methods**, that simulate trajectories based on a learned distribution of states.<d-cite key="white2024stabilizedneuraldifferentialequations"></d-cite> (TODO: mettre citations je sais pas comment) As many physical systems present continuous constraints called **algebraic constraints** ,i.e., that is respected continuously along any trajectory (for example conservation laws) or **final constraints** that should be respected at the end of the simulation (for control purposes), many methods have been developped to enforce these constraints on the learnt dynamics or distributions, in the hope of stabilizing or improving the model's simulations.

Although dynamics-learning and generative methods to enforce physical constraints appear different at first, they do not fundamentally differ and can be understood under a common **prediction-projection** framework that we will present here. 

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

**Dynamics-learning methods** learn the vector field, $f_\theta \approx f$, and integrate it in time with the same solver (such as NODEs): $\hat x(t + \Delta t) = S\big(\hat x(t), f_\theta, \Delta t\big)$.
</div>

<div class="fam fam-gen" markdown="1">

**Generative methods** learn to transport a simple prior $p_0$ (usually $\mathcal{N}(0, I)$) to the distribution of states conditioned on the past states. 
$f_\theta$ is then a velocity field (flow matching) or a score $\nabla \log p_{\theta}$ (diffusion). To solve the dynamics, the models has to learn a conditional distribution: either $p_\theta\big(x(t + \Delta t) \mid x(t)\big)$ and generate the trajectory autoregressively or $p_\theta\big(x(\Delta t), \dots, x(T) \mid x_0\big)$ and generate the whole trajectory at once.
</div>


### ... enforcing physical constraints under a common framework
By separating trajectory simulation into a prediction or **learned forward step** and a **correction/projection step**, constrained integration of the learned dynamics and constrained sampling from a generative model can both be understood under a common framework, that we describe in the algorithm below. 

<div class="algo" markdown="1">

**Algorithm: learned step + correction**

**Input:** network $f_\theta$, correction operator $\mathcal{P}_{\psi,k}$, step sizes $h_k$, initial distribution $p_0$.

1. Sample $x^0 \sim p_0$.
2. For $k = 0, \dots, K-1$:
   - learned forward step: $\tilde x^{k+1} = x^k + h_k\, f_\theta(x^k, k)$ (here Euler, could be another numerical or learned solver)
   - correction: $x^{k+1} = \mathcal{P}_{\psi,k}\big(\tilde x^{k+1}\big)$
3. Return the whole sequence $x^0, \dots, x^K$ (dynamics) or only $x^K$ (generative).

</div>
We track the residual $\lVert g(x^k) \rVert$ to measure how well the constraint is respected.

**Be careful ! The index $k$ doesn't have the same meaning for both families :** 

- **Dynamics-learning:** $k$ is the physical time, $x^k = x_k$ and $h_k = \Delta t$. Every integration step generates a physical state, so correcting every iterate enforces the constraint **along the trajectory**.
- **Generative:** in generative models $k$ represents the sampling time that goes from noise  $x^0 \sim \mathcal{N}(0, I)$ to a sample $x^K$. The intermediate states of the integration are noisy and are not physical states so imposing the constraints of those states has no physical meaning. Only the final sample needs to satisfy the constraint. For this reason, some of the constrained samplers estimates the final state to impose the constraint on it, $\hat x(x^k) = \mathbb{E}[x^K \mid x^k]$ using Tweedie's formula.
- **Autoregressive generative models** For autoregressive generative models there is an outer loop in physical time $n$ and, for each step, an inner sampling loop in $k$ that generates $x_{n+1}$ conditioned on $x_n$.

**Taxonomy of the methods** 

We can then explain various methods from both families under this framework by identifying the learned step and correction step of each. We perform this analysis for various state-of-the-art methods in the following table.

<span class="todo">TODO ecrire les cellules DPS, daisy, daps, et thermalizer et refiner ? (mais y a plus de contrainte physique ca va embrouiller je pense)
Verifier ce que j'ai ecris pour constrained sampling j'ai lu les papiers a moittie</span>

| Method | Loop index $$k$$ | $$f_\theta$$ learns | Correction $$\mathcal{P}_{\psi,k}$$ | Constraint enforced on |
|---|---|---|---|---|
| <span class="dyn"></span>Neural ODE (baseline) | physical time | vector field $$f$$ | None | – |
| <span class="dyn"></span>Stabilized NODE (SNODE) <d-cite key="white2024stabilizedneuraldifferentialequations"></d-cite> | physical time | vector field $f$ | Stabilized Neural Differential Equation (SNDE): $f_{\theta}-\gamma\, G(x)^{+} g(x)$ where $G^{+}$ is the Moore-Penrose pseudoinverse of the Jacobian of $g$, nudges the state to $\mathcal{C}$. | Every $x_n$, $\mathcal{C}$ is asymptotically stable in the SNDE equation. |
| <span class="dyn"></span>Projected NODE (PNODE) <d-cite key="pal2025semiexplicitneuraldaeslearning"></d-cite> | physical time | vector field $f$ | Projection onto $\mathcal{C}$ using Newton's method with a fixed Jacobian. SNODE as a special case of PNODE with a single step. | Every $x_n$, full projection onto $\mathcal{C}$ at each step.  |
| <span class="gen"></span>GenCast <d-cite key="price2024gencastdiffusionbasedensembleforecasting"></d-cite>, Auto Regressive conditional diffusion <d-cite key="kohl2024benchmarkingautoregressiveconditionaldiffusion"></d-cite> | physical time + inner sampling loop | conditional score of $p(x_{t+\Delta_t} \mid x_t)$ | None |–|
| <span class="gen"></span>Constrained Sampling (primal-dual) <d-cite key="chamon2025constrainedsamplingprimalduallangevin"></d-cite> | sampling time |full trajectory distribution | Takes Langevin step (diffusion sampling) with a penalty $\lambda^\top g$ solved with primal/dual formulation. | Generated distribution follows the constraint in expectation $\mathbb{E}[g(x)] = 0$. |
| <span class="gen"></span>Constrained Sampling (split) <d-cite key="blanke2026strictlyconstrainedgenerativemodeling"></d-cite> | sampling time | score | Variable splitting: introduces a variable to alternates a sampling step and a projection onto $\mathcal{C}$. | Final sample (exact constraint ?) |
| <span class="gen"></span>DPS <d-cite key="chung2024diffusionposteriorsamplinggeneral"></d-cite> |  |  |  |  |
| <span class="gen"></span>DAPS <d-cite key="Zhang_2025"></d-cite> | |  | |  |
| <span class="gen"></span>DAISI <d-cite key="andrae2026daisidataassimilationinverse"></d-cite> |  |  |  | |

Among these methods, we further illustrate <span class="c-dyn">SNODE</span> and <span class="c-gen">Primal-Dual Constrained Sampling</span> as we will use them as representants of respectively the  <span class="c-dyn">dynamics-learning</span> and generative methods in the experiments of this blog.
<span class="todo">TODO faire dessins en plus du lien d'algo pour les méthodes qu'on utilise ensuite</span>.

We will test both on a 2D ODE whose trajectories spiral onto the unit circle. This system presents both kinds of constraint, and we can measure them exactly: a quantity **conserved along every trajectory**, and a distribution of **final states** on the unit circle.

## 2. Our playground: the spiral landing system and its constraints

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

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/rockets_gt.gif" class="img-fluid rounded" caption="Rockets landing on the circle along the true trajectories." %}

This system ressembles rockets landing onto the Earth, right ?

### The two constraints
<span class="todo">est ce que ca va pas embrouiller d'evoquer les 2 contraintes. on part sur la contrainte a chaque pas de temps? -> je pense que ca peut être interessant de garder les 2 si on separe bien</span>
- **Final constraint (attractor).** Every trajectory lands on the unit circle: $g(z) = \lVert z \rVert^2 - 1 = 0$ as $t \to \infty$. A start at $(r_0, \theta_0)$ lands at angle $\theta_\infty = \theta_0 + \alpha \log r_0$.
- **Constraint along the trajectory (algebraic).** We have $\frac{d\theta}{dr}=\frac{-\alpha}{r}$, meaning $I(\theta, r)=\theta-\alpha \ln(r)$ is constant along a trajectory. In carthesian coordinates, it writes $I(x,y)=\arctan(y/x) + \alpha \ln(\sqrt{x^2+y^2})$ and we have the algebraic constraint $g(x,y)=I(x,y)-I(x_0,y_0)=0$.

With three classes of initial angle $\theta_0 \in \{0.3\pi, 0.8\pi, 1.5\pi\}$ and $r_0 \sim \mathrm{LogNormal}(0, \sigma^2)$, $\sigma = 0.15$, the landing angles are exactly Gaussian, $\theta_\infty \sim \mathcal N(\theta_0, (\alpha\sigma)^2)$:

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/landing_angles_gt.png" class="img-fluid rounded" caption="Final states: landing points on the unit circle (left) and their angle distribution per class (right)." %}

## 3. Simulating the spiral landing system without constraints

<span class="todo">TODO general: peut être qu'on peut être un blog avec du code a faire tourner pour plus de tutoriel, mais je sais pas si on aura le temps</span>

We first simulate our system without any constraint, using NODE for the dynamics-learning methods and flow matching for the generative methods. 

**Same start, three trajectories.** All models start from the same Gaussian initial points $z_0 \sim \mathcal N(0, I)$.
We compare a Neural ODE that learns the $f_\theta$ and a flow matching model that learns the transport between starting point and final state distirbution.
We show the trajectories generated by these models:

<span class="todo">TODO je pense que le fait que flow matching montrent des trajectoires rectilignes c'est un peu attendu et je me demande toujours si ca fait sens de comparer les 2 trajectoires generes, mais bon ca peut etre par curiosite aussi. Peut etre ca vaut le coup ici de mettre le FM qui apprend aussi la trajectoire entiere ? -> je suis d'accord faut rajouter les trajectoires de Blandine ici aussi en mode GT-NODE-FM-FM(traj)</span>

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/traj_gt_node_fm.gif" class="img-fluid rounded" caption="Ground truth, neural ODE and flow matching from the same initial points." %}

Qualitatively and quantitatively, the NODE trajectories are accurately reproducing the true ones (MSE on the test set: $1.63e-2$). <span class="todo">FM-traj ??</span> But do they also respect the different constraints we defined ? 

### Algebraic constraint
We can check if these methods conserve $I(x,y)$ along the trajectories. 

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/constraint_gt_node_fm.png" class="img-fluid rounded" caption="Evolution of the algebraic constraint along trajectories over time." %}

We observe that NODE is quite good at respecting the algebraic constraint. For the flow matching tasked to generate only the final state, it is expected not to enforce it at each time step since we did not task it to, however we observe that even the last state is not efficiently constrained in that setting. <span class="todo">TODO: FM-traj blandine</span>

We will show how to enforce this constraint along trajectories in both dynamics-landing and generative setups, and compare them to these baselines in section 3: following the plan (TODO mettre lien section pour plus d'interactivité).

### Landing angles
We can also check if the final constraint is respected. Starting from three classes of initial angle $\theta_0 \in \{0.3\pi, 0.8\pi, 1.5\pi\}$ with $r_0 \sim \mathrm{LogNormal}(0, 0.15^2)$, the ground truth spreads each class into $\mathcal N(\theta_0, (\alpha\sigma)^2)$ on the circle. We compare the landing-angle distributions of the three models (1-Wasserstein distance to the ground truth).

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/landing_angles_models.png" class="img-fluid rounded" caption="Angle distributions at the start (dashed lines: $\theta_0$) and at landing, per class. On the learned models' panels, the ground-truth landing histogram is outlined." %}

We observe that when no constraint is enforced during training, both dynamical learning and generative methods do not respect the landing constraint: NODE produces a distribution centered at the true mean but over-representing some angles, while flow matching generates shifted gaussians from the mean. This further motivates the usage of methods to enforce final constraint, that we will investigate lastly in section 5: landing on your feet. (TODO: mettre un lien vers la section). 

## 4. Following the plan: enforcing the algebraic constraint along the trajectory

In this section, we will try to enforce $g(x_t,y_t)=0$ for every timestep of the predicted trajectory. 

### SNODE
The idea of SNODE is to directly modify the Neural ODE with a term that “nudges” back the trajectory to the manifold of algebraic constraint $\mathcal{M}=\{u, g(u)=0\}$: 
$$
\dot{\hat z} = f_\theta(\hat z) - \gamma S(\hat z )g(\hat z)
$$
They prove that the manifold $\mathcal{M}$ is asymptotically Lyapunov-stable in the SNODE under certain constraints for the stabilization matrix $S$, which they choose to be the Moore-Penrose pseudoinverse of the Jacobian of $g$ at $\hat z$, $G^+=J_g^T(J_g J_g^T)^{-1}$, with $J_g$ the Jacobian matrix of $g$. 


**A little training challenge** The constraint has discontinuities due to arctan, so we have to use torch.remainder to ensure it is zero modulo 2pi. (dvp more + figure): this is what we use during training. For testing, a more rigourous metric can be defined: we compute an unwrapped angle that is continuous and avoid the jumps due to the arctan. 

We train several models with $\gamma \in \{0.1,0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 2.0\}$ and choose the model with the best final training loss: $\gamma_{opt}=0.4$. 

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/traj_gt_node_snode.gif" class="img-fluid rounded" caption="Ground truth, neural ODE and Stabilized NODE trajectories from the same initial points." %}

On test trajectories, NODE and SNODE perform similarly, with respective MSE of 1.63e-2 and 1.82e-2. 

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/constraint_gt_node_snode.png" class="img-fluid rounded" caption="Algebraic constraint along the trajectories for the ground truth, neural ODE and SNODE." %}

Looking at the algebraic constraint, SNODE seems to sometimes perturbate the initial trajectories, inducing some errors in the respect of the constraint for some. The $\gamma$ parameter is particularly sensitive to calibrate, as it requires 

$$\gamma \geq \frac{\Vert J_g(x)f_\theta(x)\Vert}{\lambda_0\Vert g(x)\Vert}$$
with $\lambda_0$ the smallest eigenvalue of $J_g$ the Jacobian of the constraint $g$.

Furthermore, SNODE is designed to be asymptotically stable for the manifold of constraint, meaning it is designed to prevent any divergences in the long term trajectories. Here the trajectories can be too short to observe the stabilization effect. This observation will be validated in section 5.  

### Flow matching learning all trajectory + constraint: Blandine

## 5. Landing on your feet: enforcing the final constraint 

### SNODE
Although SNODE is a method to enforce the constraint along the whole trajectory, we can analyse its impact on the landing angle distribution. 

{% include figure.liquid path="assets/img/2027-04-30-physics-constrained/landing_angles_gt_node_snode.png" class="img-fluid rounded" caption="Angle distributions at the start (dashed lines: $\theta_0$) and at landing, per class. On the learned models' panels, the ground-truth landing histogram is outlined." %}

We observe that the model with the constraint is now very good at transporting the initial gaussian distribution of angles: even though SNODE can perturbate the trajectories, the manifold of the constraint remain asymptotically stable, that is to say the constraint is progressively pushed to $\mathcal{C}$. This is why the model is good when we look at the final angle, but can be less good during the trajectory (that are quite short here).

### Flow matching learning all trajectory + constraint: Blandine


<span class="todo">TODO Plot de g(^xt) a la  en fonction du temps de rollout pour les modeles.</span>

<span class="todo">TODO Regarder ce que fait la projection a chaque etape de rollout sur un gif ?</span>

## Conclusion

- PNODE: doing the full projection, although more expensive, could ensure the respect of the algebraic constraint more strictly than SNODE

