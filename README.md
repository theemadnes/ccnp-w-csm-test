# ccnp-w-csm-test
just checking to see how CC netPols interact with CSM sidecars

using `gke_e2m-private-test-01_us-east4_edge-to-mesh-02` to host "remote services", so just converting `whereami-frontend` and `whereami-backend` to `LoadBalancer` services to expose a public IP. 

### friction log

```
kubectl --context=gke_e2m-private-test-01_us-east4_edge-to-mesh-02 -n backend edit svc whereami-backend
kubectl --context=gke_e2m-private-test-01_us-east4_edge-to-mesh-02 -n frontend edit svc whereami-frontend

# because target cluster is using STRICT peerAuth, need to set to PERMISSIVE
kubectl --context=gke_e2m-private-test-01_us-east4_edge-to-mesh-02 -n istio-system delete peerAuthentication default

# hit "backend"
$ curl 35.188.234.135
{
  "cluster_name": "edge-to-mesh-02",
  "gce_instance_id": "6874691623925427331",
  "gce_service_account": "whereami-tracer@e2m-private-test-01.iam.gserviceaccount.com",
  "host_header": "35.188.234.135",
  "metadata": "backend-v1",
  "node_name": "gk3-edge-to-mesh-02-nap-1175r8aa-3d8f21cd-1cab",
  "pod_ip": "10.124.0.145",
  "pod_name": "whereami-backend-bc9564cdd-qzrs2",
  "pod_name_emoji": "\ud83d\udc4c\ud83c\udffb",
  "pod_namespace": "backend",
  "pod_service_account": "whereami-backend",
  "project_id": "e2m-private-test-01",
  "timestamp": "2024-08-20T01:55:00",
  "zone": "us-east4-c"
}

# hit "frontend"
$ curl 34.150.153.228
{
  "backend_result": {
    "cluster_name": "edge-to-mesh-02",
    "gce_instance_id": "8387195349689851883",
    "gce_service_account": "whereami-tracer@e2m-private-test-01.iam.gserviceaccount.com",
    "host_header": "whereami-backend.backend.svc.cluster.local",
    "metadata": "backend-v1",
    "node_name": "gk3-edge-to-mesh-02-pool-3-9d6da422-plr5",
    "pod_ip": "10.124.0.198",
    "pod_name": "whereami-backend-bc9564cdd-l8q5c",
    "pod_name_emoji": "\ud83d\udc78\ud83c\udffe",
    "pod_namespace": "backend",
    "pod_service_account": "whereami-backend",
    "project_id": "e2m-private-test-01",
    "timestamp": "2024-08-20T01:55:54",
    "zone": "us-east4-b"
  },
  "cluster_name": "edge-to-mesh-02",
  "gce_instance_id": "8387195349689851883",
  "gce_service_account": "whereami-tracer@e2m-private-test-01.iam.gserviceaccount.com",
  "host_header": "34.150.153.228",
  "metadata": "frontend",
  "node_name": "gk3-edge-to-mesh-02-pool-3-9d6da422-plr5",
  "pod_ip": "10.124.0.200",
  "pod_name": "whereami-frontend-57b9f646bc-bfpjr",
  "pod_name_emoji": "\ud83d\udea3\ud83c\udffe",
  "pod_namespace": "frontend",
  "pod_service_account": "whereami-frontend",
  "project_id": "e2m-private-test-01",
  "timestamp": "2024-08-20T01:55:54",
  "zone": "us-east4-b"
}
# looks good - we can proceed

# exec into a pod on gke_e2m-private-test-01_us-central1_edge-to-mesh-01 to verify we can hit the LB VIPs
kubectl --context gke_e2m-private-test-01_us-central1_edge-to-mesh-01 -n backend exec --stdin --tty whereami-backend-6cc6f6c447-rhn67  -- /bin/sh
$ curl 34.150.153.228
{
  "backend_result": {
    "cluster_name": "edge-to-mesh-01",
    "gce_instance_id": "6065108078377000659",
    "gce_service_account": "whereami-tracer@e2m-private-test-01.iam.gserviceaccount.com",
    "host_header": "whereami-backend.backend.svc.cluster.local",
    "metadata": "backend-v1",
    "node_name": "gk3-edge-to-mesh-01-nap-khetyp5a-a613b0bb-vqm5",
    "pod_ip": "10.54.0.199",
    "pod_name": "whereami-backend-6cc6f6c447-rhn67",
    "pod_name_emoji": "\u2b07",
    "pod_namespace": "backend",
    "pod_service_account": "whereami-backend",
    "project_id": "e2m-private-test-01",
    "timestamp": "2024-08-20T02:01:01",
    "zone": "us-central1-a"
  },
  "cluster_name": "edge-to-mesh-02",
  "gce_instance_id": "5351824855028355419",
  "gce_service_account": "whereami-tracer@e2m-private-test-01.iam.gserviceaccount.com",
  "host_header": "34.150.153.228",
  "metadata": "frontend",
  "node_name": "gk3-edge-to-mesh-02-pool-3-d93fc1a1-f4ep",
  "pod_ip": "10.124.0.73",
  "pod_name": "whereami-frontend-57b9f646bc-dp2s6",
  "pod_name_emoji": "\ud83c\udfc3\ud83c\udffc\u200d\u2640\ufe0f",
  "pod_namespace": "frontend",
  "pod_service_account": "whereami-frontend",
  "project_id": "e2m-private-test-01",
  "timestamp": "2024-08-20T02:01:01",
  "zone": "us-east4-c"
}

$ curl 35.188.234.135
{
  "cluster_name": "edge-to-mesh-02",
  "gce_instance_id": "5351824855028355419",
  "gce_service_account": "whereami-tracer@e2m-private-test-01.iam.gserviceaccount.com",
  "host_header": "35.188.234.135",
  "metadata": "backend-v1",
  "node_name": "gk3-edge-to-mesh-02-pool-3-d93fc1a1-f4ep",
  "pod_ip": "10.124.0.69",
  "pod_name": "whereami-backend-bc9564cdd-jplq2",
  "pod_name_emoji": "\ud83e\uddd1\ud83c\udffc\u200d\ud83c\udfa4",
  "pod_namespace": "backend",
  "pod_service_account": "whereami-backend",
  "project_id": "e2m-private-test-01",
  "timestamp": "2024-08-20T02:01:32",
  "zone": "us-east4-c"
}
# looks good! 

# now attempt CCNP 
# first, enable CCNP on existing client cluster - this will cause downtime
gcloud container clusters update edge-to-mesh-01 \
    --location us-central1 \
    --enable-cilium-clusterwide-network-policy --project $PROJECT

kubectl --context gke_e2m-private-test-01_us-central1_edge-to-mesh-01 rollout restart ds -n kube-system anetd && \
    kubectl --context gke_e2m-private-test-01_us-central1_edge-to-mesh-01 rollout status ds -n kube-system anetd

kubectl --context gke_e2m-private-test-01_us-central1_edge-to-mesh-01 get crds ciliumclusterwidenetworkpolicies.cilium.io
```