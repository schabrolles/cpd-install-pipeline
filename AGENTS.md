# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Project Overview

**cpd-install-pipeline** is a Helm chart that automates the deployment of IBM Cloud Pak for Data (CPD) on OpenShift clusters using Tekton Pipelines. It provides a declarative, pipeline-based approach to installing, upgrading, and managing Cloud Pak for Data components.

### Key Technologies
- **Helm 3**: Chart packaging and templating
- **Tekton Pipelines**: CI/CD pipeline execution on Kubernetes
- **OpenShift**: Target platform (4.12+)
- **IBM Cloud Pak for Data**: Enterprise AI and data platform (versions 5.0.x - 5.2.x)

### Architecture

The project follows a template-driven architecture:

```
charts/cpd-install-pipeline/
├── Chart.yaml                    # Helm chart metadata
├── values.yaml                   # Default configuration values
├── values.schema.json            # JSON schema for values validation
└── templates/                    # Helm templates
    ├── _helpers.tpl              # Template helper functions
    ├── olm-utilstask.yaml        # Reusable Tekton task for OLM operations
    ├── cpd-install-pipeline_*.yaml  # Version-specific install pipelines
    ├── cpd-upgrade-pipeline_*.yaml  # Version-specific upgrade pipelines
    ├── cpd-*-pipeline.yaml       # Management pipelines (restart, shutdown, etc.)
    ├── install-option-cm_*.yaml  # ConfigMaps for install options
    ├── clusterRoleBinding.yaml   # RBAC configuration
    └── tekton-patch-*.yaml       # Pipeline configuration patches
```

### Multi-Version Support

The chart supports multiple CPD versions through conditional template rendering:
- **5.0.x and earlier**: `cpd-install-pipeline_before_5.1.yaml`
- **5.1.x**: `cpd-install-pipeline_5.1.x.yaml`
- **5.2.x**: `cpd-install-pipeline_5.2.x.yaml` (current default)

Version selection is controlled by the `cpd_version` value using Helm's `semverCompare` function.

### Key Components

1. **olm-utils Task**: Core Tekton task that wraps IBM's `olm-utils-v3` container image for executing CPD operations
2. **Install Pipelines**: Multi-stage pipelines for fresh CPD installations
3. **Upgrade Pipelines**: Version-aware upgrade workflows
4. **Management Pipelines**: Operational tasks (restart, shutdown, component management)
5. **Configuration Templates**: Dynamic generation of install options and parameters

## Building and Deployment

### Prerequisites
- OpenShift cluster (4.12+)
- OpenShift Pipelines Operator installed
- Helm 3.x CLI
- Access to `icr.io` and `cp.icr.io` registries (for connected installations)

### Local Development

**Linting the chart:**
```bash
helm lint charts/cpd-install-pipeline
```

**Template rendering (dry-run):**
```bash
helm template cpd-install charts/cpd-install-pipeline \
  --namespace cpd-install \
  --set arch=ppc64le \
  --set cpd_version=5.2.2
```

**Installing locally:**
```bash
helm install cpd-install charts/cpd-install-pipeline \
  --create-namespace -n cpd-install \
  --set arch=x86 \
  --set cpd_version=5.2.2
```

### Release Process

The project uses GitHub Actions for automated releases:
- **Trigger**: Push to `main` branch
- **Action**: `helm/chart-releaser-action@v1.6.0`
- **Output**: GitHub release with packaged chart and updated Helm repository index

The chart is published to GitHub Pages at: `https://schabrolles.github.io/cpd-install-pipeline`

### Testing Changes

1. **Template validation**: Always render templates after changes to verify syntax
2. **Schema validation**: Ensure `values.schema.json` matches `values.yaml` structure
3. **Version compatibility**: Test with different `cpd_version` values to verify conditional logic
4. **Architecture variants**: Test with `arch` set to `x86`, `ppc64le`, and `s390x`

## Development Conventions

### Helm Templating Patterns

**Conditional rendering based on CPD version:**
```yaml
{{- if semverCompare "~5.2.0" .Values.cpd_version }}
# 5.2.x specific content
{{- end }}
```

