# Why Ray Uses `num_cpus=0` + `_force_on_current_node()` for Logging Actors

These two configuration knobs are complementary, bound together to resolve a specialized compute placement requirement: running lightweight I/O-bound sidecar processes without competing with compute tasks, while simultaneously guaranteeing local execution physically adjacent to the driver.

**Core Invariant:** Sidecar actors (e.g., loggers) must fundamentally bypass admission control intended for logical compute tasks, yet mandate exact physical node locality. Using `num_cpus=1` starves legitimate compute; omitting node affinity uncontrollably scatters the sidecar across the cluster, permanently destroying localized driver I/O.

## 1. Admission Control Bypass via `num_cpus=0`

Ray's CPU resources heavily manipulate logical admission accounting rather than hard physical limitations (`resources.rst:23-28`). Specifically configuring `num_cpus=0` establishes the actor as entirely free of resource penalties—enforcing complete bypass across cluster scheduling locks (`wandb.py:705-720`). Tune’s internal system telemetry similarly deploys identical logic enabling `TunerInternal` configurations natively alongside underlying reporter queues seamlessly preventing compute starvation inherently (`tuner.py:130-132`, `progress_reporter.py:1557-1560`).

Because logging is entirely auxiliary to mainstream operations, requesting merely `1 CPU` instantly diminishes processing efficiency violently blocking subsequent model parallel pipelines. This decoupling of raw processing quotas from explicit physical task routing structurally mirrors fine-grained allocation principles seen in [Apache Mesos](http://mesos.apache.org/documentation/latest/architecture/) and Borg, isolating fundamental admission guarantees from secondary localized sidecars.

## 2. Re-pinning Placement via `_force_on_current_node()`

By purposefully bypassing cluster resource capacities, Ray trips an overlooked, implicit secondary scheduling condition explicitly designed exclusively for completely zero-resource configurations mapping instances aggressively into arbitrary random node dispersions broadly across the environment (`index.rst:70-71`). Central C++ evaluations validate this natively within `cluster_resource_scheduler.cc:159-164`.

W&B logging actors rely heavily upon local filesystem access matching directly alongside active network/driver pipelines. Random scheduling deployment violently breaks these non-negotiable IO links permanently. 

To counteract this random placement, Ray implements `_force_on_current_node()`—evaluating the driver’s localized identity immediately allocating strict placement restrictions directly. Underneath, this delegates processing straight into `NodeAffinitySchedulingStrategy(soft=False)` overrides (`node.py:48-69`, `node.py:35-45`).

The parameter `soft=False` configures strict limitations entirely overriding alternate protocols. When an affiliated execution node fundamentally degrades, the dependent actor permanently destructs rather than enduring silent reschedules blindly into corrupt states (`index.rst:97-104`). The underlying C++ documentation precisely states this policy explicitly trumps typical random dispersion mechanisms ("except for HARD node affinity scheduling policy").

## Summary

Individually, each setting fails to achieve the objective. `num_cpus=0` permits the logging operations safe separation allowing unconstrained computation flow without blocking primary worker resources. Evaluating constraints via `_force_on_current_node()` actively negates subsequent random dispersal mechanisms inherently, forcibly binding logging sidecars straight toward originating driver points—simultaneously addressing distinct Ray Client execution nodes accurately maintaining critical physical locality reliably alongside independent server operations equivalently (`node.py:53-56`).
