# Define testing strategy for Project CodeFlare

|                |                          |
| -------------- | ------------------------ |
| Date           | 06/14/2023               |
| Scope          |                          |
| Status         | Proposed                 |
| Authors        | [Karel Suta](@sutaakar), [Antonin Stefanutti](@astefanutti)  |
| Supersedes     | N/A                      |
| Superseded by: | N/A                      |
| Issues         |                          |
| Other docs:    | [PCF-ADR-0003](https://github.com/project-codeflare/adr/blob/main/PCF-ADR-0003-codeflare-release-process.md)                      |

## What

This ADR introduces an overview of testing strategy and approach for Project CodeFlare.

## Why

Various components of Project CodeFlare currently handle their testing in individual and uncoordinated way. Additionally, there are no tests verifying interactions and compatibility between different components.
This ADR aims to document a unified approach for testing of various components, extending the testing section of [Release process ADR](https://github.com/project-codeflare/adr/blob/main/PCF-ADR-0003-codeflare-release-process.md#testing).

## Goals

* Establish a common test approach for CodeFlare components
* Define a testing scope and priorities
* Analyze and decide test automation environment and tools

## Non-Goals

* Cover certification testing
* Cover non functional testing (performance, security....)

## How


### Scope

High priority components:
* CodeFlare operator
* KubeRay
* MCAD
* ODH Distributed Workloads Component

Lower priority components
* CodeFlare SDK
* MCAD dashboard
* InstaScale


### Test Approach

#### Test levels and requirements:

PR automated check:
* Test execution speed is critical to provide fast feedback (less than 30 minutes)
* Focusing on functional testing, avoid long running tests with lower value
* Test composition (first execute unit tests, when unit tests pass then run e2e tests)
* Use parallelization where possible to speed up execution

Nightly build check (to be implemented once nightly builds are available):
* Relaxed test execution speed requirements (i.e. 2 hours)
* Running complete test suite
* Test also integration with ODH Distributed Workloads Component

Release build check:
* Test speed is important to be able to provide a build fast (less than 1 hour):
* Focusing on functional testing, avoid long running tests with lower value (tested by nightly checks)

#### Testing levels and Type of testing:

| Test type/development cycle  | Feature developer | PR automated checks  | PR review by reviewer | Nightly automated checks  | Release automated checks |
| ---------------------------- | ----------------- | -------------------- | --------------------- | ------------------------- | ------------------------ |
| Manual testing               | Yes               | No                   | Yes                   | No                        | No                       |
| Unit testing                 | Yes               | Yes                  | No                    | Yes                       | Yes                      |
| End to end testing           | Yes               | Yes                  | No                    | Yes                       | Yes                      |
| Integration testing          | Optional          | Yes                  | No                    | Yes                       | Yes                      |
| Upgrade testing              | Optional          | Yes                  | No                    | Yes                       | Yes                      |

Glossary:

Manual testing	- Manual verification of the functionality.

Unit testing	- Unit tests covering the base functionality.

End to end testing 	- Testing including all CodeFlare components, running in a cloud.

Integration testing 	- Testing of interactions between ODH components and CodeFlare components in a cloud.

Upgrade testing	- Testing of component upgrades (deploy new operator version) in a cloud.

### Test Environment

Test environment depends on resource requirements of executed tests. For unit tests we will leverage GitHub actions runner. For end to end/integration/upgrade testing we will currently use KinD cluster running on GitHub actions runner. In case the resources provided by GitHub actions runner are not sufficient we can consider alternatives like [testing farm](https://docs.testing-farm.io/general/0.1/index.html).

For OpenShift testing we consider leveraging [OpenShift-CI](https://github.com/openshift/release/tree/master/ci-operator).

### Defect Management 

#### Bug logging
Bugs found by testing are reported as GitHub issues in respective repository. Every bug description has to contain steps to reproduce.

#### Bug fixing
Bugs are going to be fixed in a similar approach as new features - planned as part of sprint planning, fixed through PRs. If possible the fix contains test coverage, preventing the issue from happening again.


### Test Suite

#### E2E Test Cases

##### Setup

The following steps must be executed once, with a cluster admin role, as prerequisites to running the e2e test cases:
1. Provision a test Kubernetes cluster (resp. an OpenShift cluster)
2. Build the CodeFlare operator container image:
  * Clone the CodeFlare operator source code repository
  * Checkout the branch to be tested, e.g., the PR feature branch, i.e. `git clone --branch mytag0.1 --depth 1 https://example.com/my/repo.git`
  * Build the CodeFlare operator container image at HEAD revision
  * Push the CodeFlare operator container image into a test container image registry (resp. the OpenShift cluster internal image registry)
3. Install the CodeFlare stack components into the test cluster:
  * Configure the CodeFlare operator deployment with Kustomize,  to use the previously built container image
  * Create the codeflare-system Namespace (resp. Project)
  * Deploy the CodeFlare operator using Kustomize
    Note: the installation using OLM is covered as part of the installation / upgrade test cases
  * Deploy the KubeRay operator
  * Create a default MCAD resource
  * Grant the MCAD controller ServiceAccount edit permission on RayCluster resources
  * Create a default InstaScale resource
Wait until all the components are ready
The e2e test cases can now be executed, with a **standard user role**, ideally in parallel, to speed the execution time up and shorten the feedback loop as much as possible.
> **Note**
> The test cases should define lean / minimal compute resources, relative to the provisioned cluster, so parallel execution / throughput of the batch jobs submitted via the MCAD scheduling queue is maximized.

##### Submit a Sample PyTorch Job in a managed Ray Cluster

###### Description

Submit a test PyTorch batch job, to a Ray cluster managed by MCAD, in a user tenant, and assert successful completion of the job. Shutdown the Ray cluster, and assert successful freeing of resources.

###### Scenario

1. Create a test Namespace (resp. Project)
2. Create a test AppWrapper resource with the following specifications: https://github.com/project-codeflare/multi-cluster-app-dispatcher/blob/14d569bec1cd016dd41352e3c026f461d851f480/doc/usage/examples/kuberay/config/aw-raycluster.yaml
3. Wait until the Ray cluster is ready
4. Submit the test batch job to the Ray cluster
5. Wait until the job has completed
6. Assert the job status is successful
7. Delete the test AppWrapper resource
8. Assert all the resources have been successfully freed

##### Submit a Sample PyTorch Job directly via MCAD

Lower priority.

###### Description

Submit a test PyTorch batch job to the MCAD scheduler, in a user tenant, and assert successful completion of the job. Assert successful execution of the job, and clean-up of resources upon the batch job completion.

###### Scenario

1. Create a test Namespace (resp. Project)
2. Create an AppWrapper that allocates resources for running the sample PyTorch script as a batch Job
3. Wait until the test job has completed
4. Assert the job status is successful
5. Delete the test AppWrapper resource
6. Assert all the resources have been successfully freed

##### Run a Sample PyTorch Job in a Ray Cluster using the CodeFlare SDK

###### Description

In an isolated, vanilla Python environment, connected to the test Kubernetes cluster, and impersonated as a standard user, i.e., granted the `edit` role in the testing Namespace (resp. Project), submit a sample PyTorch job in a Ray cluster using the CodeFlare SDK.
Assert successful execution of the job, and clean-up of resources upon the job completion.

###### Scenario

1. Install the CodeFlare SDK, e.g., with `pip install codeflare_sdk==<sdk_version>`
2. Create a test Namespace (resp. Project)
3. Create a Ray cluster using the SDK Cluster API in this namespace
4. Wait until the Ray Cluster is ready
5. Create a job using the SDK Job API, by providing the sample job pip requirements and Python script
6. Submit the job into the previously created Ray cluster
7. Wait until test job has completed
8. Assert the job status is successful
9. Retrieve the job logs using the SDK Job API
10. Tear down the ray cluster using the SDK Cluster API
11. Assert all the resources have been successfully freed

#### OLM Installation / Upgrade Test Cases

##### Setup

The following steps must be executed once, with a **cluster admin role**, as prerequisites to running the OLM installation / upgrade test cases:

1. Provision a test Kubernetes cluster (resp. an OpenShift cluster)
2. For Kubernetes cluster only: install OLM using [installation instructions](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/install/install.md)
3. Build the CodeFlare operator OLM bundle:
  * Clone the CodeFlare operator source code repository
  * Checkout the branch to be tested, e.g., the PR feature branch, i.e. `git clone --branch mytag0.1 --depth 1 https://example.com/my/repo.git`
  * Build the CodeFlare operator container image at HEAD revision
  * Push the CodeFlare operator container image into a test container image registry (resp. the OpenShift cluster internal image registry)
  * Build the OLM bundle image
  * Push the OLM bundle image into a test container image registry (resp. the OpenShift cluster internal image registry)
4. Build the OLM index image:
  * Build the OLM index/catalog image with OPM from the latest Operator Hub community catalog (resp. OpenShift catalog)

> **Note**
> the above should be done according to https://github.com/operator-framework/operator-sdk/issues/5832 when available.

> **Note**
> the channel must be adapted according to https://github.com/project-codeflare/codeflare-operator/issues/126.

  * Push the OLM index/catalog image that’s been built at the previous step to the test container image registry (resp. The OpenShift cluster internal container image registry) 


The test cases can now be executed, with a **cluster admin role**, in sequence, as the upgrade test case depends on the installation one.

##### Installation of the CodeFlare Operator with OLM

###### Description

Install the latest released version of the CodeFlare operator using OLM, and assert successful deployment of the operator. Run a smoke test, to make sure the current version of the operator is working as expected. 

###### Scenario

1. Create a codeflare-system test Namespace (resp. Project)
2. Create a CatalogSource resource pointing to the latest release index image
3. Create a Subscription resource, that points to the CatalogSource resource created at the previous step
4. Run a smoke test, e.g. by running one of the e2e test cases, to make sure the operator current version is working correctly. 

##### Upgrade of the CodeFlare Operator installed via OLM

###### Description

Add a newer version of the CodeFlare operator bundle, replacing the latest released version from the channel subscribed to during the Installation of the CodeFlare Operator with OLM test case. Assert successful upgrade of the operator to the newer version, and run a smoke test, to make sure that newer version is working as expected. 

###### Scenario

> **Note**
> Note: Make sure to execute the Installation of the CodeFlare Operator with OLM test case first.

1. In the same codeflare-system Namespace (resp. Project), update the existing test-catalog CatalogSource resource, by pointing to the newly built OLM index image
2. Assert a new ClusterServiceVersion resource has been created for the newer operator version, and eventually reaches the CSVPhaseSucceeded phase
3. Run a smoke test, e.g., by running one of the e2e test cases, to make sure the operator current version is working correctly.

## Open Questions

1. [Add observability test cases](https://github.com/project-codeflare/codeflare-operator/issues/158)
2. [Add or revise integration test cases in ODH repo](https://github.com/opendatahub-io/distributed-workloads/issues/63)

## Alternatives

We didn't consider any other alternatives

## Stakeholder Impacts

| Group                         | Key Contacts       | Date       | Impacted? |
| ----------------------------- | ------------------ | ---------- | --------- |
| CodeFlare SDK                 | Mustafa Eyceoz     |            | yes       |
| MCAD                          | Abhishek Malvankar |            | yes       |
| InstaScale                    | Abhishek Malvankar |            | yes       |
| CodeFlare Operator            | Anish Asthana      |            | yes       |

## Reviews

Reviews on the pull request will suffice for the approval process. At least 2 approvals are required prior to this ADR being merged. The ADR must also remain open for at least one week.
