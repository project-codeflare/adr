# Project CodeFlare - RayCluster Authentication
|                |                                                                                  |
| -------------- | -------------------------------------------------------------------------------- |
| Date           | 12/19/2023                                                                       |
| Scope          |                                                                                  |
| Status         | provisional                                                                      |
| Authors        | [Kevin Postlethwait](@KPostOffice)                                               |
| Supersedes     | N/A                                                                              |
| Superseded by: | N/A                                                                              |
| Issues         |                                                                                  |
| Other docs:    | none                                                                             |

## What

Enabling OpenShift OAuth as an identity provider and authorization mechanism for KubeRay RayClusters.

## Why

Currently there is no authentication by default enabled in front of the Ray Dashboard and we would like to offer a
unified login experience for the whole of ODH by way of OpenShift OAuth

## Goals

* Protect the dashboard from unauthenticated access by way of an OAuth flow
* Protect the API endpoint from unauthenticated access by way of bearer tokens
* Enable HTTPS by default for accessing the RayCluster API and Dashboard

## Non-Goals

* Creating a fully customizable auth(z) solution

## How

In order to solve this we will add a RayCluster reconciler which creates the necessary resources for enabling
enabling authentication and authorization in front of the API and dashboard.

![Auth Diagram](images/PCF-ADR-008-ray-cluster-auth.jpg)

This will change in Q1 or early Q2 of 2024 with the introduction of Authorino. The overall design stays the same, but
the resources created by the CodeFlare Operator's RayCluster reconciler will change and there will likely no longer be
a need for adding the OAuth Proxy sidecar to the head node definition.

## Blockers

* Currently KubeRay is using `update` rather than `patch` to update the CRs status. This may cause the two reconcilers
  to fight over the state of the resource. We should make changes upstream so that changes are made with a `patch`
  before creating our own reconciler.

## Open Questions

* How to enable TLS? (certificate controller, or )

## Alternatives

* Make changes directly to downstream KubeRay operator
* Add the reconciler to the ODH operator
* Use a mutating webhook for adding the OAuth proxy sidecar to the headnode spec
* Implement a more generic solution upstream

## Risks

If upstream KubeRay doesn't accept changes to how they update the CRs status then we will likely have to switch gears
and use a mutating webhook instead.

## Stakeholder Impacts

| Group                  | Key Contacts        | Date | Impacted? |
| ---------------------- | ------------------- | ---- | --------- |
| CodeFlare SDK          | Dimitri Saridakis   | date | ?         |
| CodeFlare Operator     | Antonin Stefanutti  | date | ?         |

## Reviews

Reviews on the pull request will suffice for the approval process. At least 2 approvals are required prior to this ADR being merged. The ADR must also remain open for at least one week.
