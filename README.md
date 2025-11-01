# Anomaly-Detection
Detecting anomalies in time-series, tabular, or sequential data using the power of Transformers and reconstruction-based learning.
# Thesis Description
Detecting anomalies in human motion from skeleton data using reconstruction-based autoencoders. This project implements and compares multiple models, with a focus on a Transformer-based Autoencoder for unsupervised anomaly detection. Other baselines include LSTM, CNN, and a standard MLP autoencoder.
Ideal for applications in surveillance, healthcare monitoring, fall detection, and activity recognition using time-series skeleton data (e.g., from Kinect or pose estimation models).

# Methodology
# Step 1: Data Discovery & Folder Organization
The dataset is organized in a hierarchical structure under a base directory. Training and testing data are separated into distinct folders:

Normal training videos: Located in train/ under subfolders starting with N_.
Normal test videos: Located in test/ under subfolders starting with N_.
Abnormal test videos: Located in test/ under subfolders starting with ABN_.

Specific activities are excluded to maintain focus on meaningful motion patterns:

Excluded from normal: Curtain, Walk_Into_Room_and_Sit
Excluded from abnormal: Curtain, Fall, Quick_Standing_Sitting

An automated discovery function recursively scans the directory tree to compile three lists:
train_normal, test_normal, and test_abnormal.

# Step 2: Frame Loading & Preprocessing
For each video:
All .npy files in the skeletons/ subfolder are loaded and stacked into a 3D tensor of shape (T, J, 3) — where T is the number of frames and J is the number of joints.
Root joint centering: The coordinates of the root joint (index 0) are subtracted from all joints to achieve translation invariance.
Outlier removal:

Each feature is flattened and standardized using Z-scores.
Values exceeding an absolute Z-score of 3.0 are marked as outliers and replaced with NaN.
Missing values are filled using linear interpolation along the time dimension.

Feature augmentation:
First-order velocity is computed as the frame-to-frame difference.
Position and velocity features are concatenated, resulting in a feature dimension of J × 6.
Normalization:
A Min-Max scaler is fitted on the training normal data to scale all features to the range [-1, 1].
The same scaler is applied to validation and test sequences.




# Step 3: Sequence Generation with Sliding Window

Fixed sequence length: 30 frames
Overlap: 15 frames (step size = 15)
Each video is converted into multiple overlapping sequences of shape (30, J, 6).
Data augmentation is applied only to training normal sequences (2 augmentations per sequence):
Random rotation in the XY plane within [-10°, +10°]
Addition of Gaussian noise with standard deviation 0.005
Augmentations are performed before preprocessing to ensure physical consistency.

# Step 4: Train-Validation Split

All normal sequences from the training set are combined and shuffled using a fixed random seed.
80% are assigned to the training set
20% are assigned to the validation set
The test sets (normal and abnormal) remain completely untouched during training.

# Step 5: Autoencoder Model Architectures
Four autoencoder variants are implemented and compared. All take input of shape (batch, 30, J, 6) and output a reconstruction of the same shape

ModelKey ComponentsTransformer AutoencoderEmbedding layer → Positional encoding → 6-layer Transformer encoder (8 attention heads, dropout 0.2) → Linear decoderLSTM Autoencoder1-layer LSTM encoder (hidden size 4) → Latent vector (size 2) → Repeated latent input → LSTM decoderCNN Autoencoder1D convolutional encoder (channels: input to 8 to 16 to 4) → Transpose convolutional decoderStandard AutoencoderFully connected MLP: input to 64 to 32 to 16 (latent) to 32 to 64 to output

# Step 6: Training Procedure

Loss Function: Mean Squared Error (MSE) between input and reconstructed sequence
Optimizer: Adam with learning rate 1e-3
Batch Size: 64
Maximum Epochs: 30
Learning Rate Scheduler: Reduce on plateau (factor 0.5, patience 3)
Early Stopping: Training stops if validation loss does not improve for 5 consecutive epochs
Hardware: GPU (CUDA) if available; otherwise CPU

Only normal sequences are used during training and validation.

# Step 7: Anomaly Detection & Evaluation

## Reconstruction Error Calculation:
MSE is computed for each sequence in validation and test sets.
Error is averaged over time, joints, and features.

## Threshold Selection:
A subset of abnormal validation sequences is combined with normal ones.
200 candidate thresholds are evaluated using percentiles of validation errors.
The threshold maximizing F1-score is selected.
Test-Time Evaluation:
Reconstruction errors are computed on full test normal and test abnormal sets.
## Metrics Reported:
AUROC: Measures overall separability (higher = better)
Mean MSE & RMSE: Separately for normal and abnormal sequences
Confusion Matrix: True Negatives, False Positives, False Negatives, True Positives at optimal threshold

# Visualization & Comparison:
Histograms of reconstruction errors (normal vs. abnormal) for each model
Summary table comparing all four models on key metrics

# Impact of Project on Healthcare & NHS

Real-time fall detection in elderly care homes and hospitals, reducing fall-related injuries (NHS cost: ~£2.3B/year).
Early identification of abnormal gait or movement in stroke/neurological rehab, enabling personalized treatment adjustments.
Automated monitoring of at-risk patients (dementia, post-surgery) in wards, reducing need for 1:1 nursing supervision.
Privacy-preserving (skeleton-only data) surveillance using existing CCTV or low-cost depth sensors.
Home-based telehealth via edge devices — supports NHS virtual wards and aging-in-place initiatives.
Reduces A&E visits and readmissions through proactive anomaly alerts to NHS 111/GP systems.
Cost savings: Prevents one fall = £3,200–£10,000 saved; frees nursing time for critical care.
Scalable & deployable in NHS trusts with minimal infrastructure — aligns with NHS Digital Transformation goals.
