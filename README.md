# VisionBasedPlanning

## Project Overview
This project investigates vision-based traversability planning using pretrained foundation models and RGB driving sequences. Semantic and geometric information are used to construct traversability cost maps, which are then used for image-space A* path planning.

The project also investigates temporal consistency by comparing independent A* planning, temporal fusion, and RAFT-based optical-flow alignment.

The experiments are conducted on two driving scenarios (scenario_01 and scenario_02). The Google Drive folder contains the input datasets, generated semantic and geometric cost maps, and final evaluation results for both scenarios.

For each scenario, the evaluation results include planning comparison videos, cost-map source comparison videos, and quantitative metrics.

**[Access Datasets and Evaluation Results on Google Drive](https://drive.google.com/drive/folders/1ECtaFKlDo-LRSOHqZEvRgyLDYm73WzgW?usp=drive_link)**

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

## Geometric Traversability

The `geometric_cost_map.ipynb` notebook converts RGB frames from the `data/` folder into a traversability cost map, with values from 0 (traversable) to 1 (not traversable). It relies on depth alone, no semantic information is used. Depth is estimated using the [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2) model, used off the shelf with no additional training or fine-tuning. The cost of each pixel is then computed from how far it sticks out above the estimated ground surface.

#### How it works
1. The depth model gives, for every pixel, a relative inverse depth (larger = closer to the camera). Its scale is arbitrary and changes from one image to another, so each frame is normalized by dividing it by its own maximum.
2. After a few experiments, we determined that on a flat ground this inverse depth grows almost linearly from the horizon to the bottom of the image. We then fit this line on each frame and subtract it from every pixel. What is left (the residual) is about 0 on the ground and positive for everything that is closer to the camera than the ground would be on that row (a bush, a tree, a car).
3. Some rows reduce the quality of the residual map. The hood of the car (when there is one) is much closer than the ground and would pull the line up, so it is detected first and left out of the fit. The sky and the far terrain are detected from the residual, once the line is fitted. This limits the zone we use to a section between two horizontal lines (`r0` at the top, `r1` at the bottom), and both lines are found automatically.
4. The residual is turned into a cost with a linear ramp (0 up to `lo`, 1 from `hi`), and the cost of every row outside of the two lines is set to 1.
5. The costs are rounded and saved, one `.npy` file per frame, and zipped.

## How to use it

The notebook ships with a few frames from [BDD100K](https://huggingface.co/datasets/dgural/bdd100k) and is already configured to use them, so you can open it and run all the cells without changing anything. It is also configured to display this selection (through the `SHOW_FRAMES` list). The optional checks at the end are not needed to produce the maps. To run it on rougher terrain, you'll need to add another dataset (see the section below), then choose it and the depth model in the config cell.

## Output

For each frame, the notebook writes `<frame>_geometric_cost.npy`, i.e. a float32 array with the same height and width as the RGB frame, where 0 = traversable and 1 = not traversable.

## Adding a new dataset

Everything lives in a `data/` folder next to the notebook, with one sub-folder per dataset. Only `images/` has to be created by hand; the notebook creates the rest.

1. Create `data/<dataset>/images/` and copy your frames into it. For the rough-terrain dataset, unzip the RGB images from the shared drive directly into this folder. A frame is identified by its file name without the extension, and all outputs are named after it.
2. Add an entry for the dataset in `PARAMS` (config cell):
   - `b`: the block size. It must divide both the height and the width of the images. Use `1` if you do not need the coarse grid.
   - `shape`: the expected `(height, width)` of the images, or `None` to skip the check.
3. Set `DATASET = "<dataset>"` and run all the cells.

#### Pipeline directory structure 

```
data/
└── <dataset>/
    ├── images/                       <- contains the RGB frames from the dataset (.jpg, .jpeg or .png)
    ├── depth/<model>/                <- created by the notebook, contains one depth.npy per frame + metadata.json
    └── costmaps/<model>/             <- created by the notebook
        ├── geometric/
        │   ├── outputs/              <- one <frame>_geometric_cost.npy per frame
        │   └── run_info.json         <- model, constants and the rows found for each frame
        └── geometric.zip             <- the outputs/ folder zipped
```

Each depth model has its own `depth/<model>/` and `costmaps/<model>/` folders, so trying a bigger model never overwrites the results of a smaller one.

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
