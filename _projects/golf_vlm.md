---
layout: page
title: Spatially-aware tiny VLM for egocentric golf videos
description: Teaching a compact on-device VLM to understand golf swings from AR-glasses video | 2026
img: /assets/img/samsung.png
importance: 2
category: work
related_publications: false
---
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/golf/head.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Golf videos recorded from AR glasses are hard for today's Vision-Language Models (VLMs). The models have seen very little first-person golf footage, and they struggle with depth and with fast motion such as a swinging club head or a ball leaving the tee. As a result, even large VLMs often cannot tell **when** the club hits the ball, let alone **how good** the shot was. On top of that, the target product needs a **small model** that runs with low latency on the device.

This project closes that gap. We give a large "teacher" VLM extra spatial knowledge during training, then pass its reasoning on to a tiny "student" VLM that only sees the raw video frames at run time.

The fine-tuned model can:

- **Detect impact**: decide whether the ball is struck in a clip, and at which frame.
- **Evaluate the shot**: comment on the quality of the swing and the strike.
- **Give feedback**: talk to the player both as a **coach** and as a friendly **companion**.

---

#### Stage 1: Recovering 3D motion from video

We built an automated pipeline that combines object detection and tracking with 3D reconstruction to recover the real-world 3D trajectories of the golf ball and the club head. Every module is pluggable, so a stronger detector or reconstruction model can be swapped in later.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/golf/pipeline.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Overview of the trajectory-extraction pipeline.
</div>

The reconstructed trajectories are also useful on their own. For example, a player could replay each shot in 3D and follow the ball's flight afterwards.

<div class="row justify-content-center">
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/golf/traj_1.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/golf/traj_2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Reconstructed golf-ball trajectories in 3D. Colors run from red to purple over time.
</div>

From the relative motion of the ball and the club head, we can also estimate shot metrics such as the **smash factor** (ball speed divided by club-head speed). But this purely geometric approach turned out to be fragile: a single missed detection upstream can throw off the result. This told us that robust impact understanding needs the holistic, visual reasoning of a VLM, and that the trajectories work best as **supporting evidence** rather than as the final answer.

<div class="row justify-content-center" style="width:80%; margin: 0 auto;">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/golf/impact.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Estimating impact and smash factor from reconstructed club-head and ball speeds.
</div>

---

#### Stage 2: Curating training data with a privileged teacher

A small model is only as good as its training data. We used the 3D trajectories as **privileged information**: extra knowledge that the large teacher VLM can use while writing training examples, but that the student never sees.

Before any sample reaches training, it goes through a **multi-round quality filter**. The filter combines automatic consistency checks on the trajectories with human verification of the impact moment. Only clips with clear images and reliable motion data are kept.

<div class="row justify-content-center" style="width:80%; margin: 0 auto;">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/proj/golf/filter.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Multi-round filtering produces high-quality training data.
</div>

The teacher is asked to explain its reasoning **from what is visible in the frames**, not just to state an answer. A dedicated safeguard then rejects any output that leaks trajectory-derived details the student could not know. This way the extra knowledge makes the teacher's reasoning sharper without teaching the student to make things up.

---

#### Stage 3: Fine-tuning the tiny VLM

We fine-tuned a compact open-source VLM on the curated data using parameter-efficient training. The training objective puts extra emphasis on the decisions that matter most: whether an impact happened, and when.

At run time the student needs **only the image sequence**, with no trajectories and no 3D reconstruction, so it can run on the device.

---

#### Results

Compared with the same model before fine-tuning:

- **Impact detection**: the original model never detected a single impact. The fine-tuned model detects impacts with an F1 score of about **70%** and **very high precision**, with almost no false alarms.
- **Impact timing**: when it detects an impact, it finds the exact frame about **3 out of 4 times**, and is within one frame **over 90%** of the time.
- **Feedback quality**: its coaching and companion comments are about **twice as long**, use a much wider vocabulary, and are far less repetitive.

Overall, this project shows a practical path from spatially rich offline annotation to a lightweight, **image-only** golf assistant that can spot the impact, judge the shot, and talk with the player as both a coach and a companion.
