---
layout: page
title: 3D Vision
full_title: "A Unified Point-Based Framework for 3D Segmentation"
authors: Hung-Yueh Chiang, Yen-Liang Lin, Yueh-Cheng Liu, Winston H. Hsu
description: A Unified Point-Based Framework for 3D Segmentation
img: assets/img/publication_preview/jpb.png
importance: 10
category: research
---

<div style="text-align: center; padding-bottom: 1rem;">
<abbr class="badge" style="background-color:#00369f; margin-left:0.1rem; margin-right:0.1rem; font-size:1.1rem;">3DV 2019</abbr>
</div>

<div class="authors"> <a href="https://hychiang.info">Hung-Yueh Chiang</a><sup>1</sup>, Yen-Liang Lin<sup>2</sup>, <a href="https://liu115.github.io/">Yueh-Cheng Liu</a><sup>1</sup>, <a href="https://winstonhsu.info/">Winston H. Hsu</a><sup>1</sup></div>
<div class="authors" style="padding-bottom: 0.5em;"><sup>1</sup>National Taiwan University
<sup>2</sup>A9/Amazon</div>

<div style="text-align: center; padding-bottom: 0.9em;">
<p style="font-size:1.1rem;">[<a href="https://github.com/ken012git/joint_point_based">code</a>] &nbsp; [<a href="https://www.youtube.com/watch?v=rqBcxw6iaD0">talk</a>]</p>
</div>


<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/jpb/jpb.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

<div style="text-align: center;">
    <p style="font-family: cursive; font-size:1.5rem">
    :rocket: Top performing method in 2019
    </p>
</div>

<br>

# Motivations

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
    <ul style="font-size:1.2rem">
        <li>Normal describes object shapes</li>
        <li>Color shows object textures</li>
        <li>Global scene prior is crucial for semantics</li>
    </ul>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/jpb/motivation.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


# Method Overview

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/jpb/overview.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="rows" style="font-size:1.2rem">
We extract image features by applying a 2D segmentation network and backproject the 2D image features into 3D space. Both 2D image features and 3D point features are concatenated in the same point coordinates. The sub-volume and scene encoder extract the features for local details and global context priors respectively. The decoder fuses the local and global features and generate the semantic predictions for each point in the sub-volume.
</div>

<br>

# Main Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/jpb/main_results.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="rows" style="font-size:1.2rem">
Results on ScanNet testing set. Our unified framework outperforms several state-of-the-art methods by a large margin. The gain
comes from jointly optimizing 2D image features, 3D structures and global context in a point-based architecture. With synthetic camera
poses, we can further improve our performance (+1.3% gains)
</div>
<br>

# Citation
{% raw %}
```latex
@inproceedings{chiang2019unified,
  title = {A unified point-based framework for 3d segmentation},
  author = {Chiang, Hung-Yueh and Lin, Yen-Liang and Liu, Yueh-Cheng and Hsu, Winston H},
  booktitle = {International Conference on 3D Vision (3DV)},
  pages = {155--163},
  year = {2019},
  organization = {IEEE},
}
```
{% endraw %}

<br>
# Acknowledgements
This work was supported in part by the Ministry of Science and Technology, Taiwan, under Grant MOST 108-2634-F-002-004 and FIH Mobile Limited. We also benefit from the NVIDIA grants and the DGX-1 AI Supercomputer.