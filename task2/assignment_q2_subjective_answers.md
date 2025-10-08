# Observations for full-batch gradient descent without feature scaling 

### **1 Data and Method Used**
* **Data:** Targets are generated from a straight line with slope three and intercept two, plus small random noise. The input feature x ranges from 0 to 1000 (so it’s a large-scale feature).
* **Optimizer:** Full-batch gradient descent with a learning rate of 1e-6, capped at 300,000 iterations.
* **Convergence test:** We declare success when the Euclidean distance between the current parameters and the least-squares solution is less than one-thousandth.


### **2 Ground truth (closed-form)**
* **Least-squares solution (b*, w*):** Intercept about 2.2139 and slope about 2.9995.


### **3 End of run (after 300k iterations)**
* **Converged to criterion?** No (stopped at the maximum number of iterations).
* **Final parameters:** Intercept 1.6533, slope 3.0004.
* **Distance to the true solution:** About 0.5606, which is greater than the required one-thousandth.
* **Final mean-squared error:** About 0.8961 (roughly the noise level).


### **4 Parameter-wise behavior**
* **Slope:** Essentially correct very early (3.0004 vs 2.9995).
* **Intercept:** Too low at first (1.65 vs 2.21) and improved very slowly.
* **Interpretation:** Updates along the “slope direction” were fast; updates along the “bias/intercept direction” were extremely slow.


### **5 Loss curve shape (MSE vs iterations)**
* The loss starts with a very large spike (on the order of a few million) and then drops quickly within the first few steps.
* After that initial drop, the curve appears almost flat on a linear scale. Progress still happens, but it’s so small that it’s easier to see on a logarithmic scale plot.

### **6 Diagnosis**

* **Ill-conditioning due to scale:**  
  Since the input feature values are in the thousands, the data matrix becomes highly unbalanced. The column corresponding to x dominates the one for the bias term. This creates a large difference between the biggest and smallest eigenvalues, meaning the system is poorly conditioned.

* **Effect on learning rate:**  
  To prevent divergence, the learning rate must be smaller than the reciprocal of the largest eigenvalue. Because that eigenvalue is huge, the required learning rate becomes extremely small — resulting in very tiny step sizes.

* **Impact on parameter updates:**  
  With such a small learning rate, gradient descent quickly corrects the slope (since it corresponds to the direction with stronger curvature) but updates the intercept extremely slowly (since that direction has weak curvature).  
  In simple terms, the slope learns fast, but the intercept learns at a glacial pace.


## Why we used the Lipschitz-based step size 

- **Problem seen earlier:** With a hand-picked tiny learning rate, full-batch gradient descent still did **not** meet the convergence criterion (distance to the least-squares solution below one-thousandth), even after many iterations. The slope converged quickly, but the intercept crawled — a classic symptom of ill-conditioning because the feature x spans 0 to 1000.

- **Risk if we increase LR blindly:** We could diverge. Gradient descent must respect the curvature in the stiffest direction, so an overly large step will explode there.

- **What we did:** We computed a safe, “near-optimal” learning rate from the smoothness of the MSE. Concretely: estimate the Lipschitz constant **L** from the largest eigenvalue of \(X^T X\) (scaled by 2/n), then set the learning rate **η** to **0.9 divided by L**.  
  This guarantees stability and takes the largest step that is still safe for the worst curvature.

- **Goal:** Remove guesswork about the learning rate and make GD **as fast as it can be without exploding**, while keeping everything else unchanged (no feature scaling).


## What changed (method)

- Calculated **L** from \(X^T X\) and set **η = 0.9 / L**.
- Kept the same dataset (no feature scaling), full-batch GD, maximum 20k iterations, stop if the parameter distance to the least-squares solution drops below one-thousandth.

## Observations for this case

### 1) Numbers
- **L** ≈ 617,371.2 → chosen **η** ≈ 1.458 × 10⁻⁶.  
- **Converged?** No (within 20,000 iterations).
- **Final parameters:** intercept ≈ 1.5579, slope ≈ 3.0005; **distance to the true solution** ≈ 0.6559.
- **Final MSE:** ≈ 0.929.

### 2) Parameter behavior
- **Slope** is essentially correct very early (about 3.0005 vs true ≈ 2.9995).
- **Intercept** remains low (about 1.56 vs ≈ 2.21) and improves **very slowly**.

### 3) Loss curve
- **Big spike at the start** (on the order of 3.3 million), sharp drop, then it **appears flat** on a linear scale.  
  A **log-MSE** plot would reveal slow, steady decay.

### 4) Interpretation
- **η = 0.9 / L** is stability-optimal for the stiffest direction, so training is safe and as fast as possible **in that direction**.
- But the problem remains **ill-conditioned**: directions with small curvature (notably the bias/intercept) still update **microscopically**, so the parameter distance stays large even though the MSE is already near the noise floor.

  


