---
layout: page
title: Speculative sampling of LLM and deployment on auto-platform chips
description: Internship at NIO - Digital Cockpit Department | San Jose, CA, Summer 2024
img: /assets/img/nio.png
importance: 1
category: work
related_publications: false
---
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/nio/1.jpeg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

**[Speculative sampling](https://dl.acm.org/doi/10.5555/3618408.3619203)** is a technique for accelerating large language model (LLM) inference. Even for electric vehicles equipped with high-performance computing platforms, running LLMs locally remains challenging. At the same time, there is a strong commercial need to deploy LLMs locally. For example, to comply with privacy regulations in European regions, users' chat can not be uploaded to the clouds beyond the border. Motivated by this, we explore deploying LLMs directly on the in-vehicle system, enabling their use in a voice assistant application, Nomi.


<div class="row justify-content-center" style="width:60%; margin: 0 auto;">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/nio/2.png" class="rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Speculative sampling combines the target and draft model, achieves the accuracy close to target model and latency close to draft model during the inference phase.
</div>

<div class="row">
    <div class="col-7">
        <p>According to Google Research, speculative sampling integrates a large-scale <strong>target model</strong> with a smaller <strong>draft model</strong>, allowing the final inference output to achieve the accuracy of the target model while benefiting from the faster generation speed of the draft model.</p>

        <p>The in-vehicle computing platform of NIO vehicles consists of NVIDIA Orin and Qualcomm SA-8295 chips. The SA-8295 has a larger memory capacity, and its powerful NPU can accommodate the target model, whereas the NVIDIA Orin has smaller memory and can only host the draft model. Consequently, our task is to run two different language models simultaneously on these heterogeneous chips and share their output logits for speculative sampling. This approach is advantageous because only the logits need to be transfered between the two chips, alleviating concerns about inter-chip communication bandwidth. In summary, the deployment is across the <strong>heterogeneous</strong> in-vehicle devices</p>
        
    </div>
    <div class="col-5">
        <img src="/assets/img/proj/nio/3.png" class="img-fluid" alt="Speculative sampling workflow">

        <div class="caption">
            The workflow of speculative sampling
        </div>
    </div>
</div>

A major challenge during deployment from the fact that Qualcomm was still exploring AI computation on its chip at the time. Many operators were not supported by the compilation platform, which required rewriting certain functions, such as <em>top-k</em>. Beyond this project, other observed issues included limitations on implementing siamese networks \( i.e., architectures with two encoders processing different inputs but sharing parameters\) which were not allowed by the Qualcomm compilation platform. In such cases, it was necessary to define two separate encoders to avoid reusing the same structure. Overall, this process involved a substantial amount of engineering effort.

Ultimately, local inference on the in-vehicle platform achieved a **2.5× speedup** compared to running a target model solely on the SA-8295 NPU.

