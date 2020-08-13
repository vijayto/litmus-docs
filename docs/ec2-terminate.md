---
id: ec2-terminate
title: EC2 terminate Experiment Details
sidebar_label: Pod Delete
---
------

## Experiment Metadata

<table>
  <tr>
    <th> Type </th>
    <th> Description </th>
    <th> Tested K8s Platform </th>
  </tr>
  <tr>
    <td> Generic </td>
    <td> Fail the application pod </td>
    <td> GKE, Konvoy(AWS), Packet(Kubeadm), Minikube, EKS, AKS </td>
  </tr>
</table>

## Prerequisites

- Ensure that the Litmus Chaos Operator is running by executing `kubectl get pods` in operator namespace (typically, `litmus`).If not, install from [here](https://docs.litmuschaos.io/docs/getstarted/#install-litmus)
- Ensure that the `pod-delete` experiment resource is available in the cluster by executing                         `kubectl get chaosexperiments` in the desired namespace. If not, install from [here](https://hub.litmuschaos.io/api/chaos/1.6.0?file=charts/generic/pod-delete/experiment.yaml)

## Entry Criteria

- Application pods are healthy before chaos injection

## Exit Criteria

- Application pods are healthy post chaos injection

## Details

- Causes EC2 terminate of instances where application resources are running
- Tests deployment sanity (replica availability & uninterrupted service) and recovery workflow of the application

## Integrations

- EC2 failures can be effected using one of these chaos libraries: `litmus`, `powerfulseal`
- The desired chaos library can be selected by setting one of the above options as value for the env variable `LIB`

## Steps to Execute the Chaos Experiment

- This Chaos Experiment can be triggered by creating a ChaosEngine resource on the cluster. To understand the values to provide in a ChaosEngine specification, refer [Getting Started](getstarted.md/#prepare-chaosengine)

- Follow the steps in the sections below to create the chaosServiceAccount, prepare the ChaosEngine & execute the experiment.

### Prepare chaosServiceAccount

- Use this sample RBAC manifest to create a chaosServiceAccount in the desired (app) namespace. This example consists of the minimum necessary role permissions to execute the experiment.
- The RBAC sample manifest is different for both LIB (litmus, powerseal). Use the respective rbac sample manifest on the basis of LIB ENV.

#### Sample Rbac Manifest for litmus LIB

[embedmd]:# (https://raw.githubusercontent.com/litmuschaos/chaos-charts/master/charts/generic/pod-delete/rbac.yaml yaml)
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: chaos-admin
  labels:
    name: chaos-admin
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: chaos-admin
  labels:
    name: chaos-admin
rules:
- apiGroups: ["","apps","batch","extensions","litmuschaos.io","openebs.io","storage.k8s.io"]
  resources: ["chaosengines","chaosexperiments","chaosresults","configmaps","cstorpools","cstorvolumereplicas","events","jobs","persistentvolumeclaims","persistentvolumes","pods","pods/exec","pods/log","secrets","storageclasses","chaosengines","chaosexperiments","chaosresults","configmaps","cstorpools","cstorvolumereplicas","daemonsets","deployments","events","jobs","persistentvolumeclaims","persistentvolumes","pods","pods/eviction","pods/exec","pods/log","replicasets","secrets","services","statefulsets","storageclasses"]
  verbs: ["create","delete","get","list","patch","update"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","list","patch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: chaos-admin
  labels:
    name: chaos-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: chaos-admin
subjects:
- kind: ServiceAccount
  name: chaos-admin
  namespace: default

```

#### Sample Rbac Manifest for powerfulseal LIB

[embedmd]:# (https://raw.githubusercontent.com/litmuschaos/chaos-charts/master/charts/generic/pod-delete/powerfulseal_rbac.yaml yaml)
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: chaos-admin
  labels:
    name: chaos-admin
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: chaos-admin
  labels:
    name: chaos-admin
rules:
- apiGroups: ["","apps","batch","extensions","litmuschaos.io","openebs.io","storage.k8s.io"]
  resources: ["chaosengines","chaosexperiments","chaosresults","configmaps","cstorpools","cstorvolumereplicas","events","jobs","persistentvolumeclaims","persistentvolumes","pods","pods/exec","pods/log","secrets","storageclasses","chaosengines","chaosexperiments","chaosresults","configmaps","cstorpools","cstorvolumereplicas","daemonsets","deployments","events","jobs","persistentvolumeclaims","persistentvolumes","pods","pods/eviction","pods/exec","pods/log","replicasets","secrets","services","statefulsets","storageclasses"]
  verbs: ["create","delete","get","list","patch","update"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","list","patch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: chaos-admin
  labels:
    name: chaos-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: chaos-admin
subjects:
- kind: ServiceAccount
  name: chaos-admin
  namespace: default


```

### Prepare ChaosEngine

- Provide the application info in `spec.appinfo`
- Override the experiment tunables if desired in `experiments.spec.components.env`
- To understand the values to provided in a ChaosEngine specification, refer [ChaosEngine Concepts](chaosengine-concepts.md)

#### Supported Experiment Tunables

<table>
  <tr>
    <th> Variables </th>
    <th> Description  </th>
    <th> Specify In ChaosEngine </th>
    <th> Notes </th>
  </tr>
  <tr>
    <td> LABEL_NAME </td>
    <td> Labels given to application under test </td>
    <td> Required </td>
    <td> No defaults defined </td>
  </tr>
  <tr>
    <td> AWS_ACCOUNT </td>
    <td> AWS account id which is under test </td>
    <td> Required </td>
    <td> No defaults defined </td>
  </tr>
  <tr>
    <td> LIB </td>
    <td> The chaos lib used to inject the chaos </td>
    <td> Optional  </td>
    <td> Defaults to `litmus`. Supported: `litmus`, `powerfulseal`. In case of powerfulseal use the <a href="https://github.com/litmuschaos/chaos-charts/blob/master/charts/generic/pod-delete/powerfulseal_experiment.yaml">powerfulseal </a>experiment CR. </td>
  </tr>
  <tr>
    <td> AWS_ROLE  </td>
    <td> AWS role defined in AWS account which has permissions for ec2 terminate</td>
    <td> Required  </td>
    <td> No Defaults defined  </td>
  </tr>
  <tr>
    <td> AWS_REGION </td>
    <td> AWS region under test </td>
    <td> Required  </td>
    <td> No defaults defined </td>
  </tr>
  <tr>
    <td> AWS_AZ </td>
    <td> Availability zone in AWS </td>
    <td> Required  </td>
    <td> No defaults defined </td>
  </tr>
  <tr>
    <td> TEST_NAMESPACE </td>
    <td> Namespace where in results are going to be persisted - Chaosresults</td>
    <td> Required  </td>
    <td> Chaos results object is going to be applied from this namespace </td>
  </tr>
</table>

#### Sample ChaosEngine Manifest

[embedmd]:# (https://raw.githubusercontent.com/litmuschaos/chaos-charts/master/charts/generic/pod-delete/engine.yaml yaml)
```yaml
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: aws-ec2-terminate
  namespace: default
spec:
  #ex. values: ns1:name=percona,ns2:run=nginx  
  appinfo: 
    appns: default
    applabel: "app=test"
    appkind: deployment
  jobCleanUpPolicy: delete
  monitoring: false
  annotationCheck: 'false'
  engineState: 'active'
  chaosServiceAccount: chaos-admin
  components:
    runner:
      runnerannotation:
        iam.amazonaws.com/role: "pod-iam-role"
  experiments:
    - name: aws-ec2-terminate
      spec:
        components:
          experimentannotation:
            iam.amazonaws.com/role: "pod-iam-role"
          env: 
            - name: NAME_SPACE
              value: default
            - name: LABEL_NAME
              value: app=test
            - name: APP_ENDPOINT
              value: localhost
            - name: FILE
              value: 'ec2-delete.json'
            - name: AWS_ROLE
              value: 'chaos_role_at_aws_with_permissions_for_ec2_terminate'
            - name: AWS_ACCOUNT
              value: ''
            - name: AWS_REGION
              value: ''
            - name: AWS_AZ
              value: ''
            - name: AWS_RESOURCE
              value: 'ec2-iks'  
            - name: AWS_SSL
              value: 'false'
            - name: REPORT
              value: 'true'
            - name: REPORT_ENDPOINT
              value: 'none'
            - name: TEST_NAMESPACE
              value: 'default'
```

### Create the ChaosEngine Resource

- Create the ChaosEngine manifest prepared in the previous step to trigger the Chaos.

  `kubectl apply -f chaosengine.yml`

- If the chaos experiment is not executed, refer to the [troubleshooting](https://docs.litmuschaos.io/docs/faq-troubleshooting/) 
  section to identify the root cause and fix the issues.

### Watch Chaos progress

- View pod terminations & recovery by setting up a watch on the pods in the application namespace

  `watch -n 1 kubectl get pods -n <application-namespace>`

### Abort/Restart the Chaos Experiment

- To stop the pod-delete experiment immediately, either delete the ChaosEngine resource or execute the following command: 

  `kubectl patch chaosengine <chaosengine-name> -n <namespace> --type merge --patch '{"spec":{"engineState":"stop"}}'` 

- To restart the experiment, either re-apply the ChaosEngine YAML or execute the following command: 

  `kubectl patch chaosengine <chaosengine-name> -n <namespace> --type merge --patch '{"spec":{"engineState":"active"}}'`

### Check Chaos Experiment Result

- Check whether the application is resilient to the pod failure, once the experiment (job) is completed. The ChaosResult resource name is derived like this: `<ChaosEngine-Name>-<ChaosExperiment-Name>`.

  `kubectl describe chaosresult ec2-terminate-<timestamp> -n <application-namespace>`

## Application Pod Failure Demo

- A sample recording of this experiment execution is provided [here](https://youtu.be/X3JvY_58V9A) 
