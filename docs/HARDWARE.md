# Hardware Platform

This document provides a detailed overview of the hardware architecture powering CLARITY, the rationale behind component selection, and the role each subsystem plays in achieving low-latency, real-time speech enhancement.

---

## Overview

CLARITY employs a two-stage edge-computing architecture designed to balance computational efficiency, audio quality, and affordability.

The system separates responsibilities between:

1. A dedicated audio front-end processor responsible for capturing and conditioning speech signals.
2. An embedded edge-computing platform responsible for executing the DeepFilterNet3 neural speech enhancement pipeline.

This division minimizes computational overhead, reduces latency, and allows the AI model to operate on cleaner audio inputs.

---

## Hardware Architecture

```text
Acoustic Environment
         │
         ▼
┌──────────────────────┐
│   ReSpeaker Lite     │
│   (XMOS XU316 DSP)   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Raspberry Pi 4     │
│  DeepFilterNet3 AI   │
└──────────┬───────────┘
           │
           ▼
     Enhanced Audio
```

---

## Hardware Components

| Component              | Role                                |
| ---------------------- | ----------------------------------- |
| Raspberry Pi 4 Model B | Edge AI inference engine            |
| ReSpeaker Lite         | Audio capture and DSP preprocessing |
| DeepFilterNet3         | Neural speech enhancement           |
| ONNX Runtime           | Optimized model inference           |
| LADSPA                 | Low-latency audio integration layer |

---

# Raspberry Pi 4 Model B

<p align="center">
  <img src="../assets/raspberry-pi-4.png" width="500">
</p>

The Raspberry Pi 4 serves as the primary compute platform for CLARITY.

Unlike conventional hearing-assistive devices that rely on proprietary DSP hardware, CLARITY uses a general-purpose embedded Linux platform capable of running modern machine learning workloads locally and in real time.

---

## Key Specifications

| Specification    | Value                                       |
| ---------------- | ------------------------------------------- |
| Processor        | Broadcom BCM2711                            |
| CPU              | Quad-Core ARM Cortex-A72 (64-bit)           |
| Clock Speed      | Up to 1.8 GHz                               |
| Memory           | 1–8 GB LPDDR4                               |
| Connectivity     | USB 3.0, USB 2.0, Ethernet, WiFi, Bluetooth |
| Storage          | MicroSD                                     |
| Operating System | Raspberry Pi OS (64-bit)                    |

---

## Why Raspberry Pi 4?

The Raspberry Pi 4 was selected because it offers a unique balance between performance, cost, software support, and deployment flexibility.

### Sufficient Processing Power

DeepFilterNet3 requires continuous execution of neural network inference alongside real-time audio processing.

The Cortex-A72 architecture provides enough computational throughput to execute the model significantly faster than real time while maintaining a low power envelope.

### Linux Ecosystem

The platform supports:

* ALSA
* LADSPA
* ONNX Runtime
* Python
* Rust
* C/C++

This dramatically simplifies deployment and experimentation.

### Edge AI Compatibility

The Raspberry Pi supports a wide variety of machine learning frameworks and serves as an ideal prototyping platform before migrating to custom embedded hardware.

### Cost Effectiveness

Compared to commercial assistive audio systems, the Raspberry Pi provides significant computational capability at a fraction of the cost.

---

# ReSpeaker Lite (XMOS XU316)

<p align="center">
  <img src="../assets/respeaker-lite.jpg" width="500">
</p>

The ReSpeaker Lite functions as the front-end audio acquisition and preprocessing subsystem.

Rather than forwarding raw microphone signals directly to the AI model, the board performs multiple DSP operations before the audio reaches the inference engine.

This dramatically improves signal quality and reduces the burden on the neural network.

---

## Key Specifications

| Specification           | Value                           |
| ----------------------- | ------------------------------- |
| Audio Processor         | XMOS XU316                      |
| Microphones             | Dual Digital MEMS Array         |
| Signal-to-Noise Ratio   | 64 dBA                          |
| Sensitivity             | -26 dBFS                        |
| Acoustic Overload Point | 120 dBL                         |
| Sampling Rate           | Up to 16 kHz                    |
| Connectivity            | USB Audio Class 2.0             |
| Audio Output            | Speaker Connector / 3.5 mm Jack |

---

## Built-In DSP Features

The XMOS XU316 includes hardware-accelerated audio processing algorithms:

| Feature                          | Purpose                       |
| -------------------------------- | ----------------------------- |
| Beamforming                      | Focuses on directional speech |
| Acoustic Echo Cancellation (AEC) | Removes speaker feedback      |
| Noise Suppression (NS)           | Reduces environmental noise   |
| Automatic Gain Control (AGC)     | Maintains consistent volume   |
| Voice-to-Noise Ratio (VNR)       | Improves speech detection     |

---

## Why ReSpeaker Lite?

### Far-Field Speech Capture

The dual-microphone array enables speech acquisition up to approximately three meters away from the source, making it suitable for classroom environments.

### Hardware-Level Noise Reduction

Many common acoustic artifacts can be removed before they ever reach the neural network.

This increases enhancement quality while reducing computational demand.

### XMOS Processing Engine

The XU316 is specifically designed for audio workloads and can execute beamforming and signal conditioning with extremely low latency.

### USB Audio Compatibility

The board appears as a standard USB audio device under Linux, simplifying integration with ALSA and the rest of the CLARITY pipeline.

---

# Hardware Selection Strategy

Several alternative platforms were evaluated during development.

| Platform              | Limitation                               |
| --------------------- | ---------------------------------------- |
| Arduino-Based Systems | Insufficient compute resources           |
| ESP32-Based Solutions | Unable to execute DeepFilterNet3 locally |
| Commercial FM Systems | Proprietary and expensive                |
| Jetson-Class Devices  | Increased cost and power consumption     |

The Raspberry Pi 4 and ReSpeaker Lite combination provided the best balance of:

* Cost
* Performance
* Open-source compatibility
* Community support
* Deployment flexibility

---

# Why a Two-Stage Architecture?

A common design mistake in edge-AI audio systems is forcing a neural network to solve every problem in the pipeline.

CLARITY instead divides responsibilities:

### Stage 1: DSP Processing

Handled by ReSpeaker Lite.

Responsibilities:

* Beamforming
* Echo cancellation
* Gain control
* Preliminary noise suppression

### Stage 2: AI Enhancement

Handled by Raspberry Pi 4.

Responsibilities:

* Semantic speech enhancement
* Fine-grained speech reconstruction
* Dynamic noise removal
* Harmonic restoration

This layered design improves efficiency while preserving speech quality.

---

# Scalability & Future Hardware Roadmap

While the current hardware platform serves as a robust proof of concept, future iterations may explore:

### Model Quantization

Reducing inference requirements through:

* INT8 Quantization
* Quantization-Aware Training (QAT)
* Structured Pruning

### Dedicated Neural Accelerators

Potential integration with:

* Hailo AI Accelerators
* Google Coral TPU
* Intel Movidius

### Wearable Deployments

Long-term development may migrate portions of the pipeline to:

* ESP32-S3
* Neural DSP platforms
* Custom embedded audio processors

to reduce power consumption and support all-day battery-powered operation.

---

# References

* Raspberry Pi 4 Official Documentation:
  https://www.raspberrypi.com/products/raspberry-pi-4-model-b/

* ReSpeaker Lite Documentation:
  https://www.seeedstudio.com/ReSpeaker-Lite-p-5928.html

* DeepFilterNet Repository:
  https://github.com/rikorose/deepfilternet
