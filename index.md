# GSoC 2020 Report

#### Organization : International Neuroinformatics Coordinating Facility (INCF)
#### Project : A reduced time-series feature library to efficiently characterize neural dynamics
#### Student : Imran Alam
#### Mentors : Ben Fulcher, Oliver Cliff and Joseph Lizier
---
## Introduction

Time-Series Analysis is a broad, interdisciplinary field.
Research in time-series analysis has produced a plethora of features for capturing dynamical patterns.
But it is unknown which of these thousands of existing methods perform well on specific domain-related tasks such as prominent applications involving neural dynamics.
In this task, we aimed to efficiently reduce an existing library of ~7700 time-series features (from the [*hctsa*](https://github.com/benfulcher/hctsa) package) to a reduced subset and implement the result as an open-source library.
The resulting feature set should be highly computationally efficient relative to *hctsa*, give high performance on the set of training tasks (mouse fMRI manipulations), and eliminate the dependency on a Matlab licence, thereby enabling more widespread (and real-time) adoption of feature-based time-series analysis in medical and research applications.

<!--**[[IMRAN: Add a couple sentences here about the broad strategy: i.e., that we're going to use a procedure based on [catch22](https://github.com/chlubba/catch22) to first find features that perform best across a range of tasks, then reduce the redundancy across this set to find maximally independent features, and finally code them into C, with wrappers for common coding environments like python.]]**-->

In this project, we have used a similar workflow as presented in [*catch22*](https://github.com/chlubba/catch22), which selects the top features based on classification performance on the given set of tasks and performs the redundancy method to get the maximally independent features. The feature set was coded in C and also made wrappers for Matlab and Python programming languages.

## Getting Started

I started the project by first familiarizing myself with the *hctsa* library in Matlab by playing around with some test data and analysing the results which helped me to understand the data-flow and functionality of the tool.
To get started with the dataset and understand various applications of *hctsa* in neuroscience, I studied a range of relevant scientific manuscripts.
In parallel, I solved multiple issues related to data analysis and coding including stratifying imbalance class, restructuring the data loader, and implementing sanity checks.

The project can be divided into the following subsections:

1. Dataset details and preprocessing
2. Selection of features using clustering
3. Final reduced feature set
4. Implementation and Evaluation

## Dataset

### Details

During the start of the coding period, I worked on neuroimaging datasets to apply the feature reduction workflow.

We used a mouse fMRI dataset, based on chemogenetic manipulations of local neural dynamics published in a recent paper in [Cerebral Cortex](https://doi.org/10.1093/cercor/bhaa084).
The dataset includes two sub-datasets (a brain area stimulated in the right hemisphere and its contralateral left-hemisphere analogue in the isocortex) with four labelled classes: CAMK, excitatory, PVCre, SHAM.

### Preprocessing

The datasets were normalised and the features were filtered to remove missing or extreme values.
For each pairs of conditions, we extracted a binary classification task, yielding six binary tasks per hemisphere, and thus 12 tasks across two datasets.
The performance of each individual time-series feature was judged according to its performance across these twelve tasks.

## Selecting Features using Clustering

Hierarchical clustering is used here to reduce the feature set based on low redundancy (choosing one from a cluster) and high variability (considering all the clusters).

### Improve the feature reduction pipeline and testing the hyperparameters of hierarchical clustering

- Hierarchical clustering helped to reduce redundancy across the top-performing features by forming clusters of highly correlated features.
  - There are two relevant hyperparameters: clustering threshold (`ø`) and the number of top features (`n`) based on classification performance.
  - We selected a reduced feature set as the centers of each cluster using average linkage clustering with absolute correlation coefficient (`abscorr`) distances.
- We searched for the combination of parameter values which trade-offs between the classification performance and the number of features in the set.
- To avoid overfitting, we modified the current pipeline and implemented the leave-one-task-out cross-validation method, as described below:
  - In this method, we leave out one task from the dataset and run the redundancy pipeline on the rest of the tasks.
  - Once we have a reduced set of features, we evaluated its performance on the left-out task.

<!--**[[IMRAN: Add a paragraph describing this plot in context (what is the 'accuracy' measure, what are the numbers, etc.)]]**-->

| ![Alt text](avg_mouse_leave-out.png?raw=true "Leave-task")
|:--:| 
| We applied different threshold (`ø`) values to restrict the number of features in the reduced-set. Here, the color represents the balanced accuracy on the left-out task using the reduced-set and the labelled number is the total number of features in the set. |


## Final reduced feature set

We extracted a reduced set of 16 features from the 16 clusters formed (on Mouse fMRI dataset and with hyperparameters `n = 100` and `ø = 0.2`).
Some features were infeasible to implement in C, so we replaced it with the highly correlated feature in their respective clusters.

The list of features selected (one from each cluster) is as follows:
- `AC_nl_035`
- `AC_nl_036`
- `AC_nl_112`
- `CO_AddNoise_1_even_10_ami_at_10`
- `CO_HistogramAMI_even_2_3`
- `CO_HistogramAMI_even_10_3`
- `CO_TranslateShape_circle_35_pts_statav4_m`
- `CO_TranslateShape_circle_35_pts_std`
- `DN_RemovePoints_absclose_05_ac2rat`
- `FC_LoopLocalSimple_mean_stderr_chn`
- `IN_AutoMutualInfoStats_diff_20_gaussian_ami8`
- `PH_Walker_biasprop_05_01_sw_meanabsdiff`
- `PH_Walker_momentum_5_w_momentumzcross`
- `SC_FluctAnal_2_dfa_50_2_logi_r2_se2`
- `ST_LocalExtrema_n100_diffmaxabsmin`
- `SY_DriftingMean50_min`


## Implementation and Evaluation

- To create a static library for the reduced feature set, I re-coded these 16 *hctsa* features from Matlab to C.

- To reproduce the feature functionality, wherein the stochastic property is retained we incorporated the GSL library in C to use Mersenne Twister random generator Algorithm to replicate the randomization compared to Matlab function.
  It was also used for linear robust fitting and nonlinear least square method (Trust Region 2D subspace).

- I have finally implemented a wrapper of the C code to make it available in Matlab (using Mex) and Python (using CPython) programming languages.

### Performance Comparison

<!--**[[IMRAN: The key test is whether the reduced set gives similar/better performance on the problems--please include some of these results?]]**-->

It is evident from the scatter plot that the reduced feature set gives similar and in some cases better performance than the full *hctsa* feature set. Thus, in memory and time-constraint environments the *hctsa* can be replaced by *catchaMouse16*.

| ![Alt text](perf.png?raw=true "Performance comparison")
|:--:| 
| Performance of *catchaMouse16* in comparison to the *full-set*. Each scatter point represents a single task with balanced accuracy. The error bars along the X and Y axis are the standard deviations of cross-validated accuracy with *full-set* and *catchaMouse16* respectively.|


### Speed Gains
<!--**[[IMRAN: Add a description of what is plotted, and describe in words why it's relevant/interesting]]**-->

We compared the average execution time to run the reduced feature set on 2000 time-series of length 900, from *hctsa* (in `Matlab`) and *catchaMouse16* library (in `C` and `Mex`). 
Our developed library is ~60 times faster than *hctsa* as shown in the comparison plot:

![Alt text](time.png?raw=true "Speed Gains")


### Summary
<!--**[[IMRAN: Add a summary of the final repository, with a link to the repo, and some reflections on why it was interesting and will likely to useful]]**-->

The [*catchaMouse16*](https://github.com/NeuralSystemsAndSignals/catchaMouse16) library consists of 16 *hctsa* features efficiently coded in C, which was selected by following the [*op_importance*](https://github.com/imraniac/op_importance) feature reduction pipeline. 
- It is easily integrable in large-scale projects with options to choose from programming languages like Python and Matlab. 
- It is useful for neuroscience applications on huge time-series datasets.

## Future Roadmap

<!--**[[IMRAN: Remove, or replace with proper detail that a reader could understand:]]**-->
Even after the end of the GSoC period, I would continue contributing to open-source development for the neuroscience community.

- I plan to publish a scientific paper describing the novelty in our method
- Extending the work to aim for a better methodology which follows a multivariate feature selection approach.
