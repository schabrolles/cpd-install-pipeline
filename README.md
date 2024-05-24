# cpd-install-pipeline
Install automatically IBM Cloudpak for Data using Tekton Pipeline

![Image.png](https://res.craft.do/user/full/de0964ce-f4e1-6349-286f-b59814cf4260/doc/48A286AD-0E40-4681-83FF-97A2BBB33870/6ECCE18D-E3D1-4CA0-9EBC-3BCBD1939795_2/DSzsCvKLIxcAhqBUZklhC8loEtizPU2oK30b5B8VqpIz/Image.png)

## Prerequisite:

- Openshift Cluster > 4.12
- Openshift Pipeline Operator
- **Cluster can access to registries icr.io and cp.icr.io**

### 1- Configuring cpd-install Tekton Pipeline

After having installed the openshift-pipeline operator, use helm to deploy the tasks and pipeline in a namespace of your cluster.

```
helm install <name> <helm_chart> --create-namespace -n <namespace> [--set arch=(ppc64le|s390x)]
```

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

![Image.png](https://res.craft.do/user/full/de0964ce-f4e1-6349-286f-b59814cf4260/doc/48A286AD-0E40-4681-83FF-97A2BBB33870/A0D3CEA8-62C6-4C94-A66E-C1A7A684BE68_2/EIgT0wFGXxxF9xPzi3RxnqHWwHAAiYikypdAiw7wrqEz/Image.png)

Click on the **cpd-install** pipeline.

On the next page, click on the action button on the right end side of the page and select “**start**”

![Image.png](https://res.craft.do/user/full/de0964ce-f4e1-6349-286f-b59814cf4260/doc/48A286AD-0E40-4681-83FF-97A2BBB33870/E66B8D89-30E7-4766-8A1F-5077313DF19E_2/Wy7CZB3zknYeEJPvqks0abuTAZSz5BcdC67zIa5yGeAz/Image.png)

This will open a forms to customize your installation.

- You need to provide at least your **IBM Entitlement key**
- You can change the version, name of namespaces or storageclass to use
- If you are not running a production service, set **production** to **false**
- if you don’t have GPUs but still want to deploy watsonx, set **NO_GPU** to **true**

click on **Start**

The full installation duration is about ~2h

![Image.png](https://res.craft.do/user/full/de0964ce-f4e1-6349-286f-b59814cf4260/doc/48A286AD-0E40-4681-83FF-97A2BBB33870/9FBEF9B2-DBE4-4E0E-8013-8CA93A6389C2_2/dXYxIkknDp4JKJTdlqVaVXroGnyPxoBO1tQEpz45n0kz/Image.png)

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
