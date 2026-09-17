---
layout: archive
title: "Research Reading Guide"
permalink: /blogs/
author_profile: true
---

This page is a curated reading guide to efficient AI systems, large language
model (LLM) inference, quantization, compute-in-memory, and hardware-aware
machine learning.

The goal is not to provide an exhaustive bibliography. Instead, I collect a
small set of foundational and representative papers that help readers quickly
understand the central problems, key technical ideas, and major trade-offs in
each area.

Rather than reading these papers independently, I recommend following the
suggested order in each section. The papers are selected to form a connected
research story: a bottleneck motivates a method, the method exposes a new
limitation, and that limitation motivates the next generation of work.

## How to Use This Guide

For each paper, I briefly summarize:

- **Why read it?** — the role of the paper in the field;
- **Key idea** — its main technical contribution;
- **Takeaway** — what is worth remembering; and
- **Read next** — where it fits in the reading path.

---

# 1. Efficient LLM Inference and Serving

Modern LLM inference is constrained not only by arithmetic throughput, but also
by autoregressive decoding, memory bandwidth, KV-cache capacity, and request
scheduling. A useful starting point is to understand that serving an LLM is
simultaneously a model-execution problem, a memory-management problem, and a
systems-scheduling problem.

## Recommended Reading Path

1. Transformer architecture and autoregressive generation
2. Iteration-level scheduling for generation workloads
3. KV-cache memory management
4. Speculative decoding

---

## [Attention Is All You Need](https://arxiv.org/abs/1706.03762)

**Vaswani et al., NeurIPS 2017**

**Why read it?**  
This is the foundational paper for the Transformer architecture. Understanding
self-attention, multi-head attention, feed-forward layers, and autoregressive
decoding is necessary before studying LLM inference systems.

**Key idea:**  
The paper replaces recurrence and convolution with attention-based computation.
During training, tokens can be processed in parallel; during generation,
however, each new token depends on previously generated tokens.

**Takeaway:**  
The Transformer architecture scales well during training, but autoregressive
decoding creates a sequential inference process. This is the fundamental reason
why LLM serving requires specialized systems techniques.

