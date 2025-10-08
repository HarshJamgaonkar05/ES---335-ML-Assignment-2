# Observations for Full Batch Graident Descent without feature scaling

### **1 Dataset and Method Used**
* **Data:**
  
  $$
  y = 3x + 2 + \epsilon, \qquad x \in [0, 1000]
  $$

* **Optimizer:** full-batch GD, learning rate \(1\times10^{-6}\), max `300k` iterations.
* **Convergence test:**
  
  $$
  \lVert \boldsymbol{\theta} - \boldsymbol{\theta}^{*} \rVert_{2} < 10^{-3}
  \quad\text{(where }\boldsymbol{\theta}^{*}\text{ is the LS solution)}
  $$

---

### **2 Ground truth (closed-form)**
* 
  $$
  \boldsymbol{\theta}^{*} = [\,b^{*},\, w^{*}\,] \approx [\,2.2139,\, 2.9995\,].
  $$

---

### **3 End of run (after 300k iterations)**
* **Converged to criterion?** No (stopped at max iterations).
* **Final** \(\boldsymbol{\theta} = [1.6533,\, 3.0004]\).
* **Distance to** \(\boldsymbol{\theta}^{*}\): \(0.5606\) (> \(10^{-3}\)).
* **Final MSE:** \(0.8961\) (close to noise level).

---

### **4 Parameter-wise behavior**
* **Slope \(w\)**: essentially correct early on (3.0004 vs 2.9995).
* **Intercept \(b\)**: noticeably low (1.65 vs 2.21) and improved very slowly.
* **Interpretation:** GD updates along the “slope direction” were fast, while updates along the “bias direction” were extremely slow.

---

### **5 Loss curve shape (MSE vs Iterations curve)**
* There’s a **very large initial loss spike**:
  
  $$
  \sim 3.4 \times 10^{6}
  $$
  
  that **drops quickly** within a few steps.
* After the initial drop, the curve **looks flat** on a linear scale—progress still occurs, but it’s too small to see without a log-scale plot.