**Architecture-specific image tags:**
```yaml
value: $(params.OLM_UTILS_IMAGE):$(params.VERSION){{- if ne .Values.arch "x86" -}}.{{ .Values.arch }}{{- end }}
```

**Mode-specific defaults (CPD vs WatsonX):**
```yaml
{{ if eq .Values.default_cpd_install "watsonx" }}
- default: watsonx_ai,watsonx_governance,watsonx_data
{{ else }}
- default: wml,ws
{{ end }}
```

### Configuration Management

**Values hierarchy:**
1. `values.yaml`: Default values for all installations
2. `values.schema.json`: Validation rules and UI hints (enum support)
3. User overrides: Via `--set` flags or custom values files

**Key configuration patterns:**
- **Feature flags**: `enabled` boolean for optional features (proxy, airgap, tektonPatch)
- **Namespace conventions**: Separate namespaces for operators and operands
- **Storage classes**: Configurable block and file storage classes
- **Secrets management**: Docker registry secrets for airgap installations

### Tekton Pipeline Design

**Task structure:**
- All tasks use the `olm-utils` task with different `OLM_CMD` parameters
- Commands are passed as multi-line strings using YAML literal block scalars (`|` or `|-`)
- Tasks run sequentially using `runAfter` dependencies
- Conditional execution via `when` clauses

**Parameter naming:**
- Uppercase with underscores: `PROJECT_CPD_INST_OPERANDS`
- Descriptive names matching IBM documentation
- Default values aligned with common deployment scenarios

**Timeout management:**
- Default pipeline timeout: 5 hours (configurable via `tektonPatch.defaultTimeoutMinutes`)
- Task-specific timeouts for long-running operations (e.g., `components-apply-cr: 4h0m0s`)

### RBAC and Security

- Pipeline service account requires `cluster-admin` role (created via `clusterRoleBinding.yaml`)
- Secrets are mounted as volumes in tasks (not exposed as environment variables)
- Proxy credentials embedded in values (consider using secrets for production)

## Important Notes for Agents

### When Modifying Pipelines

1. **Version-specific changes**: Ensure changes are applied to the correct pipeline template(s)
2. **Backward compatibility**: Test with older CPD versions if modifying shared components
3. **Parameter propagation**: Verify all pipeline parameters are properly passed to tasks
4. **Conditional logic**: Use `when` clauses for optional steps, not shell conditionals when possible

### When Adding Features

1. **Values schema**: Update `values.schema.json` with new configuration options
2. **Documentation**: Update README.md with new parameters and usage examples
3. **Defaults**: Provide sensible defaults in `values.yaml`
4. **Validation**: Add enum constraints in schema for fixed-choice parameters

### Common Pitfalls

- **Image tags**: Always include architecture suffix for non-x86 platforms
- **Namespace references**: Use parameter values, not hardcoded namespaces
- **Secret handling**: Mount secrets as volumes, not environment variables
- **Timeout values**: Account for slow storage or large component installations
- **MachineConfig**: Some operations require MachineConfig support (not available on all platforms)

### Platform-Specific Considerations

**IBM Cloud ROKS:**
- No MachineConfig support
- Requires special kubelet configuration workarounds
- Docker config location: `/.docker/config.json`

**KubeVirt:**
- No MachineConfig support
- Docker config location: `/var/lib/kubelet/config.json`

**Standard OpenShift:**
- Full MachineConfig support
- Standard kubelet configuration paths

### Debugging Tips

1. **Pipeline failures**: Check task logs using `tkn pr logs <pipelinerun-name> -n cpd-install --follow`
2. **Template issues**: Use `helm template` with `--debug` flag
3. **Value validation**: Check against `values.schema.json` constraints
4. **OLM operations**: Examine the `olm-utils` container logs for detailed error messages

## Related Documentation

- [IBM Cloud Pak for Data Documentation](https://www.ibm.com/docs/en/software-hub/5.2.x)
- [Tekton Pipelines Documentation](https://tekton.dev/docs/pipelines/)
- [Helm Chart Development Guide](https://helm.sh/docs/chart_template_guide/)
- [OpenShift Pipelines Documentation](https://docs.openshift.com/container-platform/latest/cicd/pipelines/understanding-openshift-pipelines.html)
