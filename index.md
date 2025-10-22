---
title: "Project 2 – Image Homography and Warping"
---

<link rel="stylesheet" href="assets/css/style.css">

<!-- Tiny extras to style dropdowns and captions nicely -->
<style>
  details { margin: 16px 0; }
  details > summary {
    list-style: none;
    cursor: pointer;
    padding: 10px 12px;
    border-radius: 12px;
    background: rgba(16,18,39,0.55);
    border: 1px solid rgba(255,255,255,0.06);
    color: #FFC857;
    font-weight: 700;
  }
  details[open] > summary {
    box-shadow: 0 0 18px rgba(255,200,87,0.25);
  }
  .space-img {
    border-radius: 16px;
    box-shadow: 0 10px 30px rgba(6,7,18,0.6),
                0 0 0 1px rgba(255,255,255,0.06),
                0 0 24px rgba(111,76,255,0.25);
    transition: transform .25s ease, box-shadow .25s ease;
  }
  .space-img:hover {
    transform: scale(1.015);
    box-shadow: 0 14px 36px rgba(6,7,18,0.65),
                0 0 42px rgba(108,43,217,0.35),
                0 0 22px rgba(255,200,87,0.35) inset;
  }
  .space-caption {
    color: #FFC857;
    font-style: italic;
    opacity: .9;
  }
  .hr { height:1px; margin:24px auto; max-width: 900px;
        background:linear-gradient(90deg,transparent,rgba(255,200,87,.5),transparent); }
        @keyframes starPulse {
  0%   { filter: drop-shadow(0 0 0px rgba(255,200,87,0.0)); transform: scale(1);}
  50%  { filter: drop-shadow(0 0 8px rgba(255,200,87,0.6)); transform: scale(1.08);}
  100% { filter: drop-shadow(0 0 0px rgba(255,200,87,0.0)); transform: scale(1);}
}
.star {
  display:inline-block;
  animation: starPulse 2.5s ease-in-out infinite;
}

h1 { text-align:center; }
h1 .star { margin:0 6px; font-size:1.1em; }
  
mjx-container {
  color: #FFC857;              /* golden math text */
  text-shadow: 0 0 6px rgba(255,200,87,0.3); /* subtle glow */
  opacity: 0;
  transition: opacity 1s ease;
}
mjx-container[MathJax-Finished="true"] {
  opacity: 1;
}
</style>

# Project 2: <span class="star">🌟</span> Image Homography and Warping

This project explored how 2D transformations can manipulate and align images using homography and warping.  
**Part 1** focuses on rectification (manual correspondences).  
**Part 2** poster projection, then an automatic pipeline (vanishing points, masking, contour scoring).  
**Part 3** compares **Triangular Mesh (Piecewise Affine)** vs **Thin Plate Spline (TPS)** warping.

<div style="background:rgba(16,18,39,0.55); border:1px solid rgba(255,255,255,0.08);
border-radius:12px; padding:12px 18px; margin:18px 0; color:#FFC857;
box-shadow:0 0 12px rgba(111,76,255,0.25); font-weight:500;">
<span class="star">💫</span> <strong>Tip:</strong> Many sections below are <em>clickable</em> — tap the golden pulsing stars to expand or collapse detailed discussions, results, and reflections.
</div>

<div class="hr"></div>

## Part 1 — 🌌 Homography & Manual Rectification

<!-- Step: Input -->
<details open>
<summary><strong>1) Input capture and goal <span class="star">✨</span> </strong></summary>

I started with a photo of a flat paper taken at an angle. The goal: compute a **homography** that maps the four corners to a rectangle, producing a **fronto-parallel rectified view**.
</details>

<p align="center">
  <img src="images/paper_photo.jpg" width="65%" class="space-img">
  <br><span class="space-caption">Input photo (oblique view of a planar surface).</span>
</p>

<!-- Step: Rectified -->
<details>
<summary><strong>2) Corner correspondences → Homography (DLT) → Rectification <span class="star">✨</span></strong></summary>

I manually clicked the poster corners, assembled source→target correspondences, and solved for the homography \(H\) using DLT.  
Applying inverse warping produced a rectified output where **straight lines stay straight**, and the surface appears as if the **camera moved** to face it head-on.
</details>

<p align="center">
  <img src="images/correspondences.png" width="65%" class="space-img">
  <br><span class="space-caption">Manual correspondences (used only for debugging checks).</span>
</p>

