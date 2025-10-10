
## **Objective 1: Image Reconstruction(Observations)**

### **Low-Rank Matrix Reconstruction**

In low-rank matrix reconstruction, the **rank ($r$)** controls the **expressive capacity** of the factorization:

$$X \approx W \cdot H$$

where $X$ is the image matrix and $(W, H)$ are its low-dimensional latent factors.

A **small rank** forces a simple model that captures only the **dominant global patterns**—broad color regions or smooth intensity variations. It therefore fails to reproduce **fine edges and textures**, particularly when large areas (e.g., a $30 \times 30$ block) are missing. The reconstructed result appears **over-smoothed or blurry**, since the model cannot represent local high-frequency structure.

Increasing the **rank ($r$)** enlarges the model’s capacity to encode **more complex correlations and spatial details**, improving recovery of edges and subtle textures. Yet, if the rank is **too high**, the model may:

* **Overfit** the known pixels instead of generalizing to the missing ones, and
* Incur **greater computational cost** and slower convergence.

Hence, selecting the rank is a **trade-off** between simplicity and accuracy:

* **Moderate ranks** suffice for *randomly missing* pixels, where the model benefits from contextual information across the image.
* **Higher ranks** are useful for *contiguous missing blocks*, where the algorithm must infer structure using only distant global cues.

Tuning the **number of iterations** and **learning rate** during gradient descent also affects the final quality—well-chosen parameters let the optimization converge smoothly toward a faithful, high-resolution reconstruction.

---

### **Random Missing Pixels vs. Missing Blocks**

A clear experimental observation is that **low-rank reconstruction performs better for random missing pixels** than for block-wise missing regions.

* With **random missing pixels**, known data are **scattered throughout** the image. Each local neighborhood retains some valid information, so the algorithm can **interpolate missing intensities using nearby pixels**, producing sharp and accurate restorations.

* With a **large missing block**, however, the region lacks **any local reference**. The model must rely solely on **global correlations** learned from the rest of the image, leading to **blurred or distorted** reconstructions because low-rank models cannot reliably infer high-frequency detail from distant areas.

---

### **Alternating Least Squares (ALS) vs. Gradient Descent (GD)**

When optimizing the low-rank factors, the choice of algorithm such as Alternating Least Squares (ALS) or Gradient Descent (GD) introduces a critical performance trade-off.

* **For block-missing cases**, Gradient Descent often shows superior performance, achieving a lower Root Mean Square Error (RMSE) and a higher Peak Signal-to-Noise Ratio (PSNR). Its gradual optimization process appears better suited for inferring spatial dependencies when large, structured regions are missing.

* **For random missing pixels**, the trend reverses. ALS achieves a lower RMSE and higher PSNR, demonstrating a more effective recovery when the missing data is distributed randomly across the image.

The most significant advantage of ALS is its computational efficiency. **ALS consistently converges in far fewer iterations than Gradient Descent.** This rapid convergence makes it a highly efficient and practical algorithm. In summary, while Gradient Descent may provide more accurate reconstructions for structured missing data, ALS offers a much faster solution that excels when missing pixels are scattered randomly.

---------------------------------



## **Objective 2: Data Compression(Observations)**


The experiment demonstrates the relationship between image patch complexity, the chosen rank (`r`), and the quality of reconstruction in a lossy compression scheme.

### 1. Impact of Patch Complexity on Reconstruction Quality

The effectiveness of low-rank approximation is fundamentally tied to the information content of the image patch.

* **Single Color Patch (Low Complexity):** The patch selected from `(0,0)` consists of a simple, out-of-focus grassy area. This patch has very low spatial complexity and high correlation between pixels. As a result, even a very low rank ($r=5$) is sufficient to capture almost all the variance in the data. The reconstruction is nearly lossless across all tested ranks, with no visible artifacts. This indicates that the underlying data matrix for this patch is inherently close to a low-rank matrix.

* **2-3 Colors Patch (Medium Complexity):** The patch from `(100,170)` contains the dog's fur, which has some texture and subtle variations in color and shading. This patch has moderate complexity.
    * At a low rank ($r=5$), the reconstruction is decent but noticeably blurry, losing the fine texture of the fur.
    * As the rank increases to $r=10$ and $r=25$, the quality significantly improves, with finer details being progressively restored. This shows that more "basis components" (a higher rank) are needed to accurately represent the texture.

* **~5 Colors Patch (High Complexity):** The patch from `(150,100)` is the most complex, containing sharp edges, multiple distinct colors (pink tongue, dark lip, red collar), and high-frequency details.
    * At a low rank ($r=5$), the reconstruction is extremely poor, showing severe blocky artifacts, color bleeding, and a complete loss of detail. The result is a blurry, discolored mess.
    * Increasing the rank to $r=10$ and $r=25$ substantially improves the quality, but even at $r=25$, the reconstruction is not perfect and still appears softer and less detailed than the original. This demonstrates that a high-rank matrix is required to represent the high information content of this patch.

### 2. The Role of Rank (*r*)

The rank `r` acts as a bottleneck, controlling the amount of information that can be preserved.

* For all patches, increasing the rank `r` consistently improves the reconstruction quality (visually and as measured by RMSE/PSNR). A higher rank allows the factorized matrices to store more complex patterns and details.

*  For the "Single Color" patch, the visual and metric-based improvements are negligible after $r=5$. The reconstruction quality saturates early because the patch contains little information to begin with. Using a higher rank (e.g., $r=25$) offers no practical benefit and reduces the compression efficiency.

*  Since the patch size is $N \times N$ (where $N=50$), a rank of $r=50$ allows for a full-rank representation. Therefore, the reconstruction at $r=50$ should be (theoretically) perfect, achieving an RMSE of nearly 0 and a very high PSNR. This serves as the upper bound for reconstruction quality, where no compression is actually achieved.

### 3. Conclusion

This effectively illustrates the core trade-off in low-rank data compression. Low-rank factorization is an excellent and highly efficient method for compressing regions of an image that are simple or repetitive (e.g., sky, walls, blurry backgrounds). However, it struggles to preserve quality in complex regions with sharp edges and fine textures unless a sufficiently high rank is used, which in turn diminishes the benefit of compression. The choice of an optimal rank `r` is therefore a critical decision that depends on the specific content of the data and the desired balance between compression ratio and reconstruction fidelity.





**Note** : The results in the form of images can also be seen in the notebook itself or the results folder