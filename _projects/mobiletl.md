---
layout: page
title: MobileTL
full_title: "MobileTL: On-device Transfer Learning with Inverted Residual Blocks"
authors: Hung-Yueh Chiang, Natalia Frumkin, Feng Liang, Diana Marculescu
description: On-device Transfer Learning with Inverted Residual Blocks
img: assets/img/publication_preview/mobiletl.png
importance: 9
category: research
---

<div style="text-align: center; padding-bottom: 1rem;">
<abbr class="badge" style="background-color:#00369f; margin-left:0.1rem; margin-right:0.1rem; font-size:1.1rem;">AAAI 2023</abbr>
<abbr class="badge" style="background-color:#FF9B00; margin-left:0.1rem; margin-right:0.1rem; font-size:1.1rem;">Oral Paper</abbr>
</div>

<div class="authors"> <a href="https://hychiang.info">Hung-Yueh Chiang</a>, Natalia Frumkin, <a href="https://jeff-liangf.github.io/">Feng Liang</a>, <a href="https://users.ece.utexas.edu/~dianam/">Diana Marculescu</a></div>
<div class="authors" style="padding-bottom: 0.5em;">The University of Texas at Austin</div>

<div style="text-align: center; padding-bottom: 0.9em;">
<p style="font-size:1.1rem;">[slides] &nbsp; [poster] &nbsp; [demo]</p>
</div>


<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/mobiletl_main.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

<div style="text-align: center;">
    <p style="font-family: cursive; font-size:1.5rem">
    :rocket: <b>1.5 <span>&#215;</span> less FLOPS </b> &nbsp; &nbsp;
    :small_red_triangle_down: <b>2<span>&#215;</span> memory reduction for Inverted Residual Blocks</b>
    </p>
</div>

<br>

# Motivations

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
    <ul style="font-size:1.2rem">
        <li>IRBs are widely used in mobile-friendly models</li>
        <li>IRBs are designed for inference efficiently on edge devices</li>
        <li>Training IRB-based models on edge devices is <b>challenging</b></li>
        <li>IRBs require <b>3<span>&#215;</span> more activation storage</b> for backward pass</li>
    </ul>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/motivation.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

# Method

<ul style="font-size:1.2rem">
<li>Freeze low-level layers and fine-tune high-level layers</li>
</ul>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/trainable.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>
<ul style="font-size:1.2rem">
<li>Freeze global statistics and update bias only for intermediate normalization layers</li>
</ul>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/bn_layers.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>
<ul style="font-size:1.2rem">
<li>Approximate activation layer’s backward as a signed function</li>
</ul>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/act_layers.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

# Overview of a MobileTL Block

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/overview.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="rows" style="font-size:1.2rem">
MobileTL is an efficient training scheme for IRBs. To avoid storing activation maps for two normalization layers after expansion and depthwise convolution, we only train shifts, and freeze scales and global statistics. The weights in convolutional layers are trained as usual. To adapt the distribution to the target dataset, we update the scale, shift, and global statistics for the last normalization layer in the block. MobileTL approximates the backward function of activation layers, e.g., ReLU6 and Hard-Swish, by a signed function, so only a binary mask is stored for activation backward computing. Our method reduces the memory consumption by 46.3% and 53.3% for MobileNetV2 and V3 IRBs.
</div>

<br>

# Main Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/mobiletl/results.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="rows" style="font-size:1.2rem">
We experiment MobileTL with Proxyless Mobile (Cai, Zhu, and Han 2019) and transfer to eight downstream tasks.
We compare our method (Prox. MobileTL in orange) with vanilla fine-tuning a few IRBs of the model (Prox. FT-BLKs in grey)
and TinyTL (Cai et al. 2020). MobileTL maintains Pareto optimality for all datasets and improves accuracy over TinyTL in six
out of eight datasets.
</div>
<br>

# Citation
{% raw %}
```latex
@inproceedings{chiang2022mobiletl,
  title = {MobileTL: On-device Transfer Learning with Inverted Residual Blocks},
  author = {Chiang, Hung-Yueh and Frumkin, Natalia and Liang, Feng and Marculescu, Diana},
  booktitle = {Proceedings of the AAAI Conference on Artificial Intelligence (AAAI)},
  year = {2023},
}
```
{% endraw %}

<br>
# Acknowledgements
This research was supported in part by NSF CCF Grant No. 2107085, NSF CSR Grant No. 1815780, and the UT Cockrell School of Engineering Doctoral Fellowship. Additionally, we thank <a href="https://d31003.github.io/">Po-han Li</a> for his help with the proof.