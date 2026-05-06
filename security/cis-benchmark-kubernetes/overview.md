# Kubernetes Versionen & CIS Benchmark - Überblick

## Aktuelle Kubernetes Versionen (Stand: 2026)

| Version | Release | Support Ende |
|---------|---------|--------------|
| 1.36 | April 2026 | ~Juni 2027 |
| 1.35 | Dezember 2025 | ~Februar 2027 |
| 1.34 | August 2025 | ~Oktober 2026 |

Empfehlung: immer auf einer der drei aktuellen Minor-Versionen bleiben.  
Release-Zyklus: ca. alle 4 Monate eine neue Minor-Version.

> Der CIS Kubernetes Benchmark V1.12.0 gilt für Kubernetes **v1.32 - v1.34**.

## CIS Benchmark für Kubernetes

Das Center for Internet Security (CIS) veröffentlicht regelmäßig Benchmarks für Kubernetes.

**Aktuelle Version: CIS Kubernetes Benchmark V1.12.0**

Der Benchmark deckt folgende Bereiche ab:
- **Section 1** — Control Plane Components (API Server, Controller Manager, Scheduler, etcd)
- **Section 2** — etcd
- **Section 3** — Control Plane Configuration
- **Section 4** — Worker Nodes (kubelet, kube-proxy)
- **Section 5** — Kubernetes Policies (RBAC, Network Policies, Pod Security)

## CIS Benchmark PDF

Der Trainer stellt das Dokument unter folgendem Link bereit:

**[CIS Kubernetes Benchmark V1.12.0 (PDF)](http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf)**

## Tool: kube-bench

Für das automatisierte Scannen des Clusters gegen den CIS Benchmark verwenden wir **kube-bench** von Aqua Security:

```
https://github.com/aquasecurity/kube-bench
```

kube-bench führt die CIS Benchmark Checks automatisch durch und gibt eine Übersicht über PASS/FAIL/WARN Ergebnisse.

## Referenzen

  * https://www.cisecurity.org/benchmark/kubernetes
  * https://github.com/aquasecurity/kube-bench
  * https://kubernetes.io/releases/
