# Checkmarx One++ GitHub Action

This action internally uses the same Checkmarx One CLI as is used by the [Checkmarx AST GitHub Action](https://github.com/marketplace/actions/checkmarx-ast-github-action).  There are a few significant differences with this action:

* The workflow supports both PR decorations with scan summaries and Sarif file uploads to for use with GitHub security.
* Execution is performed in your build environment container created with the [Checkmarx Supply Chain Toolkit](https://github.com/checkmarx-ts/cx-supply-chain-toolkit).
* Supply chain scanning is performed in the build environment container.


# Quick Start

## Prerequisites

For the Quick Start, the following prerequisites apply:

* You have created a build environment container using the [Checkmarx Supply Chain Toolkit](https://github.com/checkmarx-ts/cx-supply-chain-toolkit) and the runner can retrieve that container with the tag you
provide as an input.
* You have created an [OAuth client](https://checkmarx.com/resource/documents/en/34965-68612-creating-oauth-clients.html) in Checkmarx One.
* You define the secret `CXONE_TENANT` that contains the name of your Checkmarx One tenant.
* You define the secret `CXONE_CLIENT_ID` that contains the client ID of the created OAuth client.
* You define the secret `CXONE_CLIENT_SECRET` that contains the client secret of the created OAuth client.

## Example Workflow

If your tenant is in the US1 environment, the following example workflow will perform a scan on pull requests or pushes targeting the `master` branch:


```yaml
name: Checkmarx Scan
on:
  workflow_dispatch:
  push:
    branches:
      - master
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
    branches:
      - master
jobs:
  checkmarx-scan:
    runs-on: ubuntu-latest

    steps:
    - name: Code Checkout
      uses: actions/checkout@v4
       
    - name: CxOne Scan
      id: cxscan
      uses: checkmarx-ts/cxone-plusplus-github-action@v1
      with:
        container-image: <<your image tag goes here>>
        cx-tenant: ${{ secrets.CXONE_TENANT }}
        cx-client-id: ${{ secrets.CXONE_CLIENT_ID }}
        cx-client-secret: ${{ secrets.CXONE_CLIENT_SECRET }}

    - name: Show outputs
      shell: bash
      run: |
        echo ScanID: ${{ steps.cxscan.outputs.scan-id}}
        echo ProjectID: ${{ steps.cxscan.outputs.project-id}}
```

If you are not in the US1 environment, you must provide inputs
`base-uri` and `base-auth-uri` to
define the API and authentication endpoints properly.


# Additional Configuration Options


## Checkmarx Reference Documentation

This action is an integration that uses the 
[Checkmarx One CLI](https://checkmarx.com/resource/documents/en/34965-68621-checkmarx-one-cli-quick-start-guide.html) and the 
[Checkmarx SCA Resolver](https://checkmarx.com/resource/documents/en/34965-19196-checkmarx-sca-resolver.html).  The inputs to this action translate
to command line options found in the documentation.  Please refer to this
documentation if you need to include additional configuration not explicitly
available through an input.

There are a number of options for both the CxOne CLI and SCA Resolver that can
be provided as environment variables.  Using GitHub's ability to define an
environment for the repository may also be used to inject configuration settings
into the execution of the CxOne CLI and SCA Resolver.


## Inputs

 ### `base-uri`

 The base URI for the region endpoint where your [tenant is located]( https://checkmarx.com/resource/documents/en/34965-68630-configure.html#UUID-494287f8-3019-2542-c111-cb147c286bac_id_ASTCLIConfigurationEnvironmentVariables-CLIConfigurationParameters).

Default: URI for US1 environment

###  `base-auth-uri`

 The base authentication URI for the region endpoint where your [tenant is located]( https://checkmarx.com/resource/documents/en/34965-68630-configure.html#UUID-494287f8-3019-2542-c111-cb147c286bac_id_ASTCLIConfigurationEnvironmentVariables-CLIConfigurationParameters).

Default: URI for US1 environment


### `cx-tenant` (required)
**It is advised to store this as a secret value.**

The tenant identifier for your tenant.


### `cx-client-id` (required)
**It is advised to store this as a secret value.**

The OAuth Client ID for API access.

### `cx-client-secret` (required)
**It is advised to store this as a secret value.**

The OAuth client secret key for API access.

### `container-image` (required)
The container tag that was made with the cx-supply-chain-toolkit.


### `project-name`

The name of the project where the scan will be executed.

Default: The name of the repository.

### `additional-scan-params`

Additional parameters passed to the CxOne CLI after `scan create`. 

### `cx-cli-debug`
If true, turn on debugging for the CxOne CLI and the composite action.

Default: false

### `cx-cli-agent`
The agent name to use when performing CxOne CLI commands.  This value will show as the "Scan Origin" of each scan.

Default: `cxonepp-gh-action`

### `cx-cli-additional-params`

Additional parameters to pass to the CxOne CLI (proxy settings, etc) for all CxOne CLI invocations.



### `docker-login-registry`
The name of the container registry to use for login.

Default: `docker.io`

### `docker-login-username`
**It is advised to store this as a secret value.**

The username for the container registry login.
  
  
  
### `docker-login-password`
**It is advised to store this as a secret value.**

The password for the container registry login.

### `upload-sarif-file`
If true, uploads the Sarif file to create entries on the GitHub security tab during push events. 

Default: true


### `attach-sarif-file`
If true, attaches the Sarif file to the workflow as an artifact for push or pull request events.

Default: false

### `additional-report-params`
    
Additional parameters used when compiling reports, as described in the CxOne
CLI `results show` command. Do not add the `--filter` option; use the `report-filters` input parameter to set filter options.

### `report-filters`
The criteria that selects what to include in any report results.

Default: `--filter "state=TO_VERIFY;PROPOSED_NOT_EXPLOITABLE;CONFIRMED;URGENT"`

### `attach-sbom-file`
If true, attaches the SBOM file to the workflow as an artifact. 

Default: false

### `sbom-standard`
The SBOM standard format used when generating the SBOM file. 

Default: `CycloneDxJson`

### `show-versions`
If true, emits the versions of the CxOne CLI and SCA Resolver in the action log.

Default: true

### `container-run-command`
The command used to execute the container.

Default: 'docker run'

### `container-run-command-params`
The parameters used when executing the container.

Default: `"-t --rm -v ${GITHUB_WORKSPACE}:/sandbox/input:ro -v $(realpath ${GITHUB_WORKSPACE}/../output):/sandbox/output"`


## Outputs

### `scan-id`

The GUID identifier for the scan that was performed by the action.

### `project-id`

The GUID identifier for the project where the scan information can be found.

### `project-deeplink`

A deep link to the project overview for the project where the scan was performed.