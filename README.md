# cpd-install-pipeline

Install automatically IBM Cloudpak for Data using Tekton Pipeline

<img width="1708" alt="image" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/102ca157-3dce-4930-9c26-ce2058639eef">

## Prerequisite:

- Openshift Cluster > 4.12
- Openshift Pipeline Operator
- **Cluster can access to registries icr.io and cp.icr.io**

## Table of Content:

- **Installation**
   - cli based: [Using helm cli](#1--Configuring-cpd-install-Tekton-Pipeline-from-the-cli)
   - GUI based: [Using openshift web-console](#1bis---Adding-helm-repo-to-openshift-to-use-Graphical-Interface)
- **Starting Pipeline**
   - GUI based: [Using openshift pipeline graphical interface](#2--Starting-pipeline-with-GUI)
   - cli based: [applying pipelineRun yaml file](2bis--Starting-pipeline-from-cli-(by-applying-YAML))
 
## Installation
* 1- [Using helm cli](#1--Configuring-cpd-install-Tekton-Pipeline-from-the-cli) (cli based)
* 1bis- [Using openshift web-console](#1bis---Adding-helm-repo-to-openshift-to-use-Graphical-Interface) (GUI based)

### 1- Configuring cpd-install Tekton Pipeline from the cli

After having installed the openshift-pipeline operator, use helm to deploy the tasks and pipeline in a namespace of your cluster.

- adding the helm repo
```
helm repo add cpd-install-pipeline https://schabrolles.github.io/cpd-install-pipeline
```

- deploying the pipeline configuration with helm-cli
```
helm install <name> cpd-install-pipeline/cpd-install-pipeline --create-namespace -n <namespace> [--set arch=(ppc64le|s390x)]
```
- `helm_chart` can be found in the [release](https://github.com/schabrolles/cpd-install-pipeline/releases) section.
- `arch` *optional* value allows pipeline to be adapted for **non x86** cluster (only `ppc64le` or `s390x`) (`arch=""` means `x86` which is the default when not set)

example: 
```
helm install cpd-install cpd-install-pipeline/cpd-install-pipeline \
--create-namespace -n cpd-install --set arch=ppc64le
```
This will:
   - create a project `cpd-install`
   - give the **::“cluster-admin”::** right to the “**pipeline**” service-account of this project
   - create a tekton task “**olm-utils**” based on the official `olm-utils-v2` from IBM (icr.io)
   - create a tekton pipeline to install automatically the cloud pak for data `components` you choose

### 1bis - Adding helm repo to openshift to use Graphical Interface

apply the following yaml to add the helm repo in openshift.
```
apiVersion: helm.openshift.io/v1beta1
kind: HelmChartRepository
metadata:
  name: cpd-install-pipeline
spec:
  connectionConfig:
    url: 'https://schabrolles.github.io/cpd-install-pipeline'
```

- Then create a project: `cp-install`for example.
- Switch to **developer view** / **add**
- Select Helm charts and search for cp-install

<img width="1856" alt="image" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/b15b43fd-3d01-405e-a602-b61a87bab204">

- Click on **Create**

<img width="1491" alt="image" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/0cc8c07d-0602-473e-81a6-75d8f7144e56">

- Select the Chart version you want.
- **if you Run on Power or Z, set the arch value before validating**

## Starting the pipeline
* 2- [Using openshift pipeline graphical interface](#2--Starting-pipeline-with-GUI) (GUI based)
* 2bis- [applying pipelineRun yaml file](2bis--Starting-pipeline-from-cli-(by-applying-YAML)) (cli based)

### 2- Starting pipeline with GUI

>#### Note:
>* After having deploy the helm charts, verify that the `tekton-patch` pipeline ran successfuly.  
>* This `tekton-patch` pipeline update the `openshift-pipelines` config to increase the default value from 1h to 5h (cp4d installation duration is 2h minimum).  
>* If you want to update this parameter manually or if this pipeline failed you can change update this pipeline timeout parameter by using the following command:  
>```other
>oc set data -n openshift-pipelines cm/config-defaults default-timeout-minutes="300"
>```

From the openshift console, enter in the project `cpd-install` and select the tab “pipeline" from the menu on the left.

<img width="1666" alt="pipeline-gui1" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/0eea2ace-42d6-449f-96f7-4cc3d82b311d">

Click on the **cpd-install** pipeline.

On the next page, click on the action button on the right end side of the page and select “**start**”

<img width="1666" alt="pipeline-gui2" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/73c2f844-26e7-4022-8db3-d930ad539b23">

This will open a forms to customize your installation.

- You need to provide at least your **IBM Entitlement key**
- You can change the version, name of namespaces or storageclass to use
- [list of available component](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=information-determining-which-components-install#collect-info-components__all-svcs)
- If you are not running a production service, set **production** to **false**
- if you don’t have GPUs but still want to deploy watsonx, set **NO_GPU** to **true**

click on **Start**

The full installation duration is about ~2h

<img width="1380" alt="pipeline-gui3" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/e9e6a549-1eca-4b76-9faf-79c5e4d1840a">

### 2bis- Starting pipeline from cli (by applying YAML)

It is possible to apply a PipelineRun YAML file to start the pipeline and set your variables.
An example of this yaml file is availble in the NOTES of this helm Chart. it can be print with the `helm status <release_name>` command.

- Use `helm list` in your namespace to get the release name:
```
$ helm list
NAME                	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART                     	APP VERSION
cpd-install-pipeline	cpd-install	5       	2024-06-02 06:54:12.979372883 +0000 UTC	deployed	cpd-install-pipeline-1.4.3	1.4.3
```

- Run the `helm status ...`
```
helm status cpd-install-pipeline
NAME: cpd-install-pipeline
LAST DEPLOYED: Sun Jun  2 06:54:12 2024
NAMESPACE: cpd-install
STATUS: deployed
REVISION: 5
TEST SUITE: None
NOTES:
Your Pipeline to install CPD on your cluster has been deployed.
you can use the openshift console to start the installation graphically
or use "oc create -f" with a PipelineRun yaml file. (example bellow)

apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: cpd-install-pipeline-
  namespace: cpd-install
spec:
  params:
  - name: OLM_UTILS_IMAGE
    value: icr.io/cpopen/cpd/olm-utils-v2
  - name: VERSION
    value: 4.8.5
  - name: COMPONENTS
    value: wml,ws
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
    value: cpd-enterprise
  - name: SCHEDULER
    value: "false"
  - name: PRODUCTION
    value: "false"
  - name: ACCEPT_LICENCE
    value: "true"
  - name: NO_GPU
    value: "false"
  - name: APPLY_MACHINECONFIG
    values: "true"
  - name: DB2_LIMITED_PRIV
    values: "false"
  - name: CP-INSTALL-OPTIONS
    value: cpd-install-pipeline-install-option
  pipelineRef:
    name: cpd-install-pipeline
  taskRunTemplate:
    serviceAccountName: pipeline
  timeouts:
    pipeline: 5h0m0`
```

- Use this sample (starting from apiversion) to create a yaml file.

- Change the Parameters to fit to your needs:

  - You need to provide at least your **IBM Entitlement key** ([link to get your key](https://myibm.ibm.com/products-services/containerlibrary))
  - You can change the version, name of namespaces or storageclass to use
  - If you are not running a production service, set **production** to **false**
  - if you don’t have GPUs but still want to deploy watsonx, set **NO_GPU** to **true**


- You can use openshift console to follow the pipeline.
If you prefer you can download the `tkn` cli ([https://docs.openshift.com/container-platform/4.15/cli_reference/tkn_cli/installing-tkn.html#installing-tkn](https://docs.openshift.com/container-platform/4.15/cli_reference/tkn_cli/installing-tkn.html#installing-tkn))

```plaintext
tkn pr logs -n <pipeline_name> --follow
```
