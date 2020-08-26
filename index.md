# GSoC 2020 Report

## Introduction

Time-Series Analysis is a wide domain and has various applications in the field of pattern recognition. To assist the analysis various time-domain and frequency-domain descriptors are used. With ever-increasing research in time-series and plethora of descriptors for overfitting specialised tasks, there is a need to find generalised task invariant set of features which perform well on various tasks or on at least domain level like neuroscience. Presently the *hctsa* library is used to extract a vast variety of hidden time series characteristics. But, it has ~7700 features which is computationally expensive for feature-based analysis and also requires Matlab license to run.

In this project our goal was to choose a subset of task invariant features from *hctsa* that is tailored specifically for neuroimaging data and to create an efficient open-source library which can be utilized by the researchers for further analysis.

## Initial Stage

The project started in May after my proposal was accepted by INCF as a student developer in GSoC 2020. I met my mentors (Ben Fulcher and Oliver Cliff) who are from the University of Sydney and working on the applications in neuroscience. 

I started the project by first familiarizing myself with the hctsa library in Matlab by playing around with some dummy data and analysing the results which helped me to understand the data-flow and functionality of the tool. To get started with dataset and understand various applications of *hctsa* in neuroscience I referred to various scientific manuscripts. Parallely, I solved multiple first-issues including stratifying imbalance class, restructuring data loader, and implemented various sanity checks.

The project can be divided into the following subsections:
1. Dataset details and preprocessing
2. Selection of features using clustering
3. Reduced feature set description
4. Implementation and Evaluation


## Dataset Details and Preprocessing
During the start of the coding period, I worked on neuroimaging datasets to apply the feature reduction workflow. Following are the datasets

- DREADD: mouse fMRI data
  - Includes two sub-datasets (Left and Right Certex) with four labelled classes: CAMK, excitatory, PVCre, SHAM.

### Preprocessing
The datasets were normalised and the features were filtered to remove missing and extreme values. However, the final dataset was not sufficiently large for analysis. Thus, for augmenting the dataset, time-series were grouped using subclasses (binary) to create multiple subtasks. These subtasks will act as a realm for generalization of the reduced feature set. We therefore have created 12 tasks for two datasets.

## Selecting Features using Clustering

Hierarchical clustering is used here to reduce the feature set based on low redundancy (choosing one from a cluster) and high variability (considering all the clusters).

### Improve the feature reduction pipeline and testing the hyperparameters of hierarchical clustering

- Hierarchical clustering helped to form multiple clusters of correlated features.
  - There are two hyperparameters here: clustering threshold (ø) and the number of top features (n) based on classification performance.
  - We selected the cluster centers and formed the reduced feature set. 
  - Used Average linkage method and abscorr as distance metric.
- Searched for the combination of parameter values which trade-offs between the classification performance and number of features in the set.
- To avoid overfitting, we modified the current feature reduction pipeline and implemented the leave-one-task-out cross-validation method, described below:
  - In this method we leave out one task from the dataset and run the redundancy pipeline on the rest of the tasks. Once we have a reduced set of features, we evaluated its performance on the left-out task.
<br /> ![Alt text](avg_mouse_leave-out.png?raw=true "Title")

## Reduced feature set description

We extracted a reduced set of 16 features from 16 clusters formed (on Mouse fMRI dataset and with hyperparameters: n = 100, ø = 0.2 ). Some features were infeasible to implement in C, so we replaced it with the highly correlated feature in their respective clusters.

Following are the list of features selected:
- AC_nl_035
- AC_nl_036
- AC_nl_112
- SY_DriftingMean50_min
- DN_RemovePoints_absclose_05_ac2rat
- ST_LocalExtrema_n100_diffmaxabsmin
- CO_TranslateShape_circle_35_pts_statav4_m
- CO_TranslateShape_circle_35_pts_std
- SC_FluctAnal_2_dfa_50_2_logi_r2_se2
- IN_AutoMutualInfoStats_diff_20_gaussian_ami8
- PH_Walker_momentum_5_w_momentumzcross
- PH_Walker_biasprop_05_01_sw_meanabsdiff
- FC_LoopLocalSimple_mean_stderr_chn
- CO_HistogramAMI_even_10_3
- CO_HistogramAMI_even_2_3
- CO_AddNoise_1_even_10_ami_at_10

## Implementation and Evaluation

To create static library for the reduced feature set, I re-implemented the selected hctsa features function (originally in Matlab) in C.

To reproduce the feature functionality, wherein stochastic property is retained we incorporated GSL library in C to use Mersenne Twister random generator Algorithm to replicate the randomization compared to Matlab function. It was also used for linear robust fitting and nonlinear least square method (Trust Region 2D subspace).

I have finally implemented a wrapper of the C code to make it available in Matlab (using Mex) and Python (using CPython) programming languages.

### Speed Gains
![Alt text](time.png?raw=true "Speed Gains")

### The reduced feature set vs full set performance

## Future roadmap

- Greedy Method
- Continue the work with EEG data
