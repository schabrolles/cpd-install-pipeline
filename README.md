# cpd-install-pipeline
Install automatically IBM Cloudpak for Data using Tekton Pipeline

<img width="1412" alt="cpd-pipeline" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/a85b1d1f-e93c-41d6-9b2b-dd106035c4ac">

## Prerequisite:

- Openshift Cluster > 4.12
- Openshift Pipeline Operator
- **Cluster can access to registries icr.io and cp.icr.io**

### 1- Configuring cpd-install Tekton Pipeline

After having installed the openshift-pipeline operator, use helm to deploy the tasks and pipeline in a namespace of your cluster.

```
helm install <name> <helm_chart> --create-namespace -n <namespace> [--set arch=(ppc64le|s390x)]
```
- `helm_chart` can be found in the [release](https://github.com/schabrolles/cpd-install-pipeline/releases) section.
- `arch` *optional* value allows pipeline to be adapted for **non x86** cluster (only `ppc64le` or `s390x`) (`arch=""` means `x86` which is the default when not set)

example: 
```
helm install cpd-install https://github.com/schabrolles/cpd-install-pipeline/releases/download/v1.1/cpd-install-pipeline-0.1.1.tgz \
--create-namespace -n cpd-install --set arch=ppc64le
```
This will:
   - create a project `cpd-install`
   - give the **::“cluster-admin”::** right to the “**pipeline**” service-account of this project
   - create a tekton task “**olm-utils**” based on the official `olm-utils-v2` from IBM (icr.io)
   - create a tekton pipeline to install automatically the cloud pak for data `components` you choose

### 2a- Starting pipeline with GUI

If you plan to start the pipeline from the GUI (openshift-console), you must change the default pipeline timeout by using the following command:

```other
oc set data -n openshift-pipelines cm/config-defaults default-timeout-minutes="300"
```

From the openshift console, enter in the project `cpd-install` and select the tab “pipeline" from the menu on the left.

<img width="1666" alt="pipeline-gui1" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/0eea2ace-42d6-449f-96f7-4cc3d82b311d">

Click on the **cpd-install** pipeline.

On the next page, click on the action button on the right end side of the page and select “**start**”

<img width="1666" alt="pipeline-gui2" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/73c2f844-26e7-4022-8db3-d930ad539b23">

This will open a forms to customize your installation.

- You need to provide at least your **IBM Entitlement key**
- You can change the version, name of namespaces or storageclass to use
- If you are not running a production service, set **production** to **false**
- if you don’t have GPUs but still want to deploy watsonx, set **NO_GPU** to **true**

click on **Start**

The full installation duration is about ~2h

<img width="1380" alt="pipeline-gui3" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/e9e6a549-1eca-4b76-9faf-79c5e4d1840a">

### 2b- Starting pipeline by applying YAML

Use `oc create` with the following YAML.

Change the Parameters to fit to your needs:

- You need to provide at least your **IBM Entitlement key**
- You can change the version, name of namespaces or storageclass to use
- If you are not running a production service, set **production** to **false**
- if you don’t have GPUs but still want to deploy watsonx, set **NO_GPU** to **true**

```yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: cpd-install-
  namespace: cpd-install
spec:
  params:
  - name: OLM_UTILS_IMAGE
    value: icr.io/cpopen/cpd/olm-utils-v2
  - name: VERSION
    value: 4.8.5
  - name: COMPONENTS
    value: watsonx_ai
  - name: PROJECT_CPD_INST_OPERANDS
    value: cpd
  - name: PROJECT_CPD_INST_OPERATORS
    value: cpd-operators
  - name: PROJECT_CERT_MANAGER
    value: ibm-cert-manager
  - name: PROJECT_LICENSE_SERVICE
    value: ibm-licensing
  - name: PROJECT_SCHEDULING_SERVICE
    value: ibm-scheduler
  - name: IBM_ENTITLEMENT_KEY
    value: >>> PUT YOUR IBM ICR KEY HERE <<<
  - name: STG_CLASS_BLOCK
    value: ocs-storagecluster-ceph-rbd
  - name: STG_CLASS_FILE
    value: ocs-storagecluster-cephfs
  - name: ENTITLEMENT
    value: watsonx-ai
  - name: SCHEDULER
    value: "false"
  - name: PRODUCTION
    value: "false"
  - name: ACCEPT_LICENCE
    value: "true"
  - name: NO_GPU
    value: "false"
  pipelineRef:
    name: cpd-install
  taskRunTemplate:
    serviceAccountName: pipeline
  timeouts:
    pipeline: 5h0m0s
```

You can use openshift console to follow the pipeline.
If you prefer you can download the `tkn` cli ([https://docs.openshift.com/container-platform/4.15/cli_reference/tkn_cli/installing-tkn.html#installing-tkn](https://docs.openshift.com/container-platform/4.15/cli_reference/tkn_cli/installing-tkn.html#installing-tkn))

```plaintext
tkn pr logs -n cpd-install --follow
```
