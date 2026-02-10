# cpd-install-pipeline

Install automatically IBM Cloud Pak for Data using Tekton Pipeline

<img width="1708" alt="image" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/102ca157-3dce-4930-9c26-ce2058639eef">

## Prerequisite:

- OpenShift Cluster >= 4.12
- OpenShift Pipeline Operator
- **Cluster can access registries icr.io and cp.icr.io** (for connected installations)

## Table of Content:

- **Installation**
   - GUI based: [Using OpenShift web-console](#1--adding-helm-repo-to-openshift-to-use-graphical-interface)
   - CLI based: [Using helm CLI](#1bis--configuring-cpd-install-tekton-pipeline-from-the-cli)

- **Starting Pipeline**
   - GUI based: [Using OpenShift pipeline graphical interface](#2--starting-pipeline-with-gui)
   - CLI based: [Applying pipelineRun yaml file](#2bis--starting-pipeline-from-cli-by-applying-yaml)

- **Advanced Features**
   - [Airgap/Disconnected Installation](#airgap--disconnected-installation)
   - [Proxy Configuration](#proxy-configuration)
   - [WatsonX Mode](#watsonx-mode)

## Installation

The easiest way to install the pipeline is to add this helm registry to your OpenShift cluster.  
**You must have access to internet for this.**  
For airgap mode, please use the helm CLI and download the archive from [release page](https://github.com/schabrolles/cpd-install-pipeline/releases)

### 1- Adding helm repo to OpenShift to use Graphical Interface

Apply the following yaml to add the helm repo in OpenShift (with `oc apply` or on the OpenShift console by clicking on the "**+**" button on the black banner at the top right-end corner of the screen):

```yaml
apiVersion: helm.openshift.io/v1beta1
kind: HelmChartRepository
metadata:
  name: cpd-install-pipeline
spec:
  connectionConfig:
    url: 'https://schabrolles.github.io/cpd-install-pipeline'
```

- Then create a project: `cpd-install` for example.
- Switch to **developer view** / **add**
- Select Helm charts and search for cpd-install

<img width="1856" alt="image" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/b15b43fd-3d01-405e-a602-b61a87bab204">

- Click on **Create**

<img width="1491" alt="image" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/0cc8c07d-0602-473e-81a6-75d8f7144e56">

- Select the Chart version you want (latest: **5.2.2**)
- **If you run on Power or Z, set the arch value before validating** (`ppc64le` or `s390x`)

### 1bis- Configuring cpd-install Tekton Pipeline from the CLI

After having installed the OpenShift Pipeline operator, use helm to deploy the tasks and pipeline in a namespace of your cluster.

- Adding the helm repo:
```bash
helm repo add cpd-install-pipeline https://schabrolles.github.io/cpd-install-pipeline
```

- Deploying the pipeline configuration with helm-cli:
```bash
helm install <name> cpd-install-pipeline/cpd-install-pipeline \
  --create-namespace -n <namespace> \
  [--set arch=(ppc64le|s390x)] \
  [--set cpd_version=5.2.2]
```

**Parameters:**
- `arch` *optional* - Allows pipeline to be adapted for **non x86** cluster (only `ppc64le` or `s390x`). Default is `x86` when not set.
- `cpd_version` *optional* - Cloud Pak for Data version to install. Default is `5.2.2`.

**Example:**
```bash
helm install cpd-install cpd-install-pipeline/cpd-install-pipeline \
  --create-namespace -n cpd-install --set arch=ppc64le
```

This will:
- Create a project `cpd-install`
- Give the **cluster-admin** right to the "**pipeline**" service-account of this project
- Create a Tekton task "**olm-utils**" based on the official `olm-utils-v3` from IBM (icr.io)
- Create Tekton pipelines to install, upgrade, and manage Cloud Pak for Data components

## Starting the pipeline

* [Using OpenShift pipeline graphical interface](#2--starting-pipeline-with-gui) (GUI based)
* [Applying pipelineRun yaml file](#2bis--starting-pipeline-from-cli-by-applying-yaml) (CLI based)

### 2- Starting pipeline with GUI

>#### Note:
>* After having deployed the helm charts, verify that the `tekton-patch` pipeline ran successfully.  
>* This `tekton-patch` pipeline updates the `openshift-pipelines` config to increase the default value from 1h to 5h (CP4D installation duration is 2h minimum).  
>* If you want to update this parameter manually or if this pipeline failed, you can update this pipeline timeout parameter by using the following command:  
>```bash
>oc set data -n openshift-pipelines cm/config-defaults default-timeout-minutes="300"
>```

From the OpenShift console, enter the project `cpd-install` and select the tab "**Pipelines**" from the menu on the left.

<img width="1666" alt="pipeline-gui1" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/0eea2ace-42d6-449f-96f7-4cc3d82b311d">

Click on the **cpd-install** pipeline.

On the next page, click on the action button on the right end side of the page and select "**Start**"

<img width="1666" alt="pipeline-gui2" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/73c2f844-26e7-4022-8db3-d930ad539b23">

This will open a form to customize your installation.

**Required Parameters:**
- **IBM_ENTITLEMENT_KEY**: Your IBM Entitlement key ([Get your key here](https://myibm.ibm.com/products-services/containerlibrary))

**Optional Parameters:**
- **VERSION**: Cloud Pak for Data version (default: 5.2.2)
- **COMPONENTS**: Comma-separated list of components to install (e.g., `wml,ws,watsonx_ai`)
  - [List of available components](https://www.ibm.com/docs/en/software-hub/5.2.x?topic=information-determining-which-components-install)
- **PROJECT_CPD_INST_OPERANDS**: Namespace for Cloud Pak operands (default: `cpd`)
- **PROJECT_CPD_INST_OPERATORS**: Namespace for Cloud Pak operators (default: `cpd-operators`)
- **STG_CLASS_BLOCK**: Storage class for block storage (default: `ocs-storagecluster-ceph-rbd`)
- **STG_CLASS_FILE**: Storage class for file storage (default: `ocs-storagecluster-cephfs`)
- **STORAGE_TESTS**: Run storage performance validation tests (default: `false`)
- **ENTITLEMENT**: License entitlement type (default: `cpd-enterprise`)
- **PRODUCTION**: Set to `false` for non-production environments (default: `true`)
- **NO_GPU**: Set to `true` to deploy WatsonX without GPU support (default: `false`)
- **DB2_LIMITED_PRIV**: Run DB2 pods in limited privilege mode (default: `false`)
- **SCHEDULER**: Enable IBM scheduler (default: `false`)
- **APPLY_MACHINECONFIG**: Apply KubeletConfig for Cloud Pak for Data (default: `true`)
- **CASE_FROM_OCI**: Download CASE packages from OCI registry (default: `true`)
- **OCI_LOCATION**: OCI registry location for CASE packages (default: `cp.icr.io/cpopen`)

Click on **Start**

The full installation duration is approximately **2-4 hours** depending on the components selected.

<img width="1380" alt="pipeline-gui3" src="https://github.com/schabrolles/cpd-install-pipeline/assets/19491077/e9e6a549-1eca-4b76-9faf-79c5e4d1840a">

### 2bis- Starting pipeline from CLI (by applying YAML)

It is possible to apply a PipelineRun YAML file to start the pipeline and set your variables.
An example of this yaml file is available in the NOTES of this helm Chart. It can be printed with the `helm status <release_name>` command.

- Use `helm list` in your namespace to get the release name:
```bash
$ helm list -n cpd-install
NAME                	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART                     	APP VERSION
cpd-install-pipeline	cpd-install	1       	2024-06-02 06:54:12.979372883 +0000 UTC	deployed	cpd-install-pipeline-5.2.2	5.2.2
```

- Run the `helm status` command:
```bash
helm status cpd-install-pipeline -n cpd-install
```

- Use the sample YAML (starting from `apiVersion`) to create a yaml file.

**Example PipelineRun:**

```yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: cpd-install-
  namespace: cpd-install
spec:
  params:
  - name: OLM_UTILS_IMAGE
    value: icr.io/cpopen/cpd/olm-utils-v3
  - name: VERSION
    value: 5.2.2
  - name: COMPONENTS
    value: wml,ws
  - name: PROJECT_CPD_INST_OPERANDS
    value: cpd
  - name: PROJECT_CPD_INST_OPERATORS
    value: cpd-operators
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
  - name: STORAGE_TESTS
    value: "false"
  - name: ENTITLEMENT
    value: cpd-enterprise
  - name: SCHEDULER
    value: "false"
  - name: PRODUCTION
    value: "false"
  - name: ACCEPT_LICENCE
    value: "true"
  - name: APPLY_MACHINECONFIG
    value: "true"
  - name: DB2_LIMITED_PRIV
    value: "false"
  - name: CASE_FROM_OCI
    value: "true"
  - name: OCI_LOCATION
    value: "cp.icr.io/cpopen"
  - name: NO_GPU
    value: "false"
  - name: CP-INSTALL-OPTIONS
    value: cpd-install-install-option
  pipelineRef:
    name: cpd-install
  taskRunTemplate:
    serviceAccountName: pipeline
  timeouts:
    pipeline: 5h0m0s
```

- Apply the YAML file:
```bash
oc create -f cpd-install-pipelinerun.yaml
```

- Monitor the pipeline execution:

**Using OpenShift Console:**
Navigate to Pipelines → PipelineRuns in your project

**Using tkn CLI:**
Download the `tkn` CLI ([Installation guide](https://docs.openshift.com/container-platform/4.15/cli_reference/tkn_cli/installing-tkn.html))

```bash
tkn pr logs -n cpd-install --follow
```

## Advanced Features

### Airgap / Disconnected Installation

For disconnected or airgap environments, you need to:

1. **Mirror images to your private registry** following [IBM's documentation](https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-mirroring-images-your-private-container-registry)

2. **Create a docker registry secret** in your installation namespace:
```bash
oc create secret docker-registry private-registry-secret \
  -n cpd-install \
  --docker-username=<USERNAME> \
  --docker-password=<PASSWORD> \
  --docker-server=<REGISTRY-HOSTNAME:PORT>
```

3. **Install the helm chart with airgap enabled:**
```bash
helm install cpd-install cpd-install-pipeline/cpd-install-pipeline \
  --create-namespace -n cpd-install \
  --set airgap.enabled=true \
  --set airgap.private_registry_secret=private-registry-secret
```

4. **Start the airgap pipeline** (`cpd-install-airgap`) instead of the regular `cpd-install` pipeline

### Proxy Configuration

For environments requiring proxy configuration:

```bash
helm install cpd-install cpd-install-pipeline/cpd-install-pipeline \
  --create-namespace -n cpd-install \
  --set proxy.enabled=true \
  --set proxy.http_proxy="http://user:password@proxy-ip:port" \
  --set proxy.https_proxy="http://user:password@proxy-ip:port" \
  --set proxy.no_proxy="localhost,127.0.0.1,.svc,.cluster.local"
```

### WatsonX Mode

To install WatsonX components with optimized defaults:

```bash
helm install cpd-install cpd-install-pipeline/cpd-install-pipeline \
  --create-namespace -n cpd-install \
  --set default_cpd_install=watsonx
```

This will set default values for:
- Components: `watsonx_ai,watsonx_governance,watsonx_data`
- Operands namespace: `watsonx`
- Operators namespace: `watsonx-operators`
- Entitlement: `watsonx-ai,watsonx-data,watsonx-gov-mm,watsonx-gov-rc`

## Available Pipelines

The helm chart deploys multiple pipelines for different operations:

- **cpd-install**: Main installation pipeline
- **cpd-upgrade**: Upgrade existing Cloud Pak for Data installation
- **cpd-add-components**: Add new components to existing installation
- **cpd-remove-components**: Remove components from installation
- **cpd-check**: Check installation status
- **cpd-restart**: Restart Cloud Pak for Data services
- **cpd-shutdown**: Shutdown Cloud Pak for Data services
- **cpd-delete-instance**: Delete Cloud Pak for Data instance
- **cpd-inject-custom-certs**: Inject custom certificates
- **cpd-ctrl-center-install**: Install Control Center

## Troubleshooting

### Pipeline Timeout
If your pipeline times out, increase the timeout value:
```bash
oc set data -n openshift-pipelines cm/config-defaults default-timeout-minutes="300"
```

### Check Pipeline Status
```bash
tkn pr list -n cpd-install
tkn pr describe <pipelinerun-name> -n cpd-install
```

### View Logs
```bash
tkn pr logs <pipelinerun-name> -n cpd-install --follow
```

## Support

For issues and questions:
- GitHub Issues: [https://github.com/schabrolles/cpd-install-pipeline/issues](https://github.com/schabrolles/cpd-install-pipeline/issues)
- IBM Documentation: [https://www.ibm.com/docs/en/software-hub/5.2.x](https://www.ibm.com/docs/en/software-hub/5.2.x)

## License

This project is maintained by Sebastien Chabrolles (s.chabrolles@fr.ibm.com)