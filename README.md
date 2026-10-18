# VisionBasedPlanning

## Semantic Traversability
The semantic traversability component generates a per-pixel cost map from RGB images using pretrained SAM 2 and CLIP models.

SAM 2 is used to segment the image into regions. Each segment is then evaluated with CLIP using traversability-related text prompts. The CLIP scores are converted into traversability costs, where lower values indicate more traversable terrain and higher values indicate less traversable or non-traversable regions.

The semantic pipeline is implemented in:
```text
notebooks/semantic_cost_map.ipynb