<p align="center">
  <img src="images/rectified.png" width="65%" class="space-img">
  <br><span class="space-caption">Rectified result: planarity preserved, perspective corrected.</span>
</p>


<div class="hr"></div>
<details>
<summary><strong> Discussion – How homography enables rectification <span class="star">✨</span>  </strong></summary>

Homography mathematically defines the projective relationship between two planes.  
By estimating the matrix \(H\) from four or more corresponding points, the image can be re-projected so that the plane in the photo maps directly onto a fronto-parallel view.  
This effectively *undoes perspective distortion*: lines that were converging due to camera angle become parallel again, and the surface appears as though the camera were facing it head-on.  
In practice, this lets any planar region—posters, signs, or screens—be rectified into a measurable, undistorted view.
</details>


##  Part 2 — 🌌 Planar Warping & Texture Placement

<!-- Step: Virtual poster -->
<details>
<summary><strong>1) Virtual texture placement <span class="star">✨</span> </strong></summary>

To validate the rectified geometry, I overlaid a virtual poster/texture in the rectified domain and reprojected it.  
Because homography respects planar projective geometry, the texture sits realistically on the surface without bending.
</details>

<p align="center">
  <img src="images/virtual_poster.jpg" width="65%" class="space-img">
  <br><span class="space-caption"> texture to be projected.</span>
</p>


<details open>
<summary><strong>2) Baseline – Rectified AR Poster Projection <span class="star">✨</span> </strong></summary>

Part 2 begins by extending the rectification work from Part 1 into a simple **augmented-reality placement** task.  
Using the computed homography, I projected a poster image directly onto a planar surface in a new scene.  
Because the transform preserves projective geometry, the poster adheres convincingly to the wall without bending—this forms the **baseline result** required for Part 2.
</details>

<p align="center">
  <img src="images/scene-1.jpg" width="65%" class="space-img">
  <br><span class="space-caption">Original scene prior to poster projection.</span>
</p>


<p align="center">
  <img src="images/ar_insert.png" width="65%" class="space-img">
  <br><span class="space-caption">Initial projection using clicked points.</span>
</p>

---

<details>
<summary><strong>3) Observations and Limitations of the Baseline <span class="star">✨</span> </strong></summary>

The baseline successfully made the poster appeared naturally attached to the surface.  
However, it still relied on manual point correspondences.  
i wanted to automatically determine points: in new scenes, the quad had to be chosen by hand, and noise or camera angle changes could break the illusion.  
These observations motivated my attempt to **automate the planar region detection and point choice** process.
</details>

---

<details>
<summary><strong>4) Extension – Toward Automatic Placement <span class="star">✨</span> </strong></summary>

To remove manual selection, I experimented with using the image’s geometry itself to infer placement regions.  
I detected strong line segments with **Canny + Hough**, grouped them by orientation, and estimated **vanishing points** to sketch perspective-aligned quads.  
This allowed the system to propose plausible surfaces autonomously—essentially guessing where a wall or poster might exist.
</details>


---

<details>
<summary><strong>5) Added Improvements – Masking and Quad Scoring <span class="star">✨</span> </strong></summary>

While promising, the first automatic results were unstable in cluttered or low-contrast scenes.  
To improve consistency, I added:

- a **keep-mask** to down-weight shadows and saturated areas,  
- and **contour scoring** to prefer quads with near-right angles and balanced sides.  

These filters helped the algorithm ignore distracting edges and choose more reliable surfaces.
</details>

<p align="center">
  <img src="images/vp_lines.png" width="60%" class="space-img"><br>
  <span class="space-caption">Vanishing-point-oriented line detection.</span>
</p>
<p align="center">
  <img src="images/mask_debug.png" width="60%" class="space-img"><br>
  <span class="space-caption">Heuristic keep-mask filtering shadows and highlights.</span>
</p>
<p align="center">
  <img src="images/quad_debug.png" width="60%" class="space-img"><br>
  <span class="space-caption">Candidate quadrilateral proposals (debug view).</span>
</p>

---

<details>
<summary><strong>6) Final Hybrid Result <span class="star">✨</span> </strong></summary>

By combining **vanishing-point inference**, **mask filtering**, and **contour scoring**, the system achieved robust, hands-free texture placement across multiple scenes.  
The resulting projection required no user input and maintained realistic perspective even under challenging conditions.
</details>

