# Unit Tests for cpd-install-pipeline Helm Chart

This directory contains unit tests for the cpd-install-pipeline Helm chart templates.

## Prerequisites

To run these tests, you need to install the Helm unittest plugin:

```bash
helm plugin install https://github.com/helm-unittest/helm-unittest
```

## Running Tests

### Run all tests
```bash
helm unittest charts/cpd-install-pipeline
```

### Run tests with verbose output
```bash
helm unittest -v charts/cpd-install-pipeline
```

### Run specific test file
```bash
helm unittest -f 'tests/cpd-install-pipeline_5.2.x_test.yaml' charts/cpd-install-pipeline
```

### Run tests with colored output
```bash
helm unittest --color charts/cpd-install-pipeline
```

## Test Coverage

The test suite for `cpd-install-pipeline_5.2.x.yaml` covers:

1. **Parameter Definitions**: Validates that all expected parameters are defined with correct default values
   - Standard CPD installation parameters
   - WatsonX-specific parameters
   - Storage class parameters
   - Airgap/disconnected installation parameters

2. **Task Execution Order**: Ensures tasks execute in the correct sequence with proper `runAfter` dependencies
   - Validates the pipeline workflow from initialization through completion
   - Checks proper dependency chains between tasks

3. **Conditional Logic**: Tests airgap enabled/disabled states
   - Pipeline naming (cpd-install vs cpd-install-airgap)
   - Parameter inclusion/exclusion based on airgap mode
   - Private registry vs IBM Entitlement Key usage

4. **Version Compatibility**: Validates semver comparison logic
   - Tests that pipeline renders for 5.2.x versions
   - Tests that pipeline does NOT render for other versions (5.1.x, 5.3.x)

5. **Task Configuration**: Verifies retry and timeout values
   - Retry values for kubelet configuration tasks
   - Timeout values for long-running installation tasks

6. **Conditional Task Execution**: Tests `when` clauses for conditional tasks
   - APPLY_MACHINECONFIG parameter
   - DB2_LIMITED_PRIV parameter
   - SCHEDULER parameter
   - NO_GPU parameter

7. **Proxy Configuration**: Tests proxy-related defaults
   - CASE_FROM_OCI parameter based on proxy settings

8. **Architecture Support**: Tests architecture-specific image tags
   - x86 (no suffix)
   - ppc64le (with .ppc64le suffix)
   - s390x support

## Test Structure

Each test file follows this structure:

```yaml
suite: Test <template-name>
templates:
  - templates/<template-file>.yaml
tests:
  - it: should <expected behavior>
    set:
      # Values to override
    asserts:
      # Assertions to validate
```

## Adding New Tests

When adding new tests:

1. Create a new test file in the `tests/` directory following the naming convention: `<template-name>_test.yaml`
2. Define the suite name and template path
3. Add test cases with descriptive names using `it: should <expected behavior>`
4. Use appropriate assertions from the [helm-unittest documentation](https://github.com/helm-unittest/helm-unittest/blob/main/DOCUMENT.md)

## Continuous Integration

These tests should be run as part of the CI/CD pipeline before merging changes to ensure template integrity.
