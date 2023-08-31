# CodeFlare Operator Redesign

|                |                          |
| -------------- | ------------------------ |
| Date           | 08/28/2023               |
| Scope          |                          |
| Status         | Proposed                 |
| Authors        | [Antonin Stefanutti](@astefanutti) |
| Supersedes     | N/A                      |
| Superseded by: | N/A                      |
| Issues         | https://github.com/project-codeflare/codeflare-operator/issues/153 |

## What

The CodeFlare operator currently implements a variant of the _operator of operators_ pattern (also known as _meta_ or _super_ operator), i.e., it operates on operands that are operators / controllers, specfically MCAD and InstaScale.

This ADR aims at discussing the advantages and disavantages of such a design, and proposing alternatives, in particular the _idiomatic_ operator pattern, where the CodeFlare operands, such as instances of the AppWrapper CRD, would be directly operated on by the CodeFlare operator.

## Why

The _operator of operators_ approach provides a high level of flexibility when it comes to installing the components of the CodeFlare stack.
For instance, it enables administrators to change each component version, as well as container image, dynamically, by changing the component configuration[^1].

[^1]: See [replacing images for MCAD or InstaScale operators](https://github.com/project-codeflare/codeflare-operator/wiki/Helpful-SRE-Information-on-CodeFlare-Stack#replacing-images-in-mcad-operator-or-instascale-operators)

Essentially, the CodeFlare currently owns the installation concern.

While that sounds convenient, this also raises a number of concerns:

* Installation is a cross-cutting concern, that's often already owned by the underlying platform, e.g. [OLM](https://olm.operatorframework.io) in OpenShift, or even application-level platform like [OpenDataHub](https://opendatahub.io).
  The Operator SDK documentation explicitely recommends that [operators should not managed other operators](https://sdk.operatorframework.io/docs/best-practices/common-recommendation/#ideally-operators-does-not-manage-other-operators), and mentions [it's OLM's job to manage the deployment and lifecycle of operators](https://sdk.operatorframework.io/docs/best-practices/best-practices/), as a best practice;
* Each backend component of the stack is managed as an individual operator, which linearly increases the Total Cost of Ownership (TCO), on each phase of the application lifecycle (development, release, packaging, deployment, runtime, operation, ...);
* The installation of the managed operators increases the RBAC surface, specifically it forces the CodeFlare operator to have the union of the managed operators RBAC, to satisfy Kubernetes privilege escalation prevention rules, in addition to permissions for creating ServiceAccount, ClusterRole and ClusterRoleBinding resources;
* The MCAD and InstaScale resources, managed by the CodeFlare operator, are singletons in practice, but nothing prevents users from creating multiple instances technically, which leads to undefined behaviors.
  Conceptually, it's also unclear what multiple instances of these resources means, nor entails;
* The compatibility matrix between all the components would have to be enforced at runtime, i.e., breaking changes in their lineages would have to be encoded and enforced at runtime, as discussed in [codeflare-operator#120](https://github.com/project-codeflare/codeflare-operator/pull/120), and reported in [codeflare-operator#121](https://github.com/project-codeflare/codeflare-operator/issues/121), while it comes implicitely for free, when the versions are fixed at build time.

## Goals

The goal is to refactor the CodeFlare operator, such that:
* It delegates the installation concern to the underlying platform,
* It follows Operator SDK best practices.

## Non-Goals

* While the upgrade scenario should work, it is not a goal to provide automated migration for existing users, in particular to migrate the changes they may have introduced, in the MCAD and InstaScale resources, automatically.
* It is not a goal to define a mutli-tenancy model for the CodeFlare operator.

## How

The CodeFlare stack codebase is structured in a multi-repository organisation.
To avoid disrupting this organisation, we propose to refactor the CodeFlare operator, so it embeds the MCAD and InstaScale controllers, directly into a single deployment container process.

This approach has been proved to work in [codeflare-operator/pull/216](https://github.com/project-codeflare/codeflare-operator/pull/216).
That PoC has highlighted the points detailed in the following sections.

Note that this approach aligns well with the Operator [SDK development best practices](https://sdk.operatorframework.io/docs/best-practices/best-practices/#development), in particular:

> Inside an Operator, multiple controllers should be used if multiple CRDs are managed.
This helps in separation of concerns and code readability.
Note that this doesn't necessarily mean that we need to have one container image per controller, but rather one reconciliation loop (which could be running as part of the same Operator binary) per CRD.

### Configuration

Each controller, i.e., MCAD and InstaScale, as well as the CodeFlare operator, must provide a unified configuration.

Currently each component has its own way of sourcing configuration:
* CLI options for MCAD
* CLI options, and a ConfigMap for InstaScale
* CLI options, and CRDs for the CodeFlare operator

We propose to model the configuration of each component as part of their public API.

MCAD would declare its configuration, e.g.:

```go
package config

struct MCADConfiguration {
	// Number of seconds a job will go away for, if it can not be scheduled.
	// Default is 20.
	BackoffTime int `json:"backoffTime,omitempty"`
	// Head of line job will not be bumped away for at least HeadOfLineHoldingTime seconds
	// by higher priority jobs.
	// Default setting to 0 disables this mechanism.
	HeadOfLineHoldingTime int `json:"headOfLineHoldingTime,omitempty"`

	// Remaining configuration...
}
```

InstaScale would also declare its configuration, e.g.:

```go
package config

struct InstaScaleConfiguration {
	// Maximum of cluster nodes that can be scaled out.
	// Defaults to 3.
	MaxScaleoutAllowed int `json:"maxScaleoutAllowed,omitempty"`

	// Remaining configuration...
}
```

Finally, the CodeFlare operator would embed MCAD and InstaScale configurations, along with the generic configuration taken from [component-base](https://github.com/kubernetes/component-base), e.g.:

```go
package config

import (
	configv1alpha1 "k8s.io/component-base/config/v1alpha1"
	mcad "github.com/project-codeflare/multi-cluster-app-dispatcher/pkg/config"
	instascale "github.com/project-codeflare/instascale/pkg/config"
)

struct InstaScaleConfiguration {
	// Enabled controls whether the InstaScale controller is started.
	// It may defaults to true on platforms that InstaScale supports.
	// Otherwise defaults to false.
	Enabled *bool `json:"enabled,omitempty"`

	// The InstaScale controller configuration
	instascale.InstaScaleConfiguration `json:",inline,omitempty"`
}

struct CodeFlareOperatorConfiguration {
	// The MCAD controller configuration
	MCAD *mcad.MCADConfiguration `json:"mcad,omitempty"`

	// The InstaScale controller configuration
	InstaScale *InstaScaleConfiguration `json:"instascale,omitempty"`

	// ClientConnection configures the connection to the cluster
	ClientConnection *configv1alpha1.ClientConnectionConfiguration `json:"clientConnection,omitempty"`

	// LeaderElection is the LeaderElection config to be used when configuring
	// the manager.Manager leader election
	LeaderElection *configv1alpha1.LeaderElectionConfiguration `json:"leaderElection,omitempty"`

	// Remaining configuration...
}
```

The later `struct` would be sourced from a ConfigMap, mounted into the operator deployment container.
To follows the Operator SDK best practice, which states an operator [should always be able to deploy and come up without user input](https://sdk.operatorframework.io/docs/best-practices/best-practices/#summary-1), that ConfigMap would be created when the operator starts, if it doesn't exist already, and initialized with sensible defaults.

We'll provide documentation for existing users, on how to migrate their changes in the MCAD and InstaScale resources, over to the new ConfigMap resource.

### Packaging and Automation

The building and publishing of the MCAD and InstaScale container images can be removed.
The CodeFlare operator will depend on the MCAD and InstaScale controllers Go packages,
so only tagging their respective repositories will be needed.

### Local Development

MCAD and InstaScale repositories can continue to have dedicated main packages, as a convenient way for developers to test their local changes.
The `main` function would translate the existing CLI options, into their respective [configuration](#configuration) `struct`, and pass the later to their controllers.

### Dependency Management

As MCAD and InstaScale controllers will be compiled into a single static binary, dependencies, at the intersection of the their dependency trees, must have compatible versions.

As opposed to the current situation, where they are compiled and executed independently, this will reduce the range of possible versions for common dependencies, like the Kubernetes API and client, which will make the Kubernetes [version skew](https://kubernetes.io/releases/version-skew-policy/) a theoritical lower bound.

While that may sounds like a limitation, upgrading common dependencies, like Kubernetes API and client, is usually straightforward, and has to be done anyway.
The only extra constraint is it'll have to be done in a coordinated way, between MCAD, InstaScale, and the CodeFlare operator.

This has already been done as part of the PoC for that ADR, with the following PRs:
* MCAD: [project-codeflare/multi-cluster-app-dispatcher#552](https://github.com/project-codeflare/multi-cluster-app-dispatcher/pull/552)
* InstaScale: [project-codeflare/instascale#121](https://github.com/project-codeflare/instascale/pull/121)
* CodeFlare operator: [project-codeflare/codeflare-operator#221](https://github.com/project-codeflare/codeflare-operator/pull/221)

Note, this also applies to the version of Golang.

### Deployment and Operation

With the proposed approach, only one deployment will have to be operated and monitored, instead of three.

This will also reduce the compute resources required to install the stack.

## Alternatives

An alternative would be to embed the MCAD and InstaScale controllers within the CodeFlare operator deployment, but in dedicated containers, instead of the same container process.

While this would follow Operator SDK best practices, this would not provide the same level of simplification as the proposed approach.

## Stakeholder Impacts

| Group                         | Key Contacts                           | Date       | Impacted? |
| ----------------------------- | -------------------------------------- | ---------- | --------- |
| CodeFlare SDK                 | Mustafa Eyceoz, Dimitri Saridakis      |            | no        |
| MCAD                          | Abhishek Malvankar, Antonin Stefanutti |            | yes       |
| InstaScale                    | Abhishek Malvankar, Dimitri Saridakis  |            | yes       |
| CodeFlare Operator            | Anish Asthana, Antonin Stefanutti      |            | yes       |

## Reviews

Reviews on the pull request will suffice for the approval process.
At least 2 approvals are required prior to this ADR being merged.