<p align="center">
  <img src="images/ar_keep_mask.png" width="65%" class="space-img">
  <br><span class="space-caption">Improved automatic placement with mask + scoring.</span>
</p>

<p align="center">
  <img src="images/ar_hybrid.png" width="65%" class="space-img">
  <br><span class="space-caption">Hybrid method result — geometry + heuristics for stable warping.</span>
</p>

---

<details>
<summary><strong> Discussion – Idea, Successes, and Challenges <span class="star">✨</span> </strong></summary>

**Idea.**  Use projective geometry to place textures automatically, removing manual correspondences.  

**What worked.**  The homography-based projection preserved realism, and the added heuristics made the placement self-sufficient in most scenes.  

**Challenges.**  Highly cluttered backgrounds or uneven lighting could still mislead the vanishing-point solver, occasionally generating “floating” placements.  
Even so, these experiments demonstrated that careful geometric reasoning alone can achieve near-AR-quality placement without learning-based assistance.
</details>

<div class="hr"></div>

## Part 3 — 🌌 Triangular Mesh vs Thin-Plate Spline (TPS)

<!-- Step: Digit verification -->
<details open>
<summary><strong>1) Verification on digits (3-a → 3-b) <span class="star">✨</span> </strong></summary>

I first warped one “3” to another using both **Piecewise Affine (PWA)** and **TPS**.  
PWA is piecewise linear (triangles), continuous but not smooth across edges; TPS is globally smooth, minimizing bending energy while interpolating landmarks exactly.  
Both passed a landmark RMSE ≈ 0 check.
</details>

<p align="center">
  <img src="images/3warp.png" width="60%" class="space-img">
  <br><span class="space-caption">Digit “3” warp comparison (PWA left vs TPS right).</span>
</p>

<!-- Step: Cat → Bulbasaur -->
<details>
<summary><strong>2) Cat → Bulbasaur smile (creative comparison) <span class="star">✨</span> </strong></summary>

Warping my cat (Saleen) to match **Bulbasaur’s** grin made the differences pop:
- **PWA**: locally rigid triangles create a faceted, wedge-like reshaping near the mouth.  
- **TPS**: landmarks act like pins in a rubber sheet—mouth pulls propagate globally, “inflating” the lower face.

Neither is “wrong”—they’re each **correct** under their continuity assumptions. TPS is global + smooth; PWA is local + linear.
</details>

<p align="center">
  <img src="images/cat-bulbasaurwarp.png" width="65%" class="space-img">
  <br><span class="space-caption">“Catasaur” — aligning a cat’s facial landmarks to Bulbasaur’s smile.</span>
</p>

<!-- Step: Summary panel -->
<details>
<summary><strong>3) Discussion & takeaways <span class="star">✨</span> </strong></summary>

**Rectification vs Warping.** Rectification with a homography is like **moving the camera** so the plane faces you.  
Warping is like **bending the surface** in 2D to force points to new locations.

**Why TPS bends the whole image.**  
TPS solves  
$$ f(\mathbf{p}) = a_0 + a_x x + a_y y + \sum_i w_i\,U(\|\mathbf{p}-\mathbf{c}_i\|), \quad U(r)=r^2\log r^2. $$
Because \(U\) has global support, moving a landmark influences the entire field smoothly.

**Why PWA looks faceted.** PWA triangulates the domain and applies affine maps per triangle (C⁰ continuous, not C¹).  
It preserves local linearity but introduces visible seams if landmarks are sparse or far apart.

**Practical guidance.**  
Use **PWA** when you want localized, geometry-faithful changes (rigid-ish regions).  
Use **TPS** for organic, globally smooth deformations (faces, handwriting).  
Adding **anchor points** (cheeks/chin) localizes TPS; adding **more triangles** smooths PWA transitions.

</details>

<p align="center">
  <img src="images/part3.png" width="65%" class="space-img">
  <br><span class="space-caption">Part 3 overview panel.</span>
</p>

<div class="hr"></div>

## Conclusions 🌠

- **Homography** gives planar, projective alignment (rectification) — it preserves straight lines and mimics moving the camera.  
- **PWA** provides local, piecewise-linear control; good for rigid patches.  
- **TPS** enforces global smoothness; great for non-rigid alignment, but deforms broadly unless anchored.  
- The Cat→Bulbasaur demo shows both are mathematically correct—just different continuity assumptions and deformation behavior.

**Author:** Joseph Janicki · **Course:** Computer Vision · **Term:** Fall 2025


