# 3D Mesh Normalization and Advanced Quantization Pipeline

![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)
![Libraries](https://img.shields.io/badge/libraries-numpy%20%7C%20trimesh%20%7C%20open3d%20%7C%20sklearn-green.svg)

This project, for the Mixar assignment, implements and analyzes pipelines for 3D mesh preprocessing. It demonstrates standard normalization (Min-Max, Unit Sphere) and an advanced, robust method using **Principal Component Analysis (PCA)** and **Adaptive Quantization**.

## 📖 Table of Contents

- [Final Results: At a Glance](#-final-results-at-a-glance)
- [Project Analysis & Findings](#-project-analysis--findings)
  - [Part 1: Uniform Quantization Trade-Off](#part-1-uniform-quantization-trade-off)
  - [Part 2: Bonus - PCA & Adaptive Quantization](#part-2-bonus---pca--adaptive-quantization)
  - [Part 3: Final Comparison](#part-3-final-comparison)
- [How to Run](#-how-to-run)
  - [1. Installation](#1-installation)
  - [2. Execution](#2-execution)
- [Project Structure](#-project-structure)

---

## 🚀 Final Results: At a Glance

The primary goal was to find a method that **preserves mesh shape** while **minimizing reconstruction error**. The advanced **PCA + Adaptive** pipeline was the clear winner, dramatically outperforming standard methods.

**Final Error Comparison (`person.obj`)**
| Method | Total Mean Squared Error (MSE) | Shape Preserved? |
| :--- | :--- | :--- |
| Unit Sphere (Uniform 1024) | `0.00000176` | **Yes** |
| Min-Max (Uniform 1024) | `0.00000079` | No (Distorted) |
| **🏆 PCA + Adaptive (Bonus)** | **`0.00000026`** | **Yes** |

---

## 🔬 Project Analysis & Findings

### Part 1: Uniform Quantization Trade-Off

The main tasks (1-3) were run on three diverse meshes (`person.obj`, `table.obj`, `talwar.obj`) to ensure the pipeline was robust. This test immediately revealed a classic trade-off:

1.  **Min-Max Normalization:** Stretches each axis (X, Y, Z) to the same [0, 1] range.
    * **Pro:** Low reconstruction error (MSE).
    * **Con:** Severely **distorts the mesh's shape** and aspect ratio.
2.  **Unit Sphere Normalization:** Scales the entire mesh uniformly to fit in a sphere.
    * **Pro:** **Perfectly preserves the mesh's original shape**.
    * **Con:** Higher reconstruction error, as it doesn't efficiently use the [0, 1] range.

This trade-off is visualized in the error plots. For an AI model that needs to understand *shape*, Unit Sphere is better, but its high error is a problem.

| `person.obj` Error | `table.obj` Error | `talwar.obj` Error |
| :---: | :---: | :---: |
| ![person.obj plot](person.obj_error_plot.png) | ![table.obj plot](table.obj_error_plot.png) | ![talwar.obj plot](talwar.obj_error_plot.png) |

### Part 2: Bonus - PCA & Adaptive Quantization

The bonus task was to build a superior pipeline that solves the problems above. This was done in two steps:

1.  **Rotation Invariance (PCA):** Instead of simple scaling, **Principal Component Analysis (PCA)** was used to find the "natural" axes of the mesh. The mesh was then *rotated* to align with these axes *before* normalization. This ensures the mesh is always oriented the same way, making the process **robust to random rotations**.

2.  **Adaptive Quantization:** Instead of a *uniform* 1024 bins, we analyzed the local vertex density.
    * Dense, high-detail areas (like a face) were given **more bins (e.g., 2048)**.
    * Sparse, flat areas (like a back) were given **fewer bins (e.g., 256)**.

This "intelligently" spends our data budget, focusing precision only where it's needed.

### Part 3: Final Comparison

The final plot for `person.obj` shows the massive success of this advanced pipeline.

![Final Comparison Plot](person.obj_FINAL_COMPARISON_PLOT.png)

The **PCA + Adaptive** method (light blue) achieved the **lowest error by a significant margin**—3x better than Min-Max and 6.7x better than Unit Sphere—while *also* preserving the mesh's true geometric structure.

---

## 🔧 How to Run

### 1. Installation

The script requires Python 3 and several libraries. Install them via pip:
```bash
pip install numpy trimesh open3d matplotlib scikit-learn scipy
