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
