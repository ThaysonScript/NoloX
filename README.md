# NoloX

[![Rust](https://img.shields.io/badge/rust-1.75%2B-orange.svg)](https://www.rust-lang.org/)
[![Python](https://img.shields.io/badge/python-3.11%2B-blue.svg)](https://www.python.org/)
[![CUDA](https://img.shields.io/badge/cuda-12.0%2B-green.svg)](https://developer.nvidia.com/cuda-toolkit)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Based on](https://img.shields.io/badge/based--on-NOLOcam-purple.svg)](https://github.com/doxx/NOLOcam) (Deleted)

**NoloX** is a next-generation, low-latency engine for real-time PTZ video orchestration, broadcasting, and computer vision inference. Re-architected from the ground up as a high-performance evolution of the original **[NOLOcam](https://github.com/doxx/NOLOcam) (Deleted)** ("Never Only Look Once") created by Barrett Lyon, NoloX transitions the legacy Go/Shell pipeline into an asynchronous **Rust Core** workspace with native GPU inference (ONNX Runtime / TensorRT) and a lightweight **Python Worker** for research model fallbacks.

---

## 📜 Origins & Evolution from NOLOcam

The original [NOLOcam](https://github.com/doxx/NOLOcam) (Deleted) project pioneered autonomous PTZ camera tracking using local YOLO inference, spatial coordinate calibration, and priority-based tracking (P1/P2 targets). It demonstrated that hybrid edge/cloud intelligence could solve real-time maritime tracking without relying on expensive, high-latency cloud vision APIs.

**NoloX** takes the core philosophy, spatial mathematics, and tracking logic of NOLOcam and completely re-engineers the underlying technology stack:

| Feature / Module | Original NOLOcam (`doxx/NOLOcam`) | NoloX (Next-Gen Architecture) |
| :--- | :--- | :--- |
| **Language & Core** | Go (Goroutines) + Shell Scripts (`NOLO.sh`) | **100% Asynchronous Rust Workspace** (`tokio`) |
| **Video Pipeline** | OpenCV Go Bindings (`gocv`) + FFmpeg CLI | **Native C-FFI** (`ffmpeg-next`) & Zero-Copy Frames |
| **Memory Management** | Go Garbage Collector (potential p99 stalls) | **Zero-GC Deterministic Memory** (Ownership Model) |
| **AI Vision Engine** | Local CGO / OpenCV Blob processing | **Native CUDA ONNX Runtime** (`ort` crate in Rust) |
| **Research Fallback** | External Standalone CLI Invocation | **gRPC IPC Bridge** to Python ML Worker |
| **Execution** | Multi-process Shell Invocation | **Single Optimized Static Binary** (Monorepo) |

---

## 🏛 System Architecture

NoloX eliminates shell script subprocesses and inter-process disk I/O, replacing them with a unified in-memory pipeline:

```mermaid
flowchart TD
    subgraph Rust_Workspace [NoloX Core - Rust Workspace]
        A[RTSP Stream / PTZ Camera Input] --> B[crates/broadcast - ffmpeg-next]
        B --> C[crates/core - Tokio Async Engine]
        C --> D[Stream Output / NVENC HLS]
        
        B --> E[crates/vision - ONNX/TensorRT]
        E -->|Native CUDA Inference| F[crates/calibration - Spatial Geometry]
        F --> G[crates/commentary - Async LLM Client]
    end

    subgraph Python_Worker [Auxiliary Worker - Python]
        E -- gRPC Fallback (Non-ONNX Models) --> H[services/vision-worker-python]
        H -->|PyTorch / Ultralytics| F
    end
```

### Key Architectural Improvements
* **Zero Garbage Collection Stalls**: Deterministic frame processing at 60+ FPS without runtime pauses.
* **Direct-on-GPU Execution**: ONNX models run natively inside the Rust process space via `ort` with CUDA/TensorRT execution providers.
* **Pure Rust Spatial Geometry**: Re-implemented pixel-to-PTZ unit transformations, letterboxing algorithms, and coordinate mapping using `nalgebra`.
* **Ultra-Low Memory Footprint**: Core orchestration runs with less than 80 MB of RAM usage.

---

## 📁 Repository Structure

```text
nolox/
├── Cargo.toml                   # Cargo Workspace configuration
├── crates/
│   ├── core/                    # Main engine, state orchestration, and APIs
│   ├── broadcast/               # Stream management (FFmpeg, RTSP, NVENC)
│   ├── calibration/             # Linear algebra and spatial mapping (pixel -> meters)
│   ├── commentary/              # Asynchronous LLM client for live commentary
│   └── vision/                  # ONNX Runtime local inference engine (ort)
├── services/
│   └── vision-worker-python/    # Python gRPC worker (PyTorch/FastAPI) for research fallback
├── proto/                       # Protobuf IPC definitions
├── docker/                      # Dockerfiles with CUDA runtime support
└── docs/                        # Legacy NOLOcam mapping specs and architecture docs
```

---

## ⚡ Quick Start

### Prerequisites
* **Rust** (2021 edition) and `cargo`
* **Python** 3.11+ (using `uv` or `poetry`)
* **NVIDIA CUDA Toolkit** 12.0+ (with cuDNN and TensorRT)
* **FFmpeg** 6.0+ installed on system

### 1. Build the Rust Core

```bash
# Clone the repository
git clone [https://github.com/your-username/nolox.git](https://github.com/your-username/nolox.git)
cd nolox

# Build workspace in release mode
cargo build --release
```

### 2. Configure Python Worker (Optional Fallback)

```bash
cd services/vision-worker-python

# Set up virtual environment
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt

# Start gRPC server
python server.py
```

---

## ⚙️ Configuration

NoloX is configured via `Nolo.toml` or environment variables:

```toml
[server]
host = "0.0.0.0"
port = 8080

[media]
rtsp_source = "rtsp://admin:password@192.168.1.100:554/Streaming/Channels/101"
output_hls_path = "/var/www/stream/index.m3u8"
hardware_acceleration = "nvenc"

[vision]
engine = "native" # "native" (Rust ONNX) or "python_fallback" (gRPC)
model_path = "models/yolov8n.onnx"
confidence_threshold = 0.5
p1_targets = ["boat", "kayak"]
p2_targets = ["person", "backpack"]
cuda_device_id = 0
```

---

## 📊 Performance Benchmarks

Comming Soon

---

## 🤝 Attribution & Acknowledgments to Original NOLOcam

NoloX is heavily indebted to the original **NOLOcam** project created by **Barrett Lyon**:

* **Core Vision & Philosophy**: Credit to Barrett Lyon for the "Never Only Look Once" concept, hybrid edge/cloud vision paradigm, and the P1/P2 tracking priority framework.
* **Spatial PTZ Mathematics**: Calibration formulas and letterboxing coordinate transformations were ported directly from the original NOLOcam research.
* **Original Repository**: [github.com/doxx/NOLOcam](https://github.com/doxx/NOLOcam) (Deleted)
* **Forked Repository**: [github.com/ThaysonScript/NoloX](https://github.com/ThaysonScript/NoloX)

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for details.
