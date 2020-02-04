# Contributing

This document provides guidelines for contributing to the module. This module is
slightly different than other Cloud Foundation Toolkit modules, and we have
tried to highlight the differences in the discussion below.

## Dependencies

The following dependencies must be installed on the development system:

- [Docker Engine][docker-engine]
- [Google Cloud SDK][google-cloud-sdk]
- [make]

## Generating Documentation for Inputs and Outputs

The Inputs and Outputs tables in the READMEs of the root module,
submodules, and example modules are automatically generated based on
the `variables` and `outputs` of the respective modules. These tables
must be refreshed if the module interfaces are changed.

### Execution

Run `make generate_docs` to generate new Inputs and Outputs tables.

## Integration Testing

Integration tests are used to verify the behaviour of the root module,
submodules, and example modules. Additions, changes, and fixes should
be accompanied with tests.

The integration tests are run using [Kitchen][kitchen],
[Kitchen-Terraform][kitchen-terraform], and [InSpec][inspec]. These
tools are packaged within a Docker image for convenience.

The general strategy for these tests is to verify the behaviour of the
[example modules](./examples/), thus ensuring that the root module,
submodules, and example modules are all functionally correct.

### Test Environment
The easiest way to test the module is in an isolated test project. The setup for such a project is defined in [test/setup](./test/setup/) directory.

To use this setup, you need a service account with these permissions (on a Folder or Organization):
- Project Creator
- Project Billing Manager

The project that the service account belongs to must have the following APIs enabled (the setup won't
create any resources on the service account's project):
- Cloud Resource Manager
- Cloud Billing
- Service Usage
- Identity and Access Management (IAM)

Export the Service Account credentials to your environment like so:

```
export SERVICE_ACCOUNT_JSON=$(< credentials.json)
```

You will also need to set a few environment variables:
```
export TF_VAR_org_id="your_org_id"
export TF_VAR_folder_id="your_folder_id"
export TF_VAR_billing_account="your_billing_account_id"
export TF_VAR_domain="domain.for.your.org.id"
```
Note that the domain variable is one that is not required for other Cloud
Foundation Toolkit modules.

With these settings in place, you can prepare a test project using Docker:
```
make docker_test_prepare
```

### Noninteractive Execution

Run `make docker_test_integration` to test all of the example modules
noninteractively, using the prepared test project.

### Interactive Execution

1. Run `make docker_run` to start the testing Docker container in
   interactive mode.

1. Run `kitchen_do create <EXAMPLE_NAME>` to initialize the working
   directory for an example module.

1. Run `kitchen_do converge <EXAMPLE_NAME>` to apply the example module.

1. Run `kitchen_do verify <EXAMPLE_NAME>` to test the example module.

1. Run `kitchen_do destroy <EXAMPLE_NAME>` to destroy the example module
   state.

## Linting and Formatting

Many of the files in the repository can be linted or formatted to
maintain a standard of quality.

### Execution

Run `make docker_test_lint`.

[docker-engine]: https://www.docker.com/products/docker-engine
[flake8]: http://flake8.pycqa.org/en/latest/
[gofmt]: https://golang.org/cmd/gofmt/
[google-cloud-sdk]: https://cloud.google.com/sdk/install
[hadolint]: https://github.com/hadolint/hadolint
[inspec]: https://inspec.io/
[kitchen-terraform]: https://github.com/newcontext-oss/kitchen-terraform
[kitchen]: https://kitchen.ci/
[make]: https://en.wikipedia.org/wiki/Make_(software)
[shellcheck]: https://www.shellcheck.net/
[terraform-docs]: https://github.com/segmentio/terraform-docs
[terraform]: https://terraform.io/

## TODO/Bugs

Bugs and other todos or details that should be tested for these modules.

* Move these TODO/Bugs to github issue list.
* The `container-engine-robot` account on the service account seems to be
  created very late. The first run through, the IAM bindings for subnet stuff
  can't be applied. But later on it seems to work. Note it is not due to the GKE
  cluster not being created, as that is done later than IAM in the original
  helmsman create script.
* In order to view things in Pantheon, roles must be assigned to the user
  account, as everything has been created under a service account. eg,
```
gcloud projects add-iam-policy-binding --member user:roi@mattcary.info --role roles/compute.admin h2-3001-prod-host-decd
gcloud projects add-iam-policy-binding --member user:roi@mattcary.info --role roles/container.admin h2-3001-prod-host-decd
```
* `terraform destroy` doesn't delete past projects. It appears that using a
  shared VPC puts liens on the host projet (and others?) which must be manually
  deleted before the projects can be destroyed.