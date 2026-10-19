# VisionBasedPlanning

## Project Overview
This project investigates vision-based traversability planning using pretrained foundation models and RGB driving sequences. Semantic and geometric information are used to construct traversability cost maps, which are then used for image-space A* path planning.

The project also investigates temporal consistency by comparing independent A* planning, temporal fusion, and RAFT-based optical-flow alignment.

## Semantic Traversability
The semantic traversability component generates a per-pixel cost map from RGB images using pretrained SAM 2 and CLIP models.

SAM 2 is used to segment the image into regions. Each segment is then evaluated with CLIP using traversability-related text prompts. The CLIP scores are converted into traversability costs, where lower values indicate more traversable terrain and higher values indicate less traversable or non-traversable regions.

The semantic pipeline is implemented in:
```text
notebooks/semantic_cost_map.ipynb
```

### How to run
Place input RGB frames in: ```data/```.

Then run ```notebooks/semantic_cost_map.ipynb```.

Generated semantic cost maps are saved to: ```outputs/```.

## Path Planning and Temporal Consistency
The planning component combines semantic and geometric traversability cost maps and uses A* to find low-cost paths through the scene.

Three methods are evaluated:

- **Independent Astar:** Plans independently on each frame's combined cost map.
- **Temporal Fusion + Astar:** Combines consecutive cost maps without motion alignment before planning.
- **RAFT-Aligned Temporal Fusion + Astar:** Uses RAFT optical flow to align the previous cost map with the current frame before fusion and planning.

The planner includes obstacle inflation, safe endpoint selection, cost-aware A* search, path smoothing, and driving corridor visualization.

The planning and evaluation pipeline is implemented in:

notebooks/Vision_Based_Traversability_Planning.ipynb

### How to run

The planning notebook is designed for Google Colab, with input data stored in Google Drive.

Mount Google Drive and configure the dataset directory:

/content/drive/MyDrive/VisionBasedPlanning/datasets/

Each scenario requires three ZIP files:

 - **rgb.zip** - RGB driving frames

- **semantic_costs.zip** - Semantic traversability cost maps

- **geometry_costs.zip** - Geometric traversability cost maps

Run the setup and helper cells, followed by the validation section and the full evaluation.

Results are saved to final_evaluation/scenario_XX/.

### Evaluation

Two experiments are performed:

**Temporal consistency:** Independent A*, temporal fusion, and RAFT-aligned temporal fusion are compared using path stability, path length, obstacle clearance, and planning success.

**Cost-map source comparison:** Semantic-only, geometric-only, and combined cost maps are compared to investigate their influence on path planning.

Experiments are performed on two driving scenarios. The generated results include evaluation metrics, plots, planned paths, and visualization videos.

### Results

The visualization outputs include:

- comparison.mp4 - Comparison of independent A*, temporal fusion, and RAFT-aligned fusion.

- pipeline.mp4 - Semantic, geometric, and combined cost maps, RAFT optical flow, temporal fusion, and planned paths.

- Evaluation CSV files and plots for quantitative comparison.

### Limitations

Monocular depth-based geometric traversability does not consistently identify all obstacles. Semantic classification can also be affected by ambiguous regions and segmentation errors.

Temporal fusion improves consistency but may retain outdated information, while RAFT alignment depends on reliable optical-flow estimation. Furthermore, image-space path stability does not necessarily imply real-world driving safety.
