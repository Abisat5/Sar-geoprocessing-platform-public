# SAR Geoprocessing & Automated Core Platform

A modular, microservice-ready geospatial platform designed for advanced Synthetic Aperture Radar (SAR) imagery preprocessing and distributed deep learning orchestration. This public repository showcases the core foundational pipeline for civilian remote sensing applications, such as environmental change detection, natural disaster monitoring, and smart urban planning.

---

## Architectural Overview & Core Pipeline

The platform establishes a seamless, production-grade engineering pipeline that processes raw radar telemetry maps and prepares them for neural network feature extraction.

### 1. Telemetry Ingestion & Simulation Module (`sar_processor.py`)
- Simulates or ingests high-resolution radar matrices (GeoTIFF / Sentinel-1 format compatibility).
- Handles foundational radiometric calibration pipelines.

### 2. Despeckling Engine (Signal Processing)
- Implements a programmatic Lee Filter utilizing local variance and spatial mean convolutions to eliminate multiplicative speckle noise (salt-and-pepper artifacts inherent to radar signals).
- Performs Logarithmic Decibel ($10 \cdot \log_{10}$) scaling to optimize pixel distribution for neural network convergence.

### 3. Multi-Task Deep Learning Backbone (`atr_detector.py`)
- Features a customized Convolutional Neural Network (CNN) backbone leveraging Instance Normalization and Leaky ReLU activations for complex feature preservation.
- Utilizes a parallel multi-head architecture consisting of:
  - **Classification Head**: Computes spatial grid probability maps across civilian targets.
  - **Regression Head**: Predicts bounding box coordinates relative to localized grid frameworks.

### 4. Multi-Task Optimization Layer (`multi_task_loss.py`)
- Implements a hybrid loss function bridging:
  - **Cross-Entropy Loss** for semantic categorization.
  - **Mean Squared Error (MSE) Loss** for bounding box regression.
- Features an advanced object masking technique that mathematically isolates coordinate regression penalties strictly to grid coordinates containing verified targets.

---

# Quick Start & Integration Verification

To run the full end-to-end integration test locally, execute the consolidated pipeline test script. This script verifies the integrity of the signal processing filter, PyTorch tensor formatting, forward-pass mapping, and backpropagation mechanics.

## Prerequisites

Ensure your localized virtual environment (`venv`) has the following enterprise dependencies installed:

```bash
pip install torch torchvision scipy numpy opencv-python
```

## Execution

Run the verification wrapper from the root of the project directory:

```bash
python src/train_and_test.py
```

---

# Expected Test Vector Output

Upon successful execution, the pipeline initializes the neural weights, filters the simulated radar noise, handles the loss masking matrices, and delivers the following mathematical verification output in the console:

```plaintext
====================================================
      SAR GEOPROCESSING PLATFORM - INTEGRATION TEST
====================================================

[1] Hardware Acceleration: Processing pipeline initialized on [CPU].

[2] Running Signal Processing Pipeline...
--> Generating raw simulated synthetic telemetry matrix...
--> Executing Speckle Noise Elimination (Lee Filter)...
--> Logarithmic dB transformation complete. Output Tensor Shape: (512, 512)

[3] Initializing Deep Learning Core Architecture...
--> Multi-Task Detection Heads successfully configured and cached.
--> Multi-Task Masked Loss Criterion loaded.

[4] Running End-to-End Forward & Backward Pass (1 Iteration Test)...

================ INTEGRATION RESULTS ================
Classification Probability Map Shape : torch.Size([1, 5, 64, 64])
Bounding Box Regression Map Shape   : torch.Size([1, 4, 64, 64])
Computed Total Pipeline Loss (Multi-Task): 4.1535
 -> Class Categorization Penalty         : 1.6536
 -> Bounding Box Alignment Penalty       : 1.2564
=====================================================

[SUCCESS] Core integration pipeline executed with zero exceptions.
```

---

# Technology Stack & Engineering Practices

### Deep Learning Framework
- PyTorch (Core Neural Architectures)

### Signal & Matrix Engineering
- NumPy
- SciPy
- OpenCV

### Architectural Patterns
- Multi-Task Learning
- Decoupled Grid-Based Localization
- Aspect Vector Masking

### Software Engineering Standards
- Test-Driven Infrastructure
- Strict Variable-Type Isolation
- Clean Modular Code Architecture
