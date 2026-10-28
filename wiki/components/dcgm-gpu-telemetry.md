# DCGM GPU Telemetry

NVIDIA's Data Center GPU Manager (DCGM) is the standard source of GPU health and utilisation metrics. Its `dcgm-exporter` publishes them to Prometheus for Kubernetes monitoring and autoscaling. It touches exam objectives 4.3 and 4.4, and section 8's monitoring objectives.

DCGM metrics are the GPU-level input to [LLM Inference Profiling](../concepts/llm-inference-profiling.md), and to the monitoring and autoscaling set-up in [Kubernetes GPU Inference Deployment](../concepts/kubernetes-gpu-inference-deployment.md). Above the GPU, agent-level telemetry is covered in [Agent Observability and Tracing](../concepts/agent-observability-tracing.md).

## What DCGM measures

DCGM's documentation describes three kinds of capability **[sourced]**:
- **Profiling fields:** SM activity (field 1002), SM occupancy (1003), tensor-core activity (1004), DRAM activity (1005), and NVLink and PCIe throughput.
- **Health watches.**
- **Diagnostics** at levels 1 to 3, which run from seconds to about 15 minutes.

Profiling metrics are sampled at 1 Hz by default, with a 100 ms minimum. DCGM's profiling conflicts with Nsight Systems, so pause it with `dcgmi profile --pause` and `--resume` during a deep trace. Health policies can trigger actions such as a GPU reset on XID, ECC or NVLink errors **[sourced]**. NVIDIA's Kubernetes glossary lists DCGM as part of its GPU container tooling, and the NIM Operator can autoscale on DCGM or NIM metrics **[sourced]**.

## Getting metrics into Prometheus

NVIDIA's GPU-telemetry guide sets this up in four steps **[sourced]**:
1. Install `kube-prometheus-stack` with Helm into a `prometheus` namespace. Set `serviceMonitorSelectorNilUsesHelmValues=false` so Prometheus discovers ServiceMonitors that its own release did not create, and add a `gpu-metrics` scrape job with a 1-second interval against the `gpu-operator` namespace.
2. Install `dcgm-exporter` from NVIDIA's `gpu-helm-charts` repository. Choose metrics through `arguments`, `extraConfigMapVolumes` and `extraEnv`. The exporter listens on port 9400.
3. Verify with the query `DCGM_FI_DEV_GPU_UTIL`.
4. Import Grafana dashboard 12239.

The guide uses NodePort and default credentials, which is fine for a demo but not hardened for production **[inferred]**.

## Interview Q&A

**How do GPU metrics reach Grafana?** dcgm-exporter exposes them on port 9400, Prometheus scrapes them through a ServiceMonitor or scrape job, and Grafana shows them on dashboard 12239 **[sourced]**.

**Why set `serviceMonitorSelectorNilUsesHelmValues=false`?** So Prometheus selects ServiceMonitors that its own Helm release did not create **[sourced, inferred]**.

**Nsight Systems versus DCGM?** Nsight is a deep, intrusive trace of one run, used for root-cause analysis. DCGM is continuous, low-overhead fleet telemetry. Their profiling counters conflict, so pause DCGM while tracing **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Setting up Prometheus for NVIDIA GPU telemetry](https://docs.nvidia.com/datacenter/cloud-native/gpu-telemetry/latest/kube-prometheus.html) |
| 2026-10-28 | [DCGM feature overview](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html) |
| 2026-10-28 | [Kubernetes glossary (NVIDIA)](https://www.nvidia.com/en-us/glossary/kubernetes/) |
