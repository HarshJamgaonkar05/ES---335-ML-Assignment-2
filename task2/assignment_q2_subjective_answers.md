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


  


