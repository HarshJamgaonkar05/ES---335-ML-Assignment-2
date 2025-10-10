# **Analysis of Gradient Descent, SGD, and Momentum**

## 1. **Dataset and the Method used**

### 1.1 **Data Generation**
Two synthetic datasets were created to test the algorithms under different conditions:

* **Dataset 1 (Ill-Conditioned):**
    * $y = 100x + 1 + \epsilon$ where $x \sim U[-20, 20]$.
    * The large scale of the feature `x` makes the problem numerically challenging.
* **Dataset 2 (Well-Conditioned):**
    * $y = 3x + 4 + \epsilon$ where $x \sim U[-1, 1]$.
    * The feature `x` has a small scale, leading to a well-behaved loss surface.
* **Noise:** For both datasets, $\epsilon \sim N(0, 1)$.
* **Design Matrix:** The feature vector is augmented with an intercept term: $X = [1, x]$.

### **1.2 The Challenge: Geometry of the Ill-Conditioned Problem**
Dataset 1 is difficult to optimize for a key reason:
* The MSE loss surface $J(b, w)$ is a **highly elongated quadratic** (a "thin valley").
* This is because the Hessian matrix's eigenvalues are far apart ($\lambda_{\max} \gg \lambda_{\min}$), resulting in a **large condition number** $\kappa$.
* Consequently, the curvature is **very high** along the slope ($w$) direction and much flatter along the intercept ($b$) direction. A learning rate that is safe for the steep direction is too small to make meaningful progress in the flat direction.



## **2. Implemented Algorithms**

### **2.1 Vanilla Gradient Descent (GD & SGD)**
* **Full-batch GD:** Uses the entire dataset for each update.
    * Update rule: $\theta \leftarrow \theta - \eta \nabla J(\theta)$, where $\nabla J(\theta) = \frac{2}{n} X^T (X\theta - y)$.
* **Stochastic GD (SGD):** Uses a single, randomly chosen sample for each update.
    * Update rule: $\theta \leftarrow \theta - \eta \nabla_{\text{1-sample}} J(\theta)$, with per-sample gradient $2(\langle x_i, \theta \rangle - y_i)x_i$.

### **2.2 Gradient Descent with Momentum**
Classical momentum was implemented to help accelerate convergence, especially in ravines.
* **Update Rule:**
    $$
    \begin{align*}
    \mathbf{v}_t &\leftarrow \mu \mathbf{v}_{t-1} + \eta \nabla J(\boldsymbol{\theta}_t) \\
    \boldsymbol{\theta}_{t+1} &\leftarrow \boldsymbol{\theta}_t - \mathbf{v}_t
    \end{align*}
    $$
    Where $\mu$ is the momentum coefficient and $\mathbf{v}_t$ is the velocity.



## **3. Results on Ill-Conditioned Data (Dataset 1)**

### **3.1 Vanilla GD & SGD: Divergence **
* **Observation:** With a learning rate of $\eta=10^{-2}$, both GD and SGD **failed to converge** and hit the step cap.
* **Reason:** The learning rate was too large for the high curvature of the loss surface. The parameters consistently overshot the minimum, causing the loss to explode. The problem's high condition number ($\kappa \approx 1.03 \times 10^6$) requires an extremely small learning rate for vanilla GD to be stable.

### **3.2 Full-Batch GD with Momentum: Stable Convergence** 
* **Observation:** GD with momentum **converged successfully** in approximately **$166 \pm 6$ steps**.
* **Why it works:**
    * Momentum **damps the high-curvature bouncing** (oscillations in the steep `w` direction) that causes vanilla GD to diverge.
    * Simultaneously, it **accumulates speed** along the flat direction (the `b` direction), acting like a spectral preconditioner.
    * The deterministic (noise-free) gradients allow the velocity term to stabilize and align with the valley floor, leading to fast convergence despite visible overshoots.

### **3.3 SGD with Momentum: Velocity Explosion**
* **Observation:** With the same hyperparameters, SGD with momentum **did not converge**. The parameters exploded to astronomical scales ($10^{28}$) almost immediately.
* **Why it blows up:**
    * The per-sample gradients in SGD have **huge magnitude and variance** due to the lack of a $1/n$ factor and the large scale of `x`.
    * Momentum **integrates and amplifies this noise**. Sign flips from different samples don't cancel but instead accumulate in the velocity term.
    * This "wrong-way" velocity quickly **catapults the parameters out of the stable region**.
* **Takeaway:** On a badly conditioned problem with high gradient noise, momentum **amplifies** that noise, causing divergence unless the learning rate is drastically reduced.

### **4.1 DataSet 2 Observations**
* The inputs are small ($x \in [-1, 1]$), leading to a loss surface that is an **almost circular quadratic bowl**.
* The Hessian eigenvalues are close, resulting in a **low condition number**.
* A single learning rate works well for both parameters. Here, momentum's primary role is **acceleration**, not stabilization.


### **4.2 Full-Batch GD with Momentum: Pure Acceleration**
* **Numbers:** Converged in **$154.8 \pm 10.0$ steps**, an **~8x speedup** compared to vanilla GD (~1289 steps).
* **Why it works:** On a well-conditioned quadratic, momentum reduces the effective spectral radius of the iteration. The deterministic gradients allow the velocity to accumulate signal, leading to clean, linear convergence with a larger effective step.



### **4.3 Stochastic GD with Momentum: When Momentum Hurts**
* **Numbers:** Took **$68,426 \pm 37,910$ steps** on average. This is significantly **worse** than vanilla SGD (~15k steps).
* **Why it happens:**
    * In this well-conditioned bowl, the main bottleneck is **gradient variance**, not geometry.
    * Momentum integrates the noise from per-sample gradients. The velocity can carry stale directions for several steps, leading to temporary overshoots (spikes in the loss) before the stochastic averaging corrects the course.
    * Here, **noise dominates**, and momentum **amplifies** it, yielding slower and less stable progress than vanilla SGD on average.


### **Summary and Practical Takeaways**

* **Full-Batch + Momentum:** A **powerful combination**. On ill-conditioned problems, it provides stability and acceleration. On well-conditioned problems, it provides pure acceleration.
* **SGD + Momentum:** A **fragile combination**. On ill-conditioned problems, it amplifies noise and explodes. On well-conditioned problems, it can hurt performance by integrating noise and slowing convergence compared to vanilla SGD.
