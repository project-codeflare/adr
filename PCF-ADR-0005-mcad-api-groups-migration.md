# MCAD API Groups Migration Plan

|                |                          |
| -------------- | ------------------------ |
| Date           | 07/31/2023               |
| Scope          |                          |
| Status         | Proposed                 |
| Authors        | [Antonin Stefanutti](@astefanutti) |
| Supersedes     | N/A                      |
| Superseded by: | N/A                      |
| Issues         | https://github.com/project-codeflare/multi-cluster-app-dispatcher/issues/352 |

## What

The MCAD APIs are currently defined in the `mcad.ibm.com` and `ibm.com` groups.
The objective of this document is to propose a plan to migrate them to the `.codeflare.dev` top-level group.

This migration will also be the opportunity to organize the APIs by functional areas, i.e., the `workload.codeflare.dev` group for workload management APIs, and the `quota.codeflare.dev` group for quota management APIs.

## Why

It's conventional for open-source projects to use their domain name as their top-level API group, so DNS is leveraged as a global naming system.

The choice of the current MCAD API groups predates the acquisition of the `codeflare.dev` domain, which explains why they do not follow that convention.

Some of the MCAD APIs, mainly the AppWrapper API, is used by downstream projects, as well as end-users.
From a client standpoint, the impacts of an API group change imply updating the `apiVersion` field of all the MCAD resources they define, as well as updating the group segment of the REST endpoints, created from the CRDs, that they consume.

## Goals

The goal is to inform a migration path, to provide these clients with a transition to the new API groups, that balances compatibility and costs.

## Non-Goals

It is not a goal to provide an automated migration.
Downstream projects and end-users will have to address the impacts manually to perform the migration to the new API groups.

## How

A project like [TorchX](https://github.com/pytorch/torchx), which integrates with MCAD[^1] via the AppWrapper API, has its own release cadence, and it'd be challenging to have it migrated within a single migration cycle, without breaking compatibility.

[^1]: https://pytorch.org/torchx/latest/schedulers/kubernetes_mcad.html

The following plan proposes a progressive migration path, implemented in three phases, as follows:

### Phase 1: Add Dual API Groups Support

The new API groups are introduced in MCAD, along with the existing ones.
Functionally, the MCAD and InstaScale controllers are updated, so they can equally reconcile the old and new APIs.

To implement that dual API groups support, it may be necessary to duplicate the MCAD API packages, as well as the generated client packages.
Indeed, [controller-runtime](https://github.com/kubernetes-sigs/controller-runtime) APIs, which are used by InstaScale, are designed to use the API structs, and require the mapping between the API structs and the Group/Version/Kind(GVK)s to be an injection.
Otherwise stated, controller-runtime mandates the scheme holding the mapping between the API structs and the GVKs to have a single GVK entry per API struct.

This boils down to the following tasks:

* Duplicate the API packages, with the new API groups, and mark the old ones as deprecated
* Generate the new CRDs by leveraging: https://github.com/project-codeflare/multi-cluster-app-dispatcher/pull/456
* Mark the old CRDs as internal, using the `operators.operatorframework.io/internal-objects` annotation
* Generate the new client packages, by leveraging: https://github.com/project-codeflare/multi-cluster-app-dispatcher/issues/514
* Duplicate the MCAD controllers, to reconcile the new APIs
* Add RBAC for the new API groups to MCAD manifests (both in Helm charts and in the CodeFlare operator)
* Update tests, documentation and samples to use the new API groups
* Upgrade InstaScale to the newer MCAD version
* Duplicate the InstaScale controller, to reconcile the new AppWrapper API
* Add RBAC for the new API groups to InstaScale manifests (both in Helm charts and in the CodeFlare operator)

Note: all the duplications will be cleaned up in Phase 3.

The internal APIs may directly be migrated to the new API groups, without implementing dual support for them.

### Phase 2: Progressive Migration

The migration should be announced on the usual communication channels.

The following downstream projects must be migrated to using the new API groups:

* The TorchX MCAD scheduler: https://github.com/pytorch/torchx/blob/main/torchx/schedulers/kubernetes_mcad_scheduler.py
* The CodeFlare SDK: https://github.com/project-codeflare/codeflare-sdk
* ODH Data Science Pipeline operator: https://github.com/opendatahub-io/data-science-pipelines-operator
* KubeRay documentation: https://ray-project.github.io/kuberay/guidance/kuberay-with-MCAD (no mention of the AppWrapper API group, but the links to the project materials can be updated)

### Phase 3: Decommission old API Groups

Once all the downstream projects have migrated, and existing users acknowledged the migration plan, the following clean-up tasks should be performed:

* Delete the old MCAD API packages
* Delete the old MCAD client packages
* Delete the old MCAD CRDs
* Remove RBAC for the old API groups from MCAD manifests (both in Helm charts and in the CodeFlare operator)
* Delete the duplicated MCAD controllers
* Delete the duplicated InstaScale controllers
* Remove RBAC for the old API groups from InstaScale manifests

## Open Questions

* Is dual mode support necessary for the quota APIs?
  While they'll eventually be public APIs, these are rather new, and may not be actually used yet.

## Alternatives

Given the TorchX MCAD scheduler is currently delivered as part of the [CodeFlare fork of TorchX](https://github.com/project-codeflare/torchx), all the impacted components are managed internally.
That means a _one-shot_ migration can be a viable alternative, assuming we accept the development branches of these components may transiently break, within the  span of that _one-shot_ migration release cycle.

As an example, this _one-shot_ migration could be achieved during the next development cycle of the CodeFlare stack, i.e., the upcoming v0.7.0 release, in the following order:

| Component        | Version |
| ---------------- | ------- |
| MCAD             | v1.34.0 |
| InstaScale       | v0.7.0  |
| CodeFlare TorchX | v0.7.0  |
| CodeFlare SDK    | v0.7.0  |

The ODH Data Science Pipeline operator update can be done as soon as ODH upgrades to that newer CodeFlare version.

The KubeRay documentation can be updated independently.

## Stakeholder Impacts

| Group                         | Key Contacts       | Date       | Impacted? |
| ----------------------------- | ------------------ | ---------- | --------- |
| CodeFlare SDK                 | Mustafa Eyceoz     |            | yes       |
| MCAD                          | Abhishek Malvankar |            | yes       |
| InstaScale                    | Abhishek Malvankar |            | yes       |
| CodeFlare Operator            | Anish Asthana      |            | no        |

## Reviews

Reviews on the pull request will suffice for the approval process.
At least 2 approvals are required prior to this ADR being merged.
The ADR must also remain open for one week.
