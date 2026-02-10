# README Improvement Plan for cpd-install-pipeline Chart

## Overview
This document outlines the comprehensive improvements needed to bring the chart's README.md up to date with the main project README and enhance its overall quality.

## Key Issues Identified

### 1. Formatting Inconsistencies
- **Capitalization**: Mixed use of "Openshift" vs "OpenShift", "cli" vs "CLI"
- **Code blocks**: Missing language tags (yaml, bash, plaintext)
- **Headings**: Inconsistent spacing and formatting
- **Links**: Some broken anchor references in Table of Contents

### 2. Outdated Information
- Version references need updating to 5.2.2 (currently shows 4.8.5 in examples)
- Missing latest chart version reference
- Outdated parameter examples in PipelineRun YAML

### 3. Missing Content Sections
The chart README is missing several important sections present in the main README:

#### Advanced Features Section
- Airgap/Disconnected Installation instructions
- Proxy Configuration details
- WatsonX Mode configuration

#### Available Pipelines Section
- List of all available pipelines:
  - cpd-install
  - cpd-upgrade
  - cpd-add-components
  - cpd-remove-components
  - cpd-check
  - cpd-restart
  - cpd-shutdown
  - cpd-delete-instance
  - cpd-inject-custom-certs
  - cpd-ctrl-center-install

#### Troubleshooting Section
- Pipeline timeout issues
- Check pipeline status commands
- View logs instructions

#### Support Section
- GitHub Issues link
- IBM Documentation link

#### License Section
- Maintainer information

### 4. Parameter Documentation
The chart README needs enhanced parameter descriptions:
- Add STORAGE_TESTS parameter
- Update OCI_LOCATION default value (cp.icr.io/cpopen vs icr.io/cpopen)
- Add clearer descriptions for each parameter
- Group parameters by category (Required vs Optional)

### 5. Installation Instructions
- Add note about latest chart version (5.2.2)
- Include cpd_version parameter in helm install examples
- Improve formatting of helm commands

## Detailed Changes by Section

### Prerequisites (Lines 7-11)
**Current Issues:**
- "Openshift" should be "OpenShift"
- Version format inconsistent ("> 4.12" vs ">= 4.12")

**Improvements:**
```markdown
- OpenShift Cluster >= 4.12
- OpenShift Pipeline Operator
- **Cluster can access registries icr.io and cp.icr.io** (for connected installations)
```

### Table of Contents (Lines 13-22)
**Current Issues:**
- Broken link on line 21 (missing #)
- Inconsistent formatting

**Improvements:**
- Fix all anchor links
- Ensure consistent formatting
- Add new sections (Advanced Features, Available Pipelines, Troubleshooting)

### Installation Section (Lines 23-79)
**Current Issues:**
- "acces" typo (line 25)
- "mod" should be "mode" (line 26)
- Missing cpd_version parameter
- Code blocks missing language tags

**Improvements:**
- Fix typos
- Add language tags to all code blocks
- Include cpd_version parameter in examples
- Update to version 5.2.2
- Add note about latest version

### Starting Pipeline Section (Lines 81-218)
**Current Issues:**
- Version 4.8.5 in examples (should be 5.2.2)
- Missing STORAGE_TESTS parameter
- Incorrect OCI_LOCATION value
- Code blocks missing language tags
- Inconsistent parameter descriptions

**Improvements:**
- Update all version references to 5.2.2
- Add comprehensive parameter descriptions
- Group parameters (Required vs Optional)
- Add language tags to code blocks
- Update OCI_LOCATION default
- Add STORAGE_TESTS parameter

### New Sections to Add

#### Advanced Features (After line 218)
Add complete section with:
- Airgap/Disconnected Installation
- Proxy Configuration
- WatsonX Mode

#### Available Pipelines (After Advanced Features)
List all pipelines with brief descriptions

#### Troubleshooting (After Available Pipelines)
Add common issues and solutions:
- Pipeline timeout
- Check status commands
- View logs

#### Support (After Troubleshooting)
Add links to:
- GitHub Issues
- IBM Documentation

#### License (After Support)
Add maintainer information

## Implementation Priority

1. **High Priority** (Critical for usability)
   - Fix broken links
   - Update version numbers to 5.2.2
   - Add language tags to code blocks
   - Fix typos and capitalization

2. **Medium Priority** (Important for completeness)
   - Add Advanced Features section
   - Add Available Pipelines section
   - Add Troubleshooting section
   - Update parameter descriptions

3. **Low Priority** (Nice to have)
   - Add Support section
   - Add License section
   - Improve formatting consistency

## Expected Outcome

After implementing these improvements, the chart README will:
- Be fully synchronized with the main project README
- Have consistent formatting and styling
- Include all necessary sections for users
- Provide comprehensive documentation
- Be up-to-date with version 5.2.2
- Have working links and proper code highlighting
- Follow best practices for technical documentation

## Validation Checklist

- [ ] All links work correctly
- [ ] All code blocks have language tags
- [ ] Version numbers are consistent (5.2.2)
- [ ] Terminology is consistent (OpenShift, CLI)
- [ ] All sections from main README are included
- [ ] Parameter descriptions are complete
- [ ] Examples are up-to-date
- [ ] No typos or grammatical errors
- [ ] Table of contents matches actual sections