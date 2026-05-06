# Veraltete Images erkennen mit version-checker

## Hintergrund

**version-checker** (jetstack, Apache 2.0) laeuft als Pod im Cluster, beobachtet
alle laufenden Container und vergleicht deren Image-Versionen mit der Registry.
Das Ergebnis kommt als Prometheus-Metrik - kein Prometheus noetig, ein einfaches
`curl` reicht.

---

## Schritt 1: version-checker deployen

```
kubectl apply -k https://github.com/jetstack/version-checker/deploy/yaml
```

Standardmaessig prueft version-checker nur Pods mit einer bestimmten Annotation.
Mit `--test-all-containers` prueft er den gesamten Cluster ohne Anpassung:

```
kubectl -n version-checker patch deployment version-checker \
  --type=json \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args","value":["--test-all-containers"]}]'
```

Warten bis der Pod laeuft:

```
kubectl -n version-checker rollout status deployment/version-checker
```

---

## Schritt 2: Ergebnisse lesen

Port-Forward auf den Metrics-Endpunkt:

```
kubectl -n version-checker port-forward svc/version-checker 8080:8080 &
```

Alle veralteten Images auf einen Blick (`value = 0` bedeutet: nicht aktuell):

```
curl -s http://localhost:8080/metrics \
  | grep 'version_checker_is_latest_version{' \
  | grep ' 0$'
```

Alle Images inkl. aktueller (`value = 1`):

```
curl -s http://localhost:8080/metrics \
  | grep 'version_checker_is_latest_version{'
```

**Beispielausgabe:**
```
version_checker_is_latest_version{container="nginx",current_version="1.24",image="docker.io/library/nginx",latest_version="1.27",namespace="default",pod="nginx-xxx"} 0
version_checker_is_latest_version{container="coredns",current_version="v1.11.1",image="registry.k8s.io/coredns/coredns",latest_version="v1.11.1",namespace="kube-system",pod="coredns-xxx"} 1
```

`current_version` ist was laeuft, `latest_version` ist was verfuegbar waere.

---

## Schritt 3: Fehler beim Abrufen anzeigen

Wenn version-checker ein Image nicht abfragen kann (Rate Limit, Auth, nicht erreichbar):

```
curl -s http://localhost:8080/metrics \
  | grep 'version_checker_image_failures_total'
```

Docker Hub hat ein Rate Limit fuer anonyme Abfragen. Im Workshop-Betrieb
(viele Teilnehmer, viele Images) koennen Fehler auftreten. Loesung: Docker
Hub Credentials als Secret hinterlegen:

```
kubectl -n version-checker create secret generic registry-creds \
  --from-literal=username=<dein-dockerhub-user> \
  --from-literal=password=<dein-dockerhub-token>
```

```
kubectl -n version-checker patch deployment version-checker \
  --type=json \
  -p='[
    {"op":"add","path":"/spec/template/spec/containers/0/env","value":[
      {"name":"VERSION_CHECKER_DOCKER_USERNAME","valueFrom":{"secretKeyRef":{"name":"registry-creds","key":"username"}}},
      {"name":"VERSION_CHECKER_DOCKER_PASSWORD","valueFrom":{"secretKeyRef":{"name":"registry-creds","key":"password"}}}
    ]}
  ]'
```

---

## Private Registry

version-checker unterstuetzt beliebige private Registries ueber Umgebungsvariablen.
Der `<NAME>`-Suffix erlaubt mehrere Registries gleichzeitig:

```
VERSION_CHECKER_SELFHOSTED_HOST_INTERN=registry.intern:5000
VERSION_CHECKER_SELFHOSTED_USERNAME_INTERN=nutzer
VERSION_CHECKER_SELFHOSTED_PASSWORD_INTERN=passwort
```

Danach prueft version-checker Images von `registry.intern:5000/...` genauso
wie Docker-Hub-Images - `latest_version` kommt dann aus der internen Registry.

---

## Aufraeumen

```
kill %1
kubectl delete -k https://github.com/jetstack/version-checker/deploy/yaml
```

---

## Zusammenfassung

| Was | Befehl |
|-----|--------|
| Alle veralteten Images | `curl .../metrics \| grep 'is_latest' \| grep ' 0$'` |
| Alle Images mit Status | `curl .../metrics \| grep 'is_latest'` |
| Abfragefehler | `curl .../metrics \| grep 'failures_total'` |
| Docker Hub Credentials | Secret + ENV im Deployment |
| Private Registry | `VERSION_CHECKER_SELFHOSTED_*` ENV-Variablen |
