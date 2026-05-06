# Clusterumgebung & Zugänge

## Überblick

Jeder Teilnehmer hat ein eigenes Kubernetes-Cluster auf DigitalOcean:

| Komponente | Beschreibung |
|------------|-------------|
| Control Plane | 1 Node (`k8s-tln{N}-cp`) |
| Worker | 1 Node (`k8s-tln{N}-w1`) |
| Bastion-Server | `161.35.210.204` |

## Login auf den Bastion-Server

```bash
ssh tln1@161.35.210.204
# Passwort: 11essen22
```

Jeder Teilnehmer nutzt seinen eigenen User (`tln1` bis `tln13`).

## SSH-Zugang zu den eigenen Cluster-Nodes

Nach dem Login auf dem Bastion-Server stehen folgende Shortcuts zur Verfügung:

```bash
# Zum Control Plane Node wechseln
ssh cp

# Zum Worker Node wechseln
ssh worker
```

Die SSH-Konfiguration liegt unter `~/.ssh/config` und ist bereits vorkonfiguriert.

## kubectl

kubectl ist auf dem Bastion-Server direkt nutzbar — die kubeconfig zeigt
automatisch auf das eigene Cluster:

```bash
# Nodes anzeigen
kubectl get nodes

# Ausgabe (Beispiel tln1):
# NAME          STATUS   ROLES           AGE   VERSION
# k8s-tln1-cp   Ready    control-plane   5m    v1.35.4
# k8s-tln1-w1   Ready    <none>          4m    v1.35.4
```

## Übersicht aller Zugänge

```
[Laptop]
    │
    │  ssh tln1@161.35.210.204
    ▼
[Bastion: client-bw]  ──── kubectl ────► [k8s-tln1-cp]
    │                                          │
    │  ssh cp                                  │ (Control Plane)
    ▼                                          │
[k8s-tln1-cp]                                 │
    │                                          ▼
    │  (vom Bastion)                    [k8s-tln1-w1]
    │  ssh worker                       (Worker Node)
    ▼
[k8s-tln1-w1]
```
