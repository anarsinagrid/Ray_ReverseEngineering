# Project 4 Reverse Engineering Report: Ray

- **Project Name:** Ray
- **Repository:** [ray-project/ray](https://github.com/ray-project/ray)
- **Project Category:** ML Frameworks & Libraries
- **Deadline:** April 3, 2026

## 1. Project Overview and Key Components

Ray is a unified framework for scaling AI and Python applications from a laptop to a cluster without requiring deep distributed systems expertise.

### Ray Framework Layers

Ray consists of three main layers:

- **Ray AI Libraries:** Domain-specific libraries for ML applications
- **Ray Core:** General-purpose distributed computing library
- **Ray Clusters:** Worker nodes connected to a head node

### Ray Core Primitives

Ray Core provides essential primitives for building distributed applications:

- **Tasks:** Stateless functions executed asynchronously on separate worker processes
- **Actors:** Stateful workers that can maintain and mutate state
- **Objects:** Immutable values stored in Ray's distributed object store and accessible across the cluster
- **Placement Groups:** Atomic reservation of resources across multiple nodes for gang scheduling

### Ray AI Libraries

The AI libraries provide scalable tools for common ML tasks:

- **Data:** Scalable datasets for ML preprocessing
- **Train:** Distributed model training
- **Tune:** Scalable hyperparameter tuning
- **RLlib:** Scalable reinforcement learning
- **Serve:** Scalable and programmable model serving

### Ray Cluster Architecture

A Ray cluster consists of:

- **Head Node:** Runs management processes such as the autoscaler and GCS
- **Worker Nodes:** Execute user code in tasks and actors
- **Autoscaler:** Dynamically scales cluster size based on resource demands

### Notes

Ray can run on any machine, cluster, cloud provider, or Kubernetes, and integrates with the broader ML ecosystem. The framework automatically handles orchestration, scheduling, fault tolerance, and auto-scaling for distributed applications.

## 2. Deep Reasoning Questions & Analysis

Question-wise analysis:

1. [Q1: Why the Task / Actor Split Is Fundamental](./Answers/Q_01.md)
2. [Q2: Why Ray Injects Tracing Exactly Once in `RemoteFunction`](./Answers/Q_02.md)
3. [Q3: Why Ray Uses a Separate `_WandbLoggingActor` Per Trial](./Answers/Q_03.md)
4. [Q4: Why Ray Rejects `async def` Remote Functions](./Answers/Q_04.md)
5. [Q5: Why Ray Uses `max_calls=1` for GPU Tasks by Default](./Answers/Q_05.md)
6. [Q6: Why Ray Enforces `actor.method.remote()` Instead of `actor.method()`](./Answers/Q_06.md)
7. [Q7: Why Ray Uses Different Argument Formatting for Cross-Language Functions](./Answers/Q_07.md)
8. [Q8: Why Ray Uses Implicit Session Context Capture (`get_session()`)](./Answers/Q_08.md)
9. [Q9: Why Ray Uses `num_cpus=0` + `_force_on_current_node()` for Logging Actors](./Answers/Q_09.md)
10. [Q10: Why Ray Uses `_last_export_cluster_and_job` for Deduplication](./Answers/Q_10.md)

## 3. Findings and Conclusion

This reverse engineering exercise shows that Ray's design is built around explicit execution boundaries: stateless versus stateful work, local versus remote invocation, and Python-native versus cross-language interoperability. Across the questions, Ray consistently favors correctness, fault tolerance, and predictable distributed behavior over convenience.

In summary, Ray's architecture is not just a collection of APIs but a set of carefully enforced invariants. Those invariants make it possible to scale distributed applications reliably while keeping the programming model practical for ML and systems workloads.

---
