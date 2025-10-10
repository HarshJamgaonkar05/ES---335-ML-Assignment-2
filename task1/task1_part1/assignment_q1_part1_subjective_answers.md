##**Observations — Dataset 1 (Vanilla GD & SGD)**

**Ground truth**
- Closed-form minimizer: $\theta^* = [b^*, w^*] \approx [0.951, 99.984]$.


#### **1) Non-convergence & step cap**
- Both methods hit the safety cap (`50,000` steps) → the criterion $\|\theta_t - \theta^*\|_2 < 10^{-3}$ was **never** satisfied.
- Note the tolerance is **extremely tight** relative to the parameter scale (e.g., slope $\sim 100$); any instability prevents the parameters from reaching this neighborhood.



#### **2) Divergence signatures in the plots**
- Loss vs epoch curves **increase superlinearly** (explosion), not decay → updates consistently overshoot.
- Parameter trajectories on the $(b, w)$ contour plots **shoot away** from $\theta^*$ and quickly leave the plotted range (scientific-notation ticks are a tell).


#### **3) Conditioning & geometry**
- With $x \in [-20, 20]$ and a true slope $\sim 100$, the matrix $X^T X$ has a **very large largest eigenvalue** (due to the wide spread in $x$).
- → The MSE level sets are **highly elongated ellipses** (forming a narrow valley).
- In such valleys, a fixed step size causes a **zig-zag/overshoot** pattern along the stiff direction.
  - GD overshoots deterministically.
  - SGD adds **gradient noise**, which amplifies the instability even earlier.


#### **4) Learning-rate mismatch**
- For quadratic losses, a safe learning rate must obey $\eta < 1/L$, where $L = \frac{2}{n}\lambda_{\max}(X^T X)$ is the Lipschitz constant of the gradient, $\nabla J$.
- Using $\eta = 10^{-2}$ here is **far too large** for Dataset 1's curvature. The effective "stiffness" from the wide $x$-range makes $L$ large, which in turn requires a very small $1/L$.

#### **5) Full-batch vs SGD behavior**
- GD takes one update per epoch (using the full gradient). It diverges **smoothly** once the learning rate $\eta$ exceeds the stability bound.
- SGD takes one update **per sample**. Even if the mean step is acceptable, the **variance** in the stochastic gradient can push individual steps past the stability limit, leading to a **faster explosion** and a larger spread in early epochs.


#### **6) Intercept–slope coupling**
- Because the design matrix is $X = [1, x]$, the intercept $b$ and slope $w$ updates are **coupled** (the off-diagonal elements of $X^T X$ are nonzero).
- With a large variance, $\text{var}(x)$, this coupling is **strong**. This can cause updates to rotate the parameter vector along an unstable direction, accelerating divergence.


#### **7) Practical fixes (what would make it converge)**
- **Reduce the learning rate** using a curvature-aware bound, such as choosing $\eta \approx (0.1 \text{ to } 0.9) \times 1/L$.
- **Standardize** the feature $x$ (to have zero-mean and unit-variance). This improves the problem's conditioning and widens the stable learning rate window.
- Optionally use **momentum** with a smaller LR or a **line search** algorithm. Both methods are better at tracking the narrow valley without overshooting.
- If you continue to use SGD, use a **smaller** $\eta$ than you would for full-batch GD (to account for noise) and consider using **mini-batches**.





# **Observations for Dataset 1 with safe learning rates**

### **Numbers**
- **Lipschitz estimate:** $L \approx 2.527 \times 10^2$.
- **Safe stepsizes:** $\eta_{\text{GD}} \approx 3.561 \times 10^{-3}$ $(\approx 0.9/L)$, and $\eta_{\text{SGD}} \approx 7.123 \times 10^{-4}$ (smaller to account for noise).
- **Steps to $\epsilon$:** GD ≈ `1208` updates; SGD ≈ `3233` updates.



### **Convergence behavior**

**GD (full-batch)**
- The loss drops **5–6 orders of magnitude** within the first ~4–5 steps (visible on a log-scale plot), then steadily decays and flattens as it approaches the minimizer.
- The parameter path heads straight into the valley and settles close to $\theta^*$, with no divergence or overshoot.

**SGD (bs=1)**
- The loss shows a **noisy but downward** trend each epoch, which is typical for stochastic updates.
- It needs ~2.7× more steps than GD to hit the $\epsilon$-ball, reflecting the variance in the per-sample gradients.


### **Geometry from the contours**
- The focused contour view shows **elongated, nearly horizontal level sets** in the plotted window. This indicates that the curvature is **much steeper** in the $w$ direction than in the $b$ direction locally.
- This anisotropy is exactly why a large global LR was unstable before. With a learning rate $\eta \propto 1/L$ that respects the curvature, both methods now move stably **along the stiff direction** first and then fine-tune along the shallow one.

### **Why the safe LR works**
- For a quadratic MSE, stability requires $\eta < 1/L$, with the curvature constant $L = \frac{2}{n}\lambda_{\max}(X^T X)$.
- Dataset 1's wide span in $x \in [-20, 20]$ makes $\lambda_{\max}(X^T X)$ large, which means a **small** step size is needed for stable convergence.
- Using $\eta_{\text{GD}} \approx 0.9/L$ avoids overshooting, while using an even smaller $\eta_{\text{SGD}}$ helps to damp the inherent noise of the stochastic updates.



### **GD vs SGD (takeaways)**
- **Speed:** GD converges faster in terms of **steps** (`1208` vs `3233`).
- **Stability:** Both methods are stable with curvature-aware learning rates. SGD remains noisier but still consistently descends towards the minimum.
- **Compute:** One GD “step” processes the entire batch, whereas SGD counts a single sample update as a "step." Therefore, wall-clock time trade-offs will depend heavily on the specific implementation and hardware.
