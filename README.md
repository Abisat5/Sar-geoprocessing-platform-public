# SAR Geoprocessing & Automated Core Platform

A modular, microservice-ready geospatial platform designed for advanced Synthetic Aperture Radar (SAR) imagery preprocessing and distributed deep learning orchestration. This public repository showcases the core foundational pipeline for civilian remote sensing applications, such as environmental change detection, natural disaster monitoring, and smart urban planning.

## Before / After — Multi-Sensor Geospatial Display

Coastal zone analysis at the same operational scale (512 px grid). The comparison illustrates the evolution from a baseline fused frame to an enhanced fusion product with improved contrast, radar overlay compositing, and structured target metadata.

<table>
  <tr>
    <td align="center" width="50%">
      <strong>Before</strong><br />
      <sub>Baseline fused optical + radar frame</sub><br /><br />
      <img src="docs/images/fusion-before.png" alt="Before: baseline multi-sensor geospatial display" width="100%" />
    </td>
    <td align="center" width="50%">
      <strong>After</strong><br />
      <sub>Enhanced fusion, overlay, and HUD export</sub><br /><br />
      <img src="docs/images/fusion-after.png" alt="After: enhanced multi-sensor geospatial display" width="100%" />
    </td>
  </tr>
</table>

> **Note:** This repository contains the **foundational, open-source-safe** components of the platform. Operational data-ingestion services, tactical visualization layers, and production API integrations are maintained in a separate private development environment and are summarized here at a high level only.

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

## Platform Evolution & Recent Capabilities

The following items describe engineering milestones achieved on the extended platform branch. They are documented here for transparency without exposing proprietary endpoints, credentials, or deployment-specific configuration.

### Geospatial Data Ingestion (Coordinate-Driven)
- Replaced brittle, path-dependent remote file access patterns with **latitude/longitude–centric window extraction**.
- Introduced live **optical basemap tiling** for high-resolution contextual imagery aligned to a fixed analysis grid (e.g., 512×512).
- Eliminated placeholder synthetic map fallbacks in favor of **fail-fast, auditable ingestion** when upstream sources are unavailable.

### Multi-Sensor Data Fusion Architecture
- Designed a **dual-stream fusion model** that registers optical RGB and SAR-derived channels on a **shared geographic reference frame**.
- SAR channels are supplied as a **two-band I/Q–style tensor** compatible with the existing complex-valued CNN front-end.
- Separation of concerns:
  - **Inference stream** → radar tensor for detection heads.
  - **Visualization stream** → fused optical + radar products for human review.

### SAR Signal Path (Research → Operational)
- Early prototypes used **optically derived SAR proxies** for rapid UI and model integration testing.
- Current direction integrates **real public SAR products** (e.g., Sentinel-1 family) via **cloud-optimized GeoTIFF / STAC catalogue** access patterns, with optional local GeoTIFF overrides for offline or air-gapped workflows.
- Metadata discovery aligns with standard **civilian Earth observation catalogues**; authentication is handled through environment-level credentials (not committed to this repository).

### Geospatial Image Enhancement (Visualization Pipeline)
- **Histogram stretching** (percentile-based contrast normalization) for clearer land–water–structure separation.
- **Unsharp masking** for edge-preserving crispness on optical basemaps.
- **Speckle suppression** on radar amplitude layers (box/local filtering) prior to overlay compositing.
- Semi-transparent **radar heatmap overlay** on optical imagery for intuitive multi-sensor interpretation.

### Tactical Situation Display (Export Quality)
- High-DPI raster export suitable for briefing materials and after-action review.
- Monospace HUD-style annotations: target lock, coordinates, fusion status, and signal-strength readouts.
- Configurable grid and contrast settings to preserve fine structure (harbors, vessels, coastal infrastructure) under overlay graphics.

### Service-Oriented Analytics Layer (Private Branch)
- REST endpoint pattern for **on-demand target-detection / zone analysis** from geographic coordinates.
- Startup-time model weight hydration from checkpoint storage.
- Structured JSON responses including tensor shape metadata and fusion provenance fields.

### Training Pipeline Alignment
- Dataset loaders updated to consume **real SAR windows** where catalogue access is configured.
- Training scripts decoupled from invalid granule archive URLs in favor of catalogue-driven COG reads.
- ASF-style scene search retained for **metadata and mission planning**, not as a substitute for pixel-accurate COG streaming.

---

## What Is Intentionally Not in This Repository

To protect ongoing research and operational security, the public tree **does not** include:

| Category | Reason |
|----------|--------|
| API server implementation & routes | Production deployment surface |
| Live tile server URLs & API keys | Credential hygiene |
| Catalogue subscription / Earthdata setup | User-specific secrets |
| Tactical visualizer source & styling constants | Operational UI/IP |
| Trained model checkpoints | Size, sensitivity, reproducibility |
| Domain-specific targeting logic | Application-specific |

Contributors and reviewers should treat this README as a **capability manifest**; implement details live in the private monorepo.

---

## Quick Start & Integration Verification

To run the full end-to-end integration test locally, execute the consolidated pipeline test script. This script verifies the integrity of the signal processing filter, PyTorch tensor formatting, forward-pass mapping, and backpropagation mechanics.

### Prerequisites

Ensure your localized virtual environment (`venv`) has the following enterprise dependencies installed:

```bash
pip install torch torchvision scipy numpy opencv-python rasterio
```

### Execution

Run the verification wrapper from the root of the project directory:

```bash
python src/train_and_test.py
```

> When connecting to external SAR catalogues, additional optional dependencies (STAC client libraries, HTTP tooling) may be required in the private environment. Those are not mandatory for the core integration test in this repository.

---

## Expected Test Vector Output

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

## Technology Stack & Engineering Practices

### Deep Learning Framework
- PyTorch (Core Neural Architectures)

### Signal & Matrix Engineering
- NumPy
- SciPy
- OpenCV
- Rasterio (GeoTIFF / COG window reads — extended branch)

### Geospatial Interoperability (Extended Branch)
- STAC catalogue discovery patterns
- Cloud-Optimized GeoTIFF partial reads
- Dual-sensor (optical + SAR) co-registration

### Architectural Patterns
- Multi-Task Learning
- Decoupled Grid-Based Localization
- Aspect Vector Masking
- Multi-sensor data fusion (optical context + SAR inference)

### Software Engineering Standards
- Test-Driven Infrastructure
- Strict Variable-Type Isolation
- Clean Modular Code Architecture
- Fail-fast ingestion (no silent synthetic substitutes in production paths)

---

## Roadmap (Public Summary)

| Phase | Status | Focus |
|-------|--------|--------|
| Core | Done | Lee filtering, dB scaling, multi-task CNN, masked loss |
| Fusion design | Done | Shared-grid optical + SAR tensors, overlay visualization |
| SAR catalogue ingestion | In progress | Catalogue-authenticated real SAR COG ingestion |
| Production scale | Planned | Hardened API gateway, batch inference, reproducible training manifests |

---

## License & Attribution

This project processes publicly available Earth observation data when configured to do so. Users are responsible for complying with the terms of any third-party data provider they connect to in their private deployment. No provider credentials or endorsements are implied by this repository.