**Read next:**  
[Orca: A Distributed Serving System for Transformer-Based Generative Models](https://www.usenix.org/conference/osdi22/presentation/yu)

---

## [Orca: A Distributed Serving System for Transformer-Based Generative Models](https://www.usenix.org/conference/osdi22/presentation/yu)

**Yu et al., OSDI 2022**

**Why read it?**  
Orca is a foundational systems paper for generative-model serving. It clearly
explains why conventional static batching is inefficient for autoregressive
workloads with variable output lengths.

**Key idea:**  
Orca introduces **iteration-level scheduling**. Instead of scheduling an entire
generation request as one job, the scheduler operates at the granularity of a
single decoding iteration. It can remove completed requests and insert new
requests between decoding steps.

The paper also introduces **selective batching**, which batches operations that
benefit from batching while avoiding unnecessary synchronization for others.

**Takeaway:**  
Batching is not a one-time decision made at request arrival. For LLM serving,
the batch should evolve dynamically as requests generate tokens and complete.

**Read next:**  
[Efficient Memory Management for Large Language Model Serving with PagedAttention](https://arxiv.org/abs/2309.06180)

---

## [Efficient Memory Management for Large Language Model Serving with PagedAttention](https://arxiv.org/abs/2309.06180)

**Kwon et al., SOSP 2023**

**Why read it?**  
This paper introduced PagedAttention and the vLLM serving system. It is one of
the most important references for understanding why KV-cache management is a
central problem in modern LLM inference.

**Key idea:**  
The key-value cache grows as generation continues and can consume a large
fraction of GPU memory. PagedAttention stores the KV cache in non-contiguous
blocks, inspired by virtual-memory paging. This reduces memory waste caused by
fragmentation and avoids large preallocated contiguous buffers.

The design also enables more flexible KV-cache sharing across requests, which is
useful for workloads involving shared prompts or branching decoding.

**Takeaway:**  
For long-context or high-throughput serving, efficient KV-cache management can
be as important as faster matrix multiplication.

**Read next:**  
[Fast Inference from Transformers via Speculative Decoding](https://arxiv.org/abs/2211.17192)

---

## [Fast Inference from Transformers via Speculative Decoding](https://arxiv.org/abs/2211.17192)

**Leviathan, Kalman, and Matias, ICML 2023**

**Why read it?**  
This paper is a central reference for speculative decoding, a technique that
reduces the sequential bottleneck of autoregressive generation without changing
the target model's output distribution.

**Key idea:**  
A smaller draft model proposes multiple candidate tokens. The larger target
model then verifies these candidates in parallel. Accepted tokens are emitted
directly, allowing multiple output tokens to be generated in approximately one
target-model verification step.

**Takeaway:**  
Speculative decoding shifts the question from "how fast is one decoding step?"
to "how many draft tokens can be accepted per target-model invocation?"

**Open question:**  
How should speculative decoding be co-designed with quantization, batching,
KV-cache policies, and heterogeneous accelerators?

---

# 2. Quantization for Large Language Models

Quantization reduces memory footprint, memory traffic, and potentially latency
by representing weights, activations, and intermediate states with fewer bits.
For LLMs, the major challenge is preserving quality despite activation outliers
and the large dynamic range of Transformer computations.

## Recommended Reading Path

1. Integer-only inference and quantization-aware training
2. Outlier-aware quantization for large Transformers
3. Second-order weight-only post-training quantization
4. Weight--activation quantization
5. Activation-aware weight quantization

---

## [Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference](https://openaccess.thecvf.com/content_cvpr_2018/html/Jacob_Quantization_and_Training_CVPR_2018_paper.html)

**Jacob et al., CVPR 2018**

**Why read it?**  
This paper is a useful starting point for practical neural-network quantization.
It presents quantization not merely as compression, but as a joint
algorithm--hardware deployment problem.

**Key idea:**  
The paper proposes an affine quantization scheme and quantization-aware
training procedure that enable inference using integer arithmetic. It shows how
weights, activations, biases, scaling factors, and accumulators must be handled
consistently for accurate low-precision execution.

**Takeaway:**  
Reducing bit width only matters when the representation maps efficiently onto
the arithmetic and memory hierarchy of the target hardware.

**Read next:**  
[LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale](https://arxiv.org/abs/2208.07339)

---

## [LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale](https://arxiv.org/abs/2208.07339)

**Dettmers et al., NeurIPS 2022**

**Why read it?**  
This work established that outlier features are a major obstacle to low-bit
quantization in large Transformers.

**Key idea:**  
Most matrix-multiplication values are quantized to INT8 using vector-wise
quantization. A small number of outlier dimensions are isolated and computed in
higher precision through mixed-precision decomposition.

**Takeaway:**  
Quantization error is highly non-uniform. A small number of extreme activation
features can dominate the error and determine whether low-precision inference
succeeds.

**Read next:**  
[GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers](https://arxiv.org/abs/2210.17323)

---

## [GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers](https://arxiv.org/abs/2210.17323)

**Frantar et al., ICLR 2023**

**Why read it?**  
GPTQ is a landmark paper for low-bit, weight-only post-training quantization of
autoregressive language models.

**Key idea:**  
GPTQ quantizes a model layer by layer using approximate second-order
information. When one group of weights is quantized, the method compensates for
the resulting error in the remaining full-precision weights.

This allows large generative models to be quantized to low-bit weight formats
while maintaining relatively strong accuracy.

**Takeaway:**  
Calibration data and curvature information can significantly improve
post-training quantization compared with naive rounding.

**Limitation:**  
Weight-only quantization reduces model-memory traffic, but it does not directly
solve activation quantization, KV-cache storage, or kernel-level efficiency.

**Read next:**  
[SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models](https://arxiv.org/abs/2211.10438)

---

## [SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models](https://arxiv.org/abs/2211.10438)

**Xiao et al., ICML 2023**

**Why read it?**  
SmoothQuant is a central reference for understanding weight--activation
quantization in LLMs.

**Key idea:**  
The method observes that weights are generally easier to quantize than
activations. It applies an equivalent channel-wise scaling transformation that
moves part of the quantization difficulty from activation channels to weight
channels.

This makes INT8 weight-and-activation quantization practical for major LLM
matrix multiplications.

**Takeaway:**  
The key question is not only how many bits are used, but where the dynamic range
resides and which tensors are most difficult for the target hardware to
represent accurately.

**Read next:**  
[AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration](https://arxiv.org/abs/2306.00978)

---

## [AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration](https://arxiv.org/abs/2306.00978)

**Lin et al., MLSys 2024**

**Why read it?**  
AWQ provides a complementary perspective to GPTQ. Instead of relying primarily
on second-order reconstruction, it uses activation statistics to identify
important weight channels.

**Key idea:**  
AWQ identifies salient channels using activation information and rescales them
to reduce quantization error. The method is designed for low-bit weight
quantization without requiring backpropagation.

**Takeaway:**  
Weight importance is not determined solely by weight magnitude. It also depends
on the activations that interact with those weights during inference.

**Open question:**  
Which combination of quantization granularity, calibration procedure, kernel
design, and target hardware gives the best end-to-end trade-off among accuracy,
latency, throughput, energy, and memory footprint?

---

# 3. Compute-in-Memory and AI Hardware

Compute-in-memory (CIM) reduces data movement by performing computation inside
or near memory arrays. It is particularly promising for matrix-vector
multiplication, but practical CIM systems must account for limited precision,
device variation, analog noise, data-converter overhead, peripheral circuitry,
and mapping constraints.

## Recommended Reading Path

1. Analog crossbar computation
2. Architecture-level CIM accelerators
3. Processing-in-memory using main-memory arrays
4. Mixed-precision methods for non-ideal devices
5. Hardware-aware training and robustness

---

## [ISAAC: A Convolutional Neural Network Accelerator with In-Situ Analog Arithmetic in Crossbars](https://arxiv.org/abs/1606.08152)

**Shafiee et al., ISCA 2016**

**Why read it?**  
ISAAC is one of the representative early papers on analog resistive-crossbar
accelerators for deep neural networks.

**Key idea:**  
The architecture maps neural-network computations onto resistive crossbar
arrays, where analog physical behavior performs multiply--accumulate operations
in situ. The full system also includes digital logic, buffers, interconnects,
and analog-to-digital conversion.

**Takeaway:**  
The crossbar array alone does not determine accelerator efficiency. ADCs, DACs,
buffers, communication, and non-MAC operations can dominate area, energy, or
latency.

**Read next:**  
[PRIME: A Novel Processing-in-Memory Architecture for Neural Network Computation in ReRAM-Based Main Memory](https://people.ece.cornell.edu/land/courses/ece5745/2018sp/papers/prime-isca16.pdf)

---

## [PRIME: A Novel Processing-in-Memory Architecture for Neural Network Computation in ReRAM-Based Main Memory](https://people.ece.cornell.edu/land/courses/ece5745/2018sp/papers/prime-isca16.pdf)

**Chi et al., ISCA 2016**

**Why read it?**  
PRIME demonstrates a different CIM design point from ISAAC: using ReRAM-based
main memory as a reconfigurable substrate for both storage and neural-network
computation.

**Key idea:**  
Parts of the ReRAM memory array can be configured either as conventional memory
or as compute resources. PRIME combines this substrate with architecture and
software support for mapping neural-network layers onto memory arrays.

**Takeaway:**  
Compute-in-memory is not a single fixed architecture. Important design choices
include where computation sits in the memory hierarchy, how arrays are exposed
to software, and how much memory capacity is reserved for computation.

**Read next:**  
[Mixed-Precision Deep Learning Based on Computational Memory](https://arxiv.org/abs/2001.11773)

---

## [Mixed-Precision Deep Learning Based on Computational Memory](https://arxiv.org/abs/2001.11773)

**Nandakumar et al., Nature Electronics 2020**

**Why read it?**  
This paper is important because it connects idealized analog memory computation
with realistic device non-idealities.

**Key idea:**  
The method uses computational memory for in-place weighted summation and
low-precision updates, while a digital unit accumulates and controls updates in
higher precision. This mixed-precision design can tolerate imperfect analog
conductance changes and other non-ideal device behaviors.

**Takeaway:**  
Practical analog acceleration does not require every operation to be perfectly
analog. A system can divide responsibility between high-efficiency but imperfect
memory computation and high-precision digital control.

**Open question:**  
For Transformer and LLM workloads, what is the best division of labor among
analog matrix multiplication, digital attention operations, normalization,
calibration, and error correction?

---

# 4. Hardware-Aware Machine Learning

Hardware-aware machine learning studies how models should be trained,
quantized, compressed, or adapted when hardware has limited precision, noise,
device variation, constrained dynamic range, or expensive data movement.

The central principle is that floating-point software accuracy is not the final
objective. A useful model must perform well under the actual properties of its
target hardware.

## Recommended Reading Path

1. Quantization-aware training for integer inference
2. Low-precision floating-point training
3. Mixed-precision training with computational memory
4. Training with realistic analog hardware models

---

## [Training Deep Neural Networks with 8-bit Floating Point Numbers](https://arxiv.org/abs/1812.08011)

**Wang et al., ICLR 2019**

**Why read it?**  
This paper is useful for understanding why low-precision training is more
difficult than low-precision inference.

**Key idea:**  
The authors demonstrate training using 8-bit floating-point representations.
They combine techniques such as chunk-based accumulation and stochastic
rounding to manage numerical error during forward propagation, backpropagation,
and parameter updates.

**Takeaway:**  
Precision should not necessarily be treated as one global choice. Different
tensors and operations have different numerical sensitivities.

**Read next:**  
[Mixed-Precision Deep Learning Based on Computational Memory](https://arxiv.org/abs/2001.11773)

---

## [Hardware-Aware Training for Large-Scale and Diverse Deep Learning Inference Workloads Using In-Memory Computing-Based Accelerators](https://arxiv.org/abs/2302.08469)

**Rasch et al., Nature Communications 2023**

**Why read it?**  
This work is a strong reference for realistic hardware-aware training in analog
in-memory computing. It studies CNNs, recurrent models, and Transformer-based
workloads under a detailed model of analog hardware behavior.

**Key idea:**  
The authors retrain models while incorporating realistic analog crossbar
non-idealities, including noise and device-related errors. The work examines
which sources of error are most harmful and shows that hardware-aware training
can often recover much of the accuracy lost under naive deployment.

**Takeaway:**  
Robust deployment depends on modeling the right hardware errors. Optimizing only
for weight noise may be insufficient if the dominant practical limitations come
from input/output noise, data conversion, conductance drift, or nonlinear
device behavior.

**Open question:**  
Can model architecture, quantization, training objectives, crossbar mapping,
ADC/DAC precision, and runtime calibration be optimized jointly rather than as
separate stages?

---

# Cross-Cutting Questions

The papers above suggest several research questions that I find especially
interesting:

- How should LLM inference efficiency be measured beyond tokens per second?
- When is an inference workload memory-bandwidth-bound, compute-bound,
  KV-cache-bound, or scheduler-bound?
- Which quantization methods produce actual end-to-end speedups on specific
  hardware platforms, rather than only reducing model size?
- How should KV-cache representation, model quantization, and serving policies
  be co-designed for long-context generation?
- When does analog compute-in-memory remain advantageous after accounting for
  ADC/DAC cost, calibration, peripheral circuitry, and device non-idealities?
- Can hardware-aware training produce models that generalize across chips and
  operating conditions rather than overfitting to one hardware model?
- What is the right abstraction layer for algorithm--hardware co-design in
  future LLM and generative-AI systems?

# Scope

This is a selective and opinionated reading guide rather than a comprehensive
survey. The paper selection prioritizes work that establishes a useful
technical idea, exposes an important systems or hardware limitation, or helps
connect algorithms to real deployment constraints.
