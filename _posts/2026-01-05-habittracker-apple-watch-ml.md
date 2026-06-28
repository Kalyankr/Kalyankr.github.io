---
layout: post
title: "Detecting habits from an Apple Watch with machine learning"
subtitle: "Sensor data, sliding window features, and honest batch level evaluation"
tags: [machine learning, sensor fusion, time series, apple watch, swift]
author: Kalyan Reddy Katla
---

This project detects a physical habit (in my case, touching my hair) from Apple Watch
motion sensors. It is an end to end system: a Swift watch app collects data, and an ML
pipeline turns raw motion into predictions.

I built it to test a simple idea: if a wrist wearable can recognize an unconscious habit
in real time, it can nudge you the moment it happens, which is the first step toward
breaking it.

The watch streams acceleration, rotation rate, gravity, and magnitude. I label events in
real time with a simple state: YES while doing the habit, NO otherwise, and the label
persists until I change it, which gives dense supervision. Each recording session is
stored as a batch.

The pipeline slides a configurable window over the signal and extracts features per axis
(mean, standard deviation, max, and energy), then trains a Random Forest with class
balancing.

![From raw Apple Watch motion to a YES or NO prediction](/assets/img/posts/habittracker-pipeline.svg)

I evaluate with precision, recall, and F1 plus a confusion matrix, all computed on
held-out batches so the scores reflect new recording sessions rather than memorized ones.

### Challenges I faced

1. Temporal leakage. The biggest trap in time series is letting windows from the same
   session land in both train and test. I split by batch so evaluation is honest, which
   dropped the flattering numbers but made them real.
2. Class imbalance. The habit is rare, so most windows are NO. Class weighting plus
   probability threshold tuning and temporal smoothing were needed to get usable
   precision.
3. Real world noise. Hands move in endless ways, so features that worked in one session
   failed in another. Collecting multiple batches was the only fix.
4. Building the data collector. Getting clean, labeled sensor data off the watch was
   harder than the modeling.

### What I learned

For sensor and time series problems, your evaluation protocol matters more than your
model. A Random Forest with honest batch level splits told me more truth than a fancier
model with leaky validation. I also learned how much of applied ML is really data
collection and feature engineering, not the classifier.

Code: [github.com/Kalyankr/habittracker](https://github.com/Kalyankr/habittracker)
