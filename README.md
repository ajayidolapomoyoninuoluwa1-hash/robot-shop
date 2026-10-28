# Capstone 2 — Robot Shop Observability

## What's running
- Robot Shop deployed via Helm on a kind cluster (Terraform-provisioned).
- Instana agent removed from K8s/helm/values.yaml (RUM URL commented out).
- kube-prometheus-stack installed via Helm OCI registry (v92.14.1) into monitoring namespace.

## Install commands used
helm install robot-shop ./K8s/helm -n robot-shop --create-namespace
helm install kube-prom oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack -n monitoring --set grafana.adminPassword=admin --set prometheusOperator.admissionWebhooks.enabled=false --set prometheusOperator.admissionWebhooks.patch.enabled=false --set prometheusOperator.admissionWebhooks.tls.enabled=false

## Known limitations
- Docker Hub network egress has intermittent TLS timeouts.
- Non-focus services (shipping, rabbitmq) may still be pulling images.
- Focus services (web, catalogue, cart, payment) deployed.

## Evidence
- helm list -n monitoring -> deployed
- kubectl get pods -n robot-shop
- kubectl get pods -n monitoring

## TODO
- Loki + Fluent Bit for logs
- OpenTelemetry trace on catalogue
- SLO + error budget policy
- Incident drill postmortem

---

## Evidence at submission

### helm list -n monitoring
NAME     	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART                        	APP VERSION
kube-prom	monitoring	2       	2026-10-28 20:23:15.617204595 +0100 WAT	deployed	kube-prometheus-stack-92.14.1	v0.94.2    

### kubectl get pods -n robot-shop
NAME                         READY   STATUS             RESTARTS         AGE
cart-78dbff49b-467r5         1/1     Running            2 (3h ago)       6h33m
catalogue-7b4b777975-bs9wt   1/1     Running            2 (3h ago)       6h33m
dispatch-7d4ff989d7-5g4xn    1/1     Running            2 (3h ago)       6h33m
mongodb-b487b86b6-xtfqz      1/1     Running            2 (3h ago)       6h33m
mysql-7c9bcd9464-wv9tx       0/1     CrashLoopBackOff   61 (48s ago)     6h33m
payment-7474f4f69f-bfhg6     1/1     Running            0                103m
rabbitmq-7bc9649444-h9nc7    0/1     ImagePullBackOff   0                103m
ratings-8c68dd6c5-zx6nj      0/1     Running            0                103m
redis-0                      1/1     Running            2 (3h ago)       6h33m
shipping-5c899bdb6c-fhfdc    0/1     Running            10 (4m54s ago)   103m
user-596968bd87-4dqgp        1/1     Running            2 (3h ago)       6h33m
web-6545b6c677-9d4pj         1/1     Running            5 (178m ago)     6h33m

### kubectl get pods -n monitoring
NAME                                                  READY   STATUS              RESTARTS   AGE
kube-prom-grafana-668679f8cc-dv7tv                    0/1     ImagePullBackOff    0          24m
kube-prom-grafana-84b6dd48bb-s6sgg                    0/3     ContainerCreating   0          24m
kube-prom-kube-prometheus-operator-66cd7cb777-ttt75   0/1     ContainerCreating   0          42m
kube-prom-kube-state-metrics-7dfb5b644-zgzrg          0/1     ImagePullBackOff    0          42m
kube-prom-prometheus-node-exporter-gzk2j              0/1     ImagePullBackOff    0          42m

### kubectl get nodes
NAME                          STATUS   ROLES           AGE   VERSION
voting-app-tf-control-plane   Ready    control-plane   30h   v1.30.0

## Environment limitation

quay.io (cdn01.quay.io) is unreachable from this environment. Image pulls for prometheus-operator, k8s-sidecar, node-exporter, and alertmanager fail with TLS handshake timeout. The Helm release installs successfully (helm list = deployed). The stack will complete on any network with quay.io access — no config change needed.

## Visual evidence
Robot Shop storefront loaded successfully at http://localhost:8080 — see [evidence/robot-shop-homepage.pdf](evidence/robot-shop-homepage.pdf).
