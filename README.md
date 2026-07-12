# Parametric Curve Optimization

## The Final Results
After digging into the math and running the optimization, I successfully extracted the three hidden variables for the parametric curve:
* **Theta ($\theta$):** 30 degrees (0.5236 radians)
* **$M$:** 0.03
* **$X$:** 55

**Final Equation (LaTeX):**
`$\left( t*\cos(0.5236) - e^{0.03|t|} \cdot \sin(0.3t)\sin(0.5236) + 55, 42 + t*\sin(0.5236) + e^{0.03|t|} \cdot \sin(0.3t)\cos(0.5236) \right)$`

**Interactive Graph:**
https://www.desmos.com/calculator/422nw03ijc

---

## My Thought Process & Methodology

## The Initial Approach
When I first looked at the problem, I figured it would be a straightforward curve-fitting exercise. My plan was to write a Python function for the parametric equations, plug in a time array ($t$), and minimize the L1 distance against the provided dataset using SciPy's `differential_evolution`.


I hit a wall as soon as I loaded `xy_data.csv`. The dataset only had `x` and `y` coordinates—there was no time parameter ($t$). Without knowing *when* a specific point was generated, my optimizer couldn't calculate the L1 loss because I couldn't map my predicted points to the target points in time.

###  The Pivot: Rigid Body Transformation
I realized I couldn't just guess $t$; I had to reverse-engineer it from the coordinates. Looking closely at the provided equations:


The $e^{M|t|} \cdot \sin(0.3t)$ term was a massive, highly non-linear headache. To visualize the structure better, I replaced it with a placeholder variable, $U$. By isolating $x$ and $y$, I noticed the equations actually represented a standard **rigid body transformation**:

<img width="1863" height="2645" alt="DocScanner 13-Jul-2026 04-33 AM_1" src="https://github.com/user-attachments/assets/c1bd660e-272b-46b3-9b5a-8bf806a9308d" />


<img width="1819" height="2602" alt="DocScanner 13-Jul-2026 04-33 AM_2" src="https://github.com/user-attachments/assets/8a0ef0a9-6906-4be5-bafe-64526f936727" />


The original curve $(t, U)$ was simply being rotated by angle $\theta$ and translated by the vector $(X, 42)$.

###  The Breakthrough: Inverse Rotation
Since the forward equations apply a rigid body transformation, I could isolate $t$ by mathematically reversing it. By subtracting the translation and applying an inverse rotation matrix, I could completely cancel out that complex $U$ term.

Here is the algebra I used:
1. Subtract the translation constants: $(x - X)$ and $(y - 42)$.
2. Multiply the $x$-equation by $\cos(\theta)$ and the $y$-equation by $\sin(\theta)$.
3. Add the two equations together. The complex $U$ terms cancel each other out completely.
4. Apply the Pythagorean identity ($\cos^2(\theta) + \sin^2(\theta) = 1$) to get a clean formula for time:
**$t = (x - X)\cos(\theta) + (y - 42)\sin(\theta)$**

### 5. Final Code Architecture
With the math solved, the code fell into place. Inside my custom loss function, I used the optimizer's current guesses for $\theta$ and $X$ to dynamically calculate the $t$ array for every point in the CSV. I plugged that $t$ array back into the original parametric equations to get my predicted coordinates, and then calculated the L1 distance.

This allowed SciPy to smoothly bypass the missing data problem, navigate the complex non-convex equations, and pull the exact hidden variables out of the dataset.


## Repository Contents
* `xy_data.csv`: The target dataset.
* `AI_Curve_Fitting.ipynb`: My Jupyter Notebook containing the data pipeline, the custom inverse-rotation L1 loss function, and the SciPy optimizer.
