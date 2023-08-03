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

Given the [TorchX MCAD scheduler](https://pytorch.org/torchx/latest/schedulers/kubernetes_mcad.html) is currently delivered as part of the [CodeFlare fork of TorchX](https://github.com/project-codeflare/torchx), all the impacted components are managed internally.
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

## Alternatives

A _progressive_ migration path could be implemented, i.e., one that does not require all the downstream projects to handle the impacts in locked steps with MCAD.
The goal would be to have that migration path started as soon as possible, and have each downstream project, and end-user, walking the migration path at their own pace, when they decide to upgrade their dependency.

That strategy would require the new API groups to be introduced in MCAD, along with the existing ones.
Functionally, the MCAD and InstaScale controllers would be updated, so they can equally reconcile the old and new APIs.

To implement that dual API groups support, it would be necessary to duplicate the MCAD API packages, as well as the generated client packages.
Indeed, [controller-runtime](https://github.com/kubernetes-sigs/controller-runtime) APIs, which are used by InstaScale, are designed to use the API structs, and require the mapping between the API structs and the Group/Version/Kind(GVK)s to be a bijection.
Otherwise stated, controller-runtime mandates the scheme holding the mapping between the API structs and the GVKs to have a single GVK entry per API struct.

While that'd give extra flexibility for downstream projects, and end-users, to migrate, this would increase the complexity of the migration.
Duplicated code would have to live in the MCAD codebase, for a potentially unbounded period of time.

## Stakeholder Impacts

| Group                         | Key Contacts       | Date       | Impacted? |
| ----------------------------- | ------------------ | ---------- | --------- |
| CodeFlare SDK                 | Mustafa Eyceoz     |            | yes       |
| MCAD                          | Abhishek Malvankar |            | yes       |
| InstaScale                    | Abhishek Malvankar |            | yes       |
| KubeRay                       | Laurentiu Bradin   |            | Doc only ATM, maybe Kuberay API server |
| CodeFlare Operator            | Anish Asthana      |            | yes       |

## Reviews

Reviews on the pull request will suffice for the approval process.
At least 2 approvals are required prior to this ADR being merged.
The ADR must also remain open for one week.
