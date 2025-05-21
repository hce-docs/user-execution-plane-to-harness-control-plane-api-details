# Harness Chaos Engineering API Documentation

## Overview

This document comprehensively details the APIs used for Chaos Engineering operations in Harness. The source for these API requests are the Harness components installed in the User cluster(s). 

## Table of Contents

1. [Transient Chaos Workload Creation via Delegate on Kubernetes](#transient-chaos-workload-creation-via-delegate)
   - [Create Discovery Agent on EKS Cluster](#create-discovery-agent-on-eks-cluster)
   - [Create Chaos Runner on EKS Cluster](#create-chaos-runner-on-eks-cluster)
  
2. [Discovery Agent Data (Workloads and Connections) Sent to Harness Control Plane (Service Discovery Manager)](#discovery-agent-data-transmission)

3. [Experiment Execution Data (Status, Results) Sent to Harness Control Plane (Infrastructure Server)](#experiment-execution)
   - [Experiment Data Transmission](#experiment-data-transmission)
   - [Fetch Experiment Execution Data](#fetch-experiment-execution-data)
   - [Abort/Stop Experiment Execution](#abortstop-experiment-execution)

4. [Log Data Sent to Harness Control Plane (Log Service)](#log-management)
   - [Open Logstream](#open-logstream)
   - [Stream Execution Logs](#stream-execution-logs)
   - [Close Logstream](#close-logstream)

4. [Long-Running Agent Management & Experiment Execution on Linux](#infrastructure-management-linux)
   - [Register Chaos Infrastructure](#register-chaos-infrastructure)
   - [Update Infrastructure Liveness](#update-infrastructure-liveness)
   - [Get New Task](#get-new-task)
   - [Send Task Status](#send-task-status)
   - [Send Task Logs](#send-task-logs)
   - [Send Task Result](#send-task-result)

---

## Transient Chaos Workload Creation via Delegate

### Create Discovery Agent on EKS Cluster

Creates a discovery agent on an EKS cluster to discover chaos targets and create application maps for tracking resilience score and coverage.

**Endpoint:** `POST /gateway/servicediscovery/api/v1/agents`

**Query Parameters:**
- `routingId` - Routing identifier
- `accountIdentifier` - Account identifier
- `organizationIdentifier` - Organization identifier
- `projectIdentifier` - Project identifier

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <token>
```

**Sample Request Body:**
```json
{
    "environmentIdentifier": "demo",
    "infraIdentifier": "test3",
    "name": "DA-test3",
    "config": {
        "data": {
            "namespaceSelector": "",
            "enableNodeAgent": true,
            "cron": {
                "expression": "15 * * * *"
            },
            "collectionWindowInMin": 1,
            "nodeAgentSelector": ""
        },
        "kubernetes": {
            "namespace": "hce",
            "runAsUser": 2000,
            "runAsGroup": 2000,
            "serviceAccount": ""
        },
        "mtls": {},
        "proxy": {},
        "skipSecureVerify": false
    }
}
```

**Response:**
- Status Code: 201 (Created)

**Response Headers:**
```
Content-Type: application/json; charset=UTF-8
```

**Sample Response Body:**
```json
{
    "id": "68138d5935d08901c7c1cdea",
    "name": "DA-test3",
    "identity": "test3",
    "description": "Discovery agent for infrastructure identity test3",
    "tags": null,
    "accountIdentifier": "nKGIuZPuTyub1HSTL3pnyA",
    "organizationIdentifier": "default",
    "projectIdentifier": "test1",
    "environmentIdentifier": "demo",
    "config": {
        "skipSecureVerify": false,
        "kubernetes": {
            "disableNamespaceCreation": false,
            "namespaced": false,
            "namespace": "hce",
            "serviceAccount": "",
            "runAsUser": 2000,
            "runAsGroup": 2000,
            "imagePullPolicy": "",
            "nodeSelector": null,
            "tolerations": null,
            "resources": {
                "limits": {},
                "requests": {}
            }
        },
        "data": {
            "enableNodeAgent": true,
            "nodeAgentSelector": "",
            "blacklistedNamespaces": null,
            "observedNamespaces": null,
            "namespaceSelector": "",
            "enableOrphanedPod": false,
            "enableBatchResources": false,
            "collectionWindowInMin": 1,
            "cron": {
                "expression": "15 * * * *"
            }
        },
        "collectorImage": "",
        "logWatcherImage": "",
        "imagePullSecrets": null
    },
    "permanentInstallation": false,
    "webhookURL": "",
    "installationType": "CONNECTOR",
    "createdAt": "2025-05-01T15:03:53.114Z",
    "updatedAt": "2025-05-01T15:03:53.114Z",
    "createdBy": "_jXTCj_0RUm3cWW7KQj2ig",
    "updatedBy": "_jXTCj_0RUm3cWW7KQj2ig",
    "removed": false,
    "installationDetails": {
        "id": "68138d5935d08901c7c1cdeb",
        "accountIdentifier": "nKGIuZPuTyub1HSTL3pnyA",
        "organizationIdentifier": "default",
        "projectIdentifier": "test1",
        "environmentIdentifier": "demo",
        "agentID": "68138d5935d08901c7c1cdea",
        "delegateTaskID": "ZH-h77C7TUO9Fag2jgE3cg-DEL",
        "delegateID": "",
        "delegateTaskStatus": "PROCESSED",
        "agentDetails": {
            "status": "PENDING"
        },
        "isCronTriggered": false,
        "logStreamID": "541174ea-5939-4ab1-86d1-dceca3c9843d",
        "logStreamCreatedAt": "2025-05-01T15:03:53.273Z",
        "stopped": false,
        "createdAt": "2025-05-01T15:03:53.273Z",
        "createdBy": "",
        "updatedBy": "",
        "removed": false
    },
    "correlationID": "3148e264ec6b4e438c53ad77699ee77c"
}
```

### Discovery Agent Data Transmission

Allows Discovery Agent to send workloads and connection data to the Service Discovery Manager.

**Endpoint:** Not specified in provided data

**Component:** Discovery Agent (Cluster, Node Collector Pods)

**Request Headers:**
```
Authorization: Token <agent-token>
Content-Type: application/json
Content-Encoding: gzip
```

**Sample Request Body:**
```json
{
    "namespace": "",
    "observedNamespaces": ["",""],
    "namespaceList": [],
    "nodeList": [],
    "serviceList": [],
    "podList": [],
    "replicaSetList": [],
    "replicationControllerList": [],
    "deploymentList": [],
    "statefulSetList": [],
    "daemonSetList": [],
    "jobList": [],
    "cronJobList": []
}
```

OR

```json
{
    "nodeName": "",
    "connectionList": []
}
```
namespace, node, pod denotes to the actual k8s object here is one sameple namespace 👇, We don't fectch config map or secret we any fetch namespace, node, service, pod, replicaset, deployment, statefulset, daemonset, jobs, cronjobs, replication controller.
```text
namespaces --->
typemeta:
    kind: ""
    apiversion: ""
objectmeta:
    name: bofa
    generatename: ""
    namespace: ""
    selflink: ""
    uid: 4baa3793-6f45-436a-8c54-3afe130adaed
    resourceversion: "107365833"
    generation: 0
    creationtimestamp: "2024-01-18T06:22:34Z"
    deletiontimestamp: null
    deletiongraceperiodseconds: null
    labels:
        key: value
        kubernetes.io/metadata.name: bofa
    annotations: {}
    ownerreferences: []
    finalizers: []
    managedfields: []
spec:
    finalizers:
        - kubernetes
status:
    phase: Active
    conditions: []
```

As part of connection we discovery we get IP port of the services, if any k8s application if making any TCP connect call that will be recorded. Sample log is attached 👇
```text
go version go1.23.9
go os linux
go arch amd64
collector configured with value : {WindowInMin:2 Probe:{Kubernetes:{Namespaced:false Namespace:hce EnableBatchResources:false EnableOrphanedPod:false NamespaceSelector: BlacklistedNamespaces:[] ObservedNamespaces:[bofa] NodeSelector:}}}
pref buffer closed, exiting..
connection 10.16.3.18 -> 10.24.11.58:8080 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {db679cb5-a6f7-476a-b2b6-5cebcc5c382b} or pod {} added to the list
skiping connection {1074200576 10.16.3.1 10.16.3.6 8080} as source is not a pod
skiping connection {3088056320 10.16.3.5 10.24.7.97 8080} as source is not a pod
skiping connection {1074200576 10.16.3.1 10.16.3.18 8080} as source is not a pod
connection 10.16.3.6 -> 74.125.202.95:443 from pod {e9adc5f4-fbd2-4973-a4ed-9e0108a64ac4} to svc {} or pod {} added to the list
skiping connection {3088056320 10.16.3.5 10.24.11.156 80} as source is not a pod
skiping connection {3088056320 10.16.3.5 10.24.7.164 5432} as source is not a pod
skiping connection {1074200576 10.16.3.1 10.16.3.13 8080} as source is not a pod
skiping connection {3088056320 10.16.3.5 10.24.8.10 8080} as source is not a pod
skiping connection {118622208 10.16.3.49 35.185.253.115 443} as source is not a pod
skiping connection {1074200576 10.16.3.1 10.16.3.9 8080} as source is not a pod
skiping connection {1074200576 10.16.3.1 10.16.3.12 8080} as source is not a pod
skiping connection {1074200576 127.0.0.1 127.0.0.1 989} as source is not a pod
connection 10.16.3.18 -> 142.250.125.95:443 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {} or pod {} added to the list
skiping connection {454363136 10.128.0.95 35.185.253.115 443} as source is not a pod
connection 10.16.3.18 -> 10.24.2.142:8080 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {07e7eacc-0bcf-4fbd-84dd-e705d9e7935e} or pod {} added to the list
skiping connection {3088056320 10.16.3.5 10.24.13.45 8080} as source is not a pod
connection 10.16.3.18 -> 10.24.7.97:8080 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {d706bf2a-8224-4ebe-9547-4ba5e80262a2} or pod {} added to the list
skiping connection {2551908352 127.0.0.1 127.0.0.1 10248} as source is not a pod
skiping connection {454363136 10.128.0.95 10.24.0.1 443} as source is not a pod
skiping connection {1410662400 10.128.0.95 10.24.14.4 8090} as source is not a pod
connection 10.16.3.18 -> 10.24.13.45:8080 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {738d98ef-36f5-4628-bf2d-ccadd7d00afa} or pod {} added to the list
skiping connection {1074200576 10.16.3.1 10.16.3.10 8000} as source is not a pod
skiping connection {3809740800 10.16.3.51 10.24.0.1 443} as source is not a pod
connection 10.16.3.18 -> 173.194.194.95:443 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {} or pod {} added to the list
skiping connection {1074200576 10.16.3.1 10.16.3.3 5680} as source is not a pod
skiping connection {3088056320 10.16.3.5 10.24.9.222 5432} as source is not a pod
skiping connection {1074200576 10.16.3.1 10.16.3.17 8080} as source is not a pod
skiping connection {3809740800 10.16.3.51 35.185.253.115 443} as source is not a pod
skiping connection {3088056320 10.16.3.5 10.24.2.142 8080} as source is not a pod
connection 10.16.3.18 -> 10.24.8.10:8080 from pod {0d651d0c-c7e4-47bc-8933-9dd16b52ef35} to svc {300a69d2-19c6-436f-8c18-5b7f1d5af0fb} or pod {} added to the list
skiping connection {1074200576 10.16.3.1 10.16.3.5 9115} as source is not a pod
skiping connection {3088056320 10.16.3.5 10.24.11.58 8080} as source is not a pod
skiping connection {1929838592 10.128.0.95 10.128.0.106 443} as source is not a pod
connection 10.16.3.17 -> 10.24.2.142:8080 from pod {94f11ff3-ebee-43a6-8134-06ce58fcfb1d} to svc {07e7eacc-0bcf-4fbd-84dd-e705d9e7935e} or pod {} added to the list
skiping connection {1980893184 10.16.3.50 35.185.253.115 443} as source is not a pod
pushed 9 connections for node name gke-v2-onboarding-pool-1-6ed659e7-6zmq
```
**Response:**
- Status Code: 201/401/404

### Create Chaos Runner on EKS Cluster

Creates a Chaos Runner on an EKS cluster to inject and rollback specified fault into the application microservices/infrastructure and validate hypotheses via specified probes.

Harness controlplane sends a list of Kubernetes resource manifests to the delegate and the delegate deploys it in the target cluster. The delegate token is used in this communication.

Here is one sample yaml spec

```yaml
---
# Source: nginx/templates/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: hce
  labels:
    kubernetes.io/metadata.name: hce
---
# Source: nginx/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: litmus
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
---
# Source: nginx/templates/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ddci-preview
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
stringData:
  ACCOUNT_ID: "nKGIuZPuTyub1HSTL3pnyA"
  INDIRECT_UPLOAD: "true"
  LOG_SERVICE_URL: "https://shovanmaity.pr2.harness.io/gateway/log-service"
  SKIP_SECURE_VERIFY: "false"
  ACCESS_KEY: "<Token>"
  LOG_SERVICE_TOKEN: "<LogServiceToken>"
  PROXY_URL: ""
  HTTP_PROXY: ""
  HTTPS_PROXY: ""
  NO_PROXY: ""
  CLIENT_CERTIFICATE_SECRET: ""
  CLIENT_CERTIFICATE_FILE_PATH: ""
  CLIENT_CERTIFICATE_KEY_FILE_PATH: ""
  CLIENT_CERTIFICATE_PATH: ""
  CLIENT_CERTIFICATE_KEY_PATH: ""
---
# Source: nginx/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ddci-preview
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
data:
  EMISSARY_URL: ""
  INSECURE_SKIP_VERIFY: "false"
  ORG_ID: default
  PROJECT_ID: test1
  ENVIRONMENT_ID: demo
  DDCR_ENDPOINT: http://ddci-preview.hce:8000
  CORRELATION_ID: ddci-preview
  KUBERNETES_INFRA_SERVICE_ENDPOINT: https://shovanmaity.pr2.harness.io/chaos/kserver/api
  INFRA_ID: test3
  LOG_SERVICE_ENDPOINT: https://shovanmaity.pr2.harness.io/gateway/log-service
  LOG_SERVICE_ENABLED: "true"
  LOG_WATCHER_IMAGE: docker.io/harness/chaos-log-watcher:main-latest
---
# Source: nginx/templates/serviceaccount.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: litmus
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
rules:
    # for injecting chaos
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create","delete","get","list","patch","update","watch","deletecollection"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["networkpolicies"]
    verbs: ["create","delete","get","list"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get","list", "patch", "update"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["get", "list"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods","nodes"]
    verbs: ["get", "list"]

    # for checking the app parent resources as they are eligible chaos candidates
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "daemonsets", "statefulsets"]
    verbs: ["list", "get","update"]
  - apiGroups: [""]
    resources: ["replicationcontrollers"]
    verbs: ["get","list"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["get","list"]
  - apiGroups: ["apps.openshift.io"]
    resources: ["deploymentconfigs"]
    verbs: ["list", "get"]
  - apiGroups: ["argoproj.io"]
    resources: ["rollouts"]
    verbs: ["list", "get"]
---
# Source: nginx/templates/serviceaccount.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: litmus
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: litmus
subjects:
- kind: ServiceAccount
  name: litmus
  namespace: hce
---
# Source: nginx/templates/serviceaccount.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: litmus
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
rules:
    # for creating and monitoring the helper pods
  - apiGroups: [""]
    resources: ["pods","secrets", "configmaps","services"]
    verbs: ["create","delete","get","list","patch","update","watch","deletecollection"]
  - apiGroups: ["batch"]
    resources: ["jobs"]
    verbs: ["create","delete","get","list","patch","update","watch","deletecollection"]

    # for tracking & getting logs of helper pods
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["get","list","create"]

    # for self delete
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["create","delete","get","list","patch","update","deletecollection"]
---
# Source: nginx/templates/serviceaccount.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: litmus
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: litmus
subjects:
- kind: ServiceAccount
  name: litmus
  namespace: hce
---
# Source: nginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ddci-preview
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
spec:
  type: ClusterIP
  ports:
  - port: 8000
    targetPort: http
    protocol: TCP
    name: http
  selector:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
---
# Source: nginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ddci-preview
  namespace: hce
  labels:
    ddcr.app/name: ddci-preview
    ddcr.app/namespace: hce
spec:
  replicas: 1
  selector:
    matchLabels:
      ddcr.app/name: ddci-preview
      ddcr.app/namespace: hce
  template:
    metadata:
      labels:
        ddcr.app/name: ddci-preview
        ddcr.app/namespace: hce
    spec:
      serviceAccountName: litmus
      containers:
      - name: scheduler
        image: docker.io/harness/chaos-ddcr:main-latest
        imagePullPolicy: Always
        securityContext:
          runAsGroup: 2000
          runAsUser: 2000
        env:
        - name: CHAOS_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        envFrom:
        - configMapRef:
            name: ddci-preview
        - secretRef:
            name: ddci-preview
        ports:
        - name: http
          containerPort: 8000
          protocol: TCP
        resources:
          limits:
            cpu: 100m
            ephemeral-storage: 500Mi
            memory: 500Mi
          requests:
            cpu: 50m
            ephemeral-storage: 250Mi
            memory: 250Mi

```

**Component:** Delegate

**Notes:** Needed to inject and rollback the specified fault into the application microservices/infrastructure as well as validate the hypothesis via the specified probe.

---

## Experiment Execution

### Experiment Data Transmission

Allows Chaos Runner to send experiment execution data (fault, action, and probe details) to the Infrastructure Server.

**Endpoint:** `POST https://app.harness.io/gateway/chaos/kserver/api/infra/v2/task/response/prodecommerce`

**Query Parameters:**
- `environmentIdentifier` - Environment identifier
- `accountIdentifier` - Account identifier
- `organizationIdentifier` - Organization identifier
- `projectIdentifier` - Project identifier

**Request Headers:**
```
X-Access-Key: accessKey
Content-Type: application/json
Accept: application/json
```

**Sample Request Body:**
```json
{
  "taskType": "WORKFLOW_EVENT",
  "executionType": "SYNC",
  "infraId": "prodecommerce",
  "args": {
    "completed": "false",
    "executionData": "{\"workflowType\":\"events\",\"eventType\":\"UPDATE\",\"namespace\":\"v2\",\"name\":\"api-payload-test\",\"creationTimestamp\":\"1746117494\",\"phase\":\"Running\",\"message\":\"\",\"startedAt\":\"1746117494\",\"finishedAt\":\"\",\"nodes\":{\"api-payload-test\":{\"name\":\"api-payload-test\",\"phase\":\"Running\",\"message\":\"\",\"startedAt\":\"1746117494\",\"finishedAt\":\"\",\"children\":[\"api-payload-test-pod-delete-wm3\"],\"type\":\"Steps\"},\"api-payload-test-pod-cpu-hog-pys\":{\"name\":\"pod-cpu-hog-pys\",\"phase\":\"Pending\",\"message\":\"\",\"startedAt\":\"\",\"finishedAt\":\"\",\"children\":null,\"type\":\"K8sTask\",\"chaosData\":{\"namespace\":\"v2\",\"experimentName\":\"pod-cpu-hog\",\"experimentStatus\":\"Pending\",\"lastUpdatedAt\":\"1746117494\",\"experimentVerdict\":\"\",\"probeSuccessPercentage\":\"\",\"failStep\":\"\",\"taskDefinitionCR\":\"\",\"errorCode\":\"\",\"streamID\":\"\",\"helperPodDetails\":null}},\"api-payload-test-pod-delete-wm3\":{\"name\":\"pod-delete-wm3\",\"phase\":\"Running\",\"message\":\"\",\"startedAt\":\"1746117528\",\"finishedAt\":\"\",\"children\":[\"api-payload-test-pod-cpu-hog-pys\"],\"type\":\"K8sTask\",\"chaosData\":{\"namespace\":\"v2\",\"experimentName\":\"pod-delete\",\"experimentStatus\":\"Running\",\"lastUpdatedAt\":\"1746117528\",\"experimentVerdict\":\"Awaited\",\"probeSuccessPercentage\":\"\",\"failStep\":\"\",\"taskDefinitionCR\":\"\",\"errorCode\":\"\",\"streamID\":\"76079280-cfd6-4390-8ea4-86771c10f44e-pod-delete-wm3\",\"helperPodDetails\":null}}},\"executedBy\":\"\"}",
    "infraID": "prodecommerce",
    "notifyID": "b957d7d0-a83d-4728-988d-a035575672e4",
    "updatedBy": "",
    "workflowID": "23a6c18c-63c6-4948-b932-ae8eef3921f6",
    "workflowName": "api-payload-test",
    "workflowRunID": "76079280-cfd6-4390-8ea4-86771c10f44e"
  }
}
```

**Response:**
- Status Code: 200 (Created)

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```
Received task callback for taskType: WORKFLOW_EVENT, response: Workflow run received for for WorkflowID: 23a6c18c-63c6-4948-b932-ae8eef3921f6, WorkflowRunID: 76079280-cfd6-4390-8ea4-86771c10f44e
```

### Fetch Experiment Execution Data

Allows Chaos Runner to fetch "Current" Experiment Execution Data for State Management Purposes.

**Endpoint:** `GET https://app.harness.io/gateway/chaos/kserver/api/infra/v2/task/experimentrun/prodecommerce/23a6c18c-63c6-4948-b932-ae8eef3921f6/76079280-cfd6-4390-8ea4-86771c10f44e`

**Query Parameters:**
- `environmentIdentifier` - Environment identifier
- `accountIdentifier` - Account identifier
- `organizationIdentifier` - Organization identifier
- `projectIdentifier` - Project identifier

**Request Headers:**
```
X-Access-Key: accessKey
Content-Type: application/json
Accept: application/json
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```json
{
  "data": {
    "notifyID": "b957d7d0-a83d-4728-988d-a035575672e4",
    "resiliencyScore": 0,
    "faultsPassed": 0,
    "faultsFailed": 0,
    "faultsAwaited": 0,
    "faultsStopped": 0,
    "faultsNA": 0,
    "totalFaults": 0,
    "accountID": "vAeOvaJ8R_iQiy_GpA6sTQ",
    "orgID": "default",
    "projectID": "airecommendations",
    "experimentType": "experiment_v2",
    "infraID": "test/prodecommerce",
    "experimentRunID": "76079280-cfd6-4390-8ea4-86771c10f44e",
    "experimentID": "23a6c18c-63c6-4948-b932-ae8eef3921f6",
    "experimentName": "api-payload-test",
    "phase": "Running",
    "executionData": {
      "workflowType": "events",
      "eventType": "CREATE",
      "namespace": "v2",
      "name": "api-payload-test",
      "creationTimestamp": "1746117494",
      "phase": "Running",
      "message": "",
      "startedAt": "1746117494",
      "finishedAt": "",
      "nodes": {
        "api-payload-test": {
          "name": "api-payload-test",
          "phase": "Running",
          "message": "",
          "startedAt": "1746117494",
          "finishedAt": "",
          "children": [
            "api-payload-test-pod-delete-wm3"
          ],
          "type": "Steps"
        },
        "api-payload-test-pod-cpu-hog-pys": {
          "name": "pod-cpu-hog-pys",
          "phase": "Pending",
          "message": "",
          "startedAt": "",
          "finishedAt": "",
          "children": null,
          "type": "K8sTask",
          "chaosData": {
            "namespace": "v2",
            "experimentName": "pod-cpu-hog",
            "experimentStatus": "Pending",
            "lastUpdatedAt": "1746117494",
            "experimentVerdict": "",
            "probeSuccessPercentage": "",
            "failStep": "",
            "taskDefinitionCR": "",
            "errorCode": "",
            "streamID": "",
            "helperPodDetails": null
          }
        },
        "api-payload-test-pod-delete-wm3": {
          "name": "pod-delete-wm3",
          "phase": "Pending",
          "message": "",
          "startedAt": "",
          "finishedAt": "",
          "children": [
            "api-payload-test-pod-cpu-hog-pys"
          ],
          "type": "K8sTask",
          "chaosData": {
            "namespace": "v2",
            "experimentName": "pod-delete",
            "experimentStatus": "Pending",
            "lastUpdatedAt": "1746117494",
            "experimentVerdict": "",
            "probeSuccessPercentage": "",
            "failStep": "",
            "taskDefinitionCR": "",
            "errorCode": "",
            "streamID": "",
            "helperPodDetails": null
          }
        }
      },
      "executedBy": ""
    },
    "revisionID": "98471b89-0ad4-416e-b0d9-053cae665faf",
    "errorResponse": "",
    "networkMapID": "",
    "experimentYaml": {
      "kind": "KubernetesChaosExperiment",
      "apiVersion": "litmuschaos.io/v1alpha1",
      "metadata": {
        "name": "api-payload-test",
        "namespace": "v2"
      },
      "spec": {
        "tasks": [
          {
            "name": "pod-delete-wm3",
            "definition": {
              "targets": {
                "selectors": {
                  "workloads": [
                    {
                      "kind": "deployment",
                      "namespace": "demo",
                      "names": "podtato-hats",
                      "labels": ""
                    }
                  ]
                },
                "application": null
              },
              "chaos": {
                "experiment": "pod-delete",
                "image": "docker.io/harness/chaos-ddcr-faults:main-latest",
                "imagePullPolicy": "Always",
                "defaultHealthCheck": false,
                "env": [
                  {
                    "name": "TOTAL_CHAOS_DURATION",
                    "value": "30"
                  },
                  {
                    "name": "CHAOS_INTERVAL",
                    "value": "10"
                  },
                  {
                    "name": "FORCE",
                    "value": "false"
                  },
                  {
                    "name": "RAMP_TIME"
                  },
                  {
                    "name": "POD_AFFECTED_PERCENTAGE"
                  },
                  {
                    "name": "TARGET_PODS"
                  },
                  {
                    "name": "NODE_LABEL"
                  },
                  {
                    "name": "SEQUENCE",
                    "value": "parallel"
                  }
                ],
                "components": {
                  "resources": {},
                  "sidecar": [
                    {
                      "image": "docker.io/harness/chaos-log-watcher:main-latest",
                      "imagePullPolicy": "Always",
                      "secrets": null,
                      "envFrom": null,
                      "env": null
                    }
                  ]
                }
              }
            },
            "probeRef": [
              {
                "probeID": "google-ping",
                "mode": "SOT"
              }
            ],
            "values": null
          },
          {
            "name": "pod-cpu-hog-pys",
            "definition": {
              "targets": {
                "selectors": {
                  "workloads": [
                    {
                      "kind": "deployment",
                      "namespace": "demo",
                      "names": "podtato-left-leg",
                      "labels": ""
                    }
                  ]
                },
                "application": null
              },
              "chaos": {
                "experiment": "pod-cpu-hog",
                "image": "docker.io/harness/chaos-ddcr-faults:main-latest",
                "imagePullPolicy": "Always",
                "defaultHealthCheck": false,
                "env": [
                  {
                    "name": "TOTAL_CHAOS_DURATION",
                    "value": "60"
                  },
                  {
                    "name": "CPU_CORES",
                    "value": "1"
                  },
                  {
                    "name": "CPU_LOAD",
                    "value": "100"
                  },
                  {
                    "name": "POD_AFFECTED_PERCENTAGE"
                  },
                  {
                    "name": "CONTAINER_RUNTIME",
                    "value": "containerd"
                  },
                  {
                    "name": "SOCKET_PATH",
                    "value": "/run/containerd/containerd.sock"
                  },
                  {
                    "name": "LIB_IMAGE",
                    "value": "docker.io/harness/chaos-ddcr-faults:main-latest"
                  },
                  {
                    "name": "RAMP_TIME"
                  },
                  {
                    "name": "TARGET_CONTAINER"
                  },
                  {
                    "name": "TARGET_PODS"
                  },
                  {
                    "name": "NODE_LABEL"
                  },
                  {
                    "name": "SEQUENCE",
                    "value": "parallel"
                  }
                ],
                "components": {
                  "resources": {},
                  "sidecar": [
                    {
                      "image": "docker.io/harness/chaos-log-watcher:main-latest",
                      "imagePullPolicy": "Always",
                      "secrets": null,
                      "envFrom": null,
                      "env": null
                    }
                  ]
                }
              }
            },
            "probeRef": [
              {
                "probeID": "google-ping",
                "mode": "SOT"
              }
            ],
            "values": null
          }
        ],
        "experimentId": "23a6c18c-63c6-4948-b932-ae8eef3921f6",
        "experimentRunId": "",
        "steps": [
          [
            {
              "name": "pod-delete-wm3",
              "status": ""
            }
          ],
          [
            {
              "name": "pod-cpu-hog-pys",
              "status": ""
            }
          ]
        ],
        "cleanupPolicy": "delete",
        "serviceAccountName": "litmus"
      },
      "variables": null
    },
    "targetedServices": null,
    "securityGovernance": {
      "Name": "Evaluate Rules",
      "Type": "SGNode",
      "Message": "No rules apply to this experiment run",
      "Phase": "Passed",
      "SecurityGovernanceNodeData": {
        "PassedRules": [],
        "FailedRules": [],
        "SkippedRules": []
      },
      "StartedAt": 1746117474328,
      "FinishedAt": 1746117474330
    },
    "faultsWithProbes": [
      {
        "faultName": "pod-delete-wm3",
        "probes": [
          {
            "ProbeID": "google-ping",
            "Name": "google-ping",
            "Mode": "SOT",
            "RevisionID": "2a795356-ada8-4027-9c97-c3e4fd337fed"
          }
        ]
      },
      {
        "faultName": "pod-cpu-hog-pys",
        "probes": [
          {
            "ProbeID": "google-ping",
            "Name": "google-ping",
            "Mode": "SOT",
            "RevisionID": "2a795356-ada8-4027-9c97-c3e4fd337fed"
          }
        ]
      }
    ],
    "createdBy": "ADh0pl0pS7WE1a12H7flDA",
    "updatedBy": "",
    "updatedAt": 1746117495518,
    "createdAt": 1746117475263,
    "isRemoved": false,
    "runSequence": 1,
    "completed": false
  }
}
```

---

### Abort/Stop Experiment Execution

Enables aborting of one or more ongoing experiments in the EKS cluster.

Same as the [Chaos-Runner creation flow](#create-chaos-runner-on-eks-cluster), with some extra annotations in the  config map

**Component:** Delegate

**Notes:** Needed to enable abort of one or more ongoing experiments in the EKS cluster.

## Log Management

### Open Logstream

Opens a log stream for capturing execution logs.

**Endpoint:** `POST https://app.harness.io/gateway/log-service`

**Query Parameters:**
- `accountID` - Account identifier
- `key` - Log stream key (e.g., `76079280-cfd6-4390-8ea4-86771c10f44e-pod-delete`)

**Request Headers:**
```
X-Harness-Token: accessKey
Content-Type: application/json
Accept: application/json
```

**Component:** Fault & Helper Pods

### Stream Execution Logs

Streams fault and helper pods execution logs to the Log Service.

**Endpoint:** `PUT https://app.harness.io/gateway/log-service/stream`

**Query Parameters:**
- `accountID` - Account identifier
- `key` - Log stream key (e.g., `76079280-cfd6-4390-8ea4-86771c10f44e-pod-delete`)

**Request Headers:**
```
X-Harness-Token: accessKey
Content-Type: application/json
Accept: application/json
```

**Sample Request Body:**
```json
[{
  "level": "info",
  "pos": 1,
  "out": "[Start]: pod-delete FAULT START",
  "time": "2025-05-01T17:27:56Z",
  "args": {
    "PodName": "pod-delete-wm3-d6moyr-vwkng",
    "PodNamespace": "v2"
  }
}]
```

**Notes:** Needed to capture execution logs for debug and reference purposes.

**Alternative Endpoint:** `POST https://app.harness.io/gateway/log-service/blob`

### Close Logstream

Closes a log stream.

**Endpoint:** `DELETE https://app.harness.io/gateway/log-service`

**Query Parameters:**
- `accountID` - Account identifier
- `key` - Log stream key (e.g., `76079280-cfd6-4390-8ea4-86771c10f44e-pod-delete`)

**Request Headers:**
```
X-Harness-Token: accessKey
Content-Type: application/json
Accept: application/json
```

**Component:** Fault & Helper Pods

---

## Infrastructure Management (Linux)

### Register Chaos Infrastructure

Confirms and registers a new Linux or Windows chaos infrastructure with the control plane.

**Endpoint:** `POST https://app.harness.io/gratis/chaos/lserver/api/infra/confirm`

**Request Headers:**
```
Content-Type: application/json
```

**Sample Request Body:**
```json
{
  "hostname": "ip-172-31-13-123",
  "version": "1.59.0"
}
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```json
{
  "infraID": "6bdacf96-14fc-4edd-9277-8707b14c2f75",
  "newAccessKey": "v1yln9st88k66rcgxe0rwhqhpifw71qz"
}
```

### Update Infrastructure Liveness

Periodically signals the control plane that the chaos infrastructure is alive and running.

**Endpoint:** `POST https://app.harness.io/gratis/chaos/lserver/api/infra/liveness`

**Request Headers:**
```
Content-Type: application/json
```

**Sample Request Body:**
```json
{}
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```json
{ "message": "OK" }
```

### Get New Task

Polls the control plane for new chaos experiment tasks to execute.

**Endpoint:** `GET https://app.harness.io/gratis/chaos/lserver/api/infra/task`

**Request Headers:**
```
Content-Type: application/json
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```json
{
  "taskList": [
    {
      "taskName": "linux-cpu-hog-1746115610-linux-cpu-stress-hv3pg",
      "action": {
        "chaos": {
          "experiment": "linux-cpu-stress",
          "experimentInputs": [
            { "name": "Load", "value": "100" },
            { "name": "Workers", "value": "1" },
            { "name": "Duration", "value": "30s" }
          ],
          "probes": [
            {
              "name": "system-probe",
              "type": "cmdProbe",
              "cmdProbe/inputs": {
                "command": "./healthcheck",
                "comparator": {
                  "type": "string",
                  "criteria": "contains",
                  "value": "[P000]"
                }
              },
              "runProperties": {
                "probeTimeout": "180s",
                "interval": "1s",
                "attempt": 1
              },
              "mode": "EOT"
            }
          ]
        }
      },
      "lceUID": "4e2a8fb3-3736-4c12-ae55-c86c6030697d",
      "streamID": "5e83d1aa-bee0-48f6-a1a4-9bb235b17e95-linux-cpu-hog-1746115610-linux-cpu-stress-hv3pg",
      "accountID": "cTU1lRSWS2SSRV9phKvuOA"
    }
  ],
  "isActive": false
}
```

### Send Task Status

Sends the current status (e.g., Running, Completed) of a specific chaos task back to the control plane.

**Endpoint:** `POST https://app.harness.io/gratis/chaos/lserver/api/infra/task/status/<task-name>`

**Request Headers:**
```
Content-Type: application/json
```

**Sample Request Body:**
```json
{
  "taskStatus": "Running",
  "lceUID": "4e2a8fb3-3736-4c12-ae55-c86c6030697d"
}
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```json
{ "success": true }
```

### Send Task Logs

Streams logs generated during chaos execution (e.g., probe output, chaos injection) to the control plane.

**Endpoint:** `POST https://app.harness.io/gratis/chaos/lserver/api/infra/logs`

**Request Headers:**
```
Content-Type: application/json
```

**Sample Request Body:**
```json
{
  "streamID": "5e83d1aa-bee0-48f6-a1a4-9bb235b17e95-linux-cpu-hog-1746115610-linux-cpu-stress-hv3pg",
  "accountID": "cTU1lRSWS2SSRV9phKvuOA",
  "logs": [
    {
      "level": "info",
      "pos": 0,
      "out": "[Status]: Verify experiment steady state",
      "time": "2025-05-01T16:06:55Z"
    },
    {
      "level": "info",
      "pos": 0,
      "out": "[Info]: The stress process parameters are as follows",
      "time": "2025-05-01T16:06:55Z",
      "args": {
        "CPU": "\"1\"",
        "Load": "\"100%!(MISSING)\""
      }
    },
    {
      "level": "info",
      "pos": 0,
      "out": "[Chaos]: Injecting CPU stress chaos",
      "time": "2025-05-01T16:06:55Z"
    }
  ]
}
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```json
{ "success": true }
```

### Send Task Result

Sends the final result with probe outcomes of a chaos experiment to the control plane after execution ends.

**Endpoint:** `POST https://app.harness.io/gratis/chaos/lserver/api/infra/task/result/<task-name>`

**Request Headers:**
```
Content-Type: application/json
```

**Sample Request Body:**
```json
{
  "result": {
    "taskStatus": "Error",
    "verdict": "Error",
    "errorOutput": {
      "errorCode": "E2008: CMD_PROBE_ERROR",
      "reason": "{\"errorCode\":\"E2008: CMD_PROBE_ERROR\",\"phase\":\"PostChaos\",\"reason\":\"unable to run command, exit status 127; /bin/sh: 1: ./healthcheck: not found\\n\",\"target\":\"{chaosInfraID: 6bdacf96-14fc-4edd-9277-8707b14c2f75, probeName: system-probe}\"}"
    },
    "probeSuccessPercentage": "0",
    "probeStatuses": [
      {
        "name": "system-probe",
        "type": "cmdProbe",
        "mode": "EOT",
        "status": {
          "verdict": "N/A",
          "description": "Either probe is not executed or not evaluated"
        }
      }
    ]
  },
  "logs": "{...large multiline string with merged logs...}"
}
```

**Response:**
- Status Code: 200

**Response Headers:**
```
Content-Type: application/json
```

**Sample Response Body:**
```
HTTP/1.1 200 OK
```
