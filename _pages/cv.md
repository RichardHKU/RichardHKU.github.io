---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

My research focuses on **Efficient and Reliable AI Systems**. As foundation models continue to grow in parameter count, context length, and deployment scale, the central challenge facing AI systems is no longer simply insufficient compute. Instead, it is the increasingly severe systemic mismatch among computation, memory, data movement, communication, numerical precision, and hardware reliability.

I seek to address a question that spans algorithms, models, architectures, and emerging devices:

> **How can advanced AI models achieve efficient, stable, and trustworthy training, inference, and generation on real-world hardware that is resource-constrained and imperfect?**

To answer this question, I explore cross-layer co-design from machine-learning algorithms to computing hardware. My research investigates emerging computing platforms—including Compute-in-Memory (CIM), heterogeneous memory systems, and package-aware architectures—as well as the computational structure of foundation models themselves, including long-context attention, KV caches, speculative decoding, diffusion language models, parameter-efficient adaptation, and low-bit model representations. Beyond improving throughput, energy efficiency, and memory footprint, I also study the deeper effects of approximate computing, analog noise, and device nonidealities on model behavior—particularly how they alter generation trajectories, routing decisions, internal states, and reasoning capabilities.

My research is organized around four closely connected directions.

---

### 1. Efficient AI Computing Platforms

For large-scale AI workloads, energy consumption and latency often arise not primarily from arithmetic operations, but from frequent data movement, analog-to-digital conversion, memory accesses, chip-to-chip communication, and package-level interconnects. I therefore study how new computing interfaces, heterogeneous architectures, and system-level modeling can enable more efficient AI computing platforms.

In the context of compute-in-memory, I explore stochastic computing interfaces, configurable-precision representations, and ADC-less analog computing mechanisms to reduce the overhead of data conversion and numerical representation while improving adaptability to diverse models and tasks. I also investigate heterogeneous multi-core CIM systems for AI training and LLM training. By analyzing the coupling among computation, memory capacity, bandwidth, communication, and packaging, I develop design-space exploration and performance-modeling frameworks for these systems.

The central goal of this research direction is to move beyond optimizing individual compute units and instead understand and alleviate data-movement and integration bottlenecks at the system level.

---

### 2. Hardware-Aware Model Representation and Adaptation

Models are not fixed workloads. Their numerical representations, precision configurations, parameter structures, and adaptation mechanisms can substantially affect efficiency, accuracy, and deployability on a target hardware platform. I therefore investigate how models themselves can actively adapt to hardware resource constraints and computational characteristics.

Specifically, I study quantization, mixed precision, non-uniform precision allocation, and adaptive integer representations. My goal is to assign appropriate numerical budgets according to the information importance of different layers, channels, or model components. In analog CIM settings, I further co-design quantization strategies with hardware noise, device characteristics, and hybrid memory-cell structures, enabling model representations that are not only more compact but also more resilient to underlying computational errors.

I also study hardware-aware parameter-efficient fine-tuning methods for large language models, such as the co-design of low-rank adaptation (LoRA) and hybrid CIM architectures. The objective is not merely to compress models, but to redesign the adaptation process so that limited compute, memory, and communication resources are allocated to the parameters and computational paths that matter most.

This direction reflects one of my core beliefs: **efficient AI should not rely solely on faster hardware; model representations and learning mechanisms must evolve together with hardware capabilities.**

---

### 3. Efficient Execution and Generation of Foundation Models

For long-context LLMs and generative foundation models, inference cost is often dominated by attention, KV-cache management, token-by-token decoding, and verification and synchronization procedures. Even when the underlying hardware provides high computational throughput, redundant retrieval, storage, memory access, and serial dependencies can severely limit end-to-end efficiency.

Starting from the execution mechanisms of models, I investigate how to eliminate these unnecessary costs. For example, in long-context attention, I explore reusing historical top-*k* retrieval results based on similarity, thereby avoiding repeated expensive search and ranking operations. For KV-cache management, I study how to identify and retain the contextual states that truly affect current generation, reducing both memory footprint and access overhead. In speculative decoding, I investigate approximate verification and context-aware cache pruning to mitigate the serial bottleneck of autoregressive generation.

I also study non-autoregressive or weakly autoregressive generation paradigms, including diffusion language models, and explore mechanisms such as parallel token commitment to improve the practical efficiency of few-step generation. Together, these efforts revisit a fundamental question:

> **Must foundation models operate in the traditional token-by-token, exact, and highly serial manner?**

By jointly rethinking algorithms and execution mechanisms, I aim to reduce unnecessary computation, storage, and synchronization during inference, enabling foundation models to serve real-world applications more efficiently and scalably.

---

### 4. Reliable AI under Approximate Computing and Imperfect Hardware

Efficient computing often entails approximation: low-bit representations introduce quantization errors; analog computing is affected by noise, drift, nonlinearity, and device-to-device variation; and aggressive cache pruning, approximate verification, or parallel generation may also alter a model’s original behavior. Efficiency and reliability are therefore not independent objectives, but system properties that must be designed and evaluated jointly.

I study how hardware nonidealities in analog CIM propagate through the internal computation of Transformers, mixture-of-experts (MoE) models, diffusion models, and LLMs. For example, analog noise can alter attention distributions; errors in the KV cache can accumulate progressively during generation; router bias in MoE models can direct tokens to unsuitable experts; and errors in diffusion models can disrupt guidance and iterative denoising processes.

To address these challenges, I develop cross-layer robustness techniques, including noise-aware training, stochastic sampling strategies, protection of critical KV-cache states, expert replacement, router calibration, and guidance recalibration for diffusion Transformers. Beyond conventional task accuracy and generation quality, I also focus on higher-level reliability questions: Can hardware nonidealities affect the reasoning behavior of language models, the stability of decision-making in complex tasks, or the trustworthiness of model outputs?

The goal of this research direction is to move AI systems beyond “achieving the best performance on ideal hardware” toward “maintaining stable, predictable, and trustworthy behavior under real hardware constraints.”

---

## Long-Term Vision

My long-term goal is to establish a **cross-layer co-design paradigm** for next-generation AI. In this paradigm, algorithms, model representations, execution mechanisms, system architectures, and device characteristics are no longer optimized in isolation. Instead, they are jointly designed around end-to-end efficiency, reliability, and scalability.

I aim to help drive the following transitions in AI systems:

- From focusing only on peak compute performance to optimizing end-to-end data movement, storage, communication, and generation overhead;
- From treating models as fixed workloads to enabling model representations, precision configurations, and adaptation mechanisms to actively match hardware capabilities;
- From pursuing accuracy only under ideal conditions to understanding and safeguarding model behavior under approximate computing and hardware nonidealities;
- From point-wise hardware acceleration to algorithm–model–system–device co-innovation for real-world AI workloads.

Ultimately, I aim to build AI systems that remain **efficient, stable, trustworthy, and scalable** under resource constraints, approximate computation, and imperfect hardware.
