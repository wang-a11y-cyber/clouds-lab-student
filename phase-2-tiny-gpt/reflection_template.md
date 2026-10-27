# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->248.161<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.071<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->433048.3<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->12854.3<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.5019<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2499.007 ms, 2 threads: 2202.611 ms, 4 threads: 2329.92 ms, 8 threads: 2380.158 ms. The fastest configuration was 2 threads.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive implementation spends much of its time executing nested loops and individual operations through the Python interpreter. The vectorized implementation expresses the same arithmetic as array and matrix operations, which allowing NumPy and optimized OpenBLAS routines to perform the work in compiled code and exploit SIMD instructions, such as AVX. This reduced the attention time from 248.161 ms to 1.071 ms in my experiment.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The best result was at 2 threads (2202.611 ms), compared with 2499.007 ms for one thread. Increasing the count to 4 and 8 threads made the step slower, which are at 2329.92 ms and 2380.158 ms respectively. The Colab runtime has limited CPU resources, thus adding software threads does not necessarily add physical execution capacity. Extra threads must also coordinate and compete for such as CPU time, caches, memory bandwidth, and shared data, thus their overhead eventually exceeds the benefit of additional parallelism.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training consists largely of highly parallel matrix multiplications and other tensor operations. A GPU can thus execute many such operations concurrently across a large number of processing units, which suits this workload very well. In my run, 100 steps fell from 433048.3 ms on the CPU to 12854.3 ms on the T4. However, in one case that a small sequential workload with little independent computation will not obtain the same GPU speed-up.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
The experiments show that parallelism stops helping for different reasons at different layers. Vectorization removed Python-loop overhead and exposed data-parallel arithmetic to optimized machine code, where attention fell from 248.161 ms to 1.071 ms, thus the naive implementation was primarily limited by how the work was expressed and executed. CPU threading then encountered a hardware and coordination limit. Two threads were fastest at 2202.611 ms, while four threads increased the time to 2329.92 ms. Because additional threads had to compete for such as limited CPU execution resources, caches and memory bandwidth, while paying synchronization and scheduling costs. The GPU exposed much more parallel hardware for the transformer's matrix operations computation, reducing 100 training steps from 433048.3 ms on CPU to 12854.3 ms. However, the GPU acceleration is also bounded by such as sequential work, available GPU resources, synchronization, and the cost of moving data between host and device. Parallelism therefore helps only while the useful concurrent work outweighs these overheads and resource limits.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I will repeat the threading sweep on machines with different numbers of CPU cores, while keeping the model and workload fixed. If the machines with more cores continued improving beyond the 2-thread minimum observed here, it would support the conclusion that it was primarily caused by limited CPU hardware, rather than an inherent limit of the workload.
<!--/answer-->
