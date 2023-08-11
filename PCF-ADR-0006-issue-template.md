# Project CodeFlare - Issue Template

|                |                          |
| -------------- | ------------------------ |
| Date           | 8/11/2023                |
| Scope          |                          |
| Status         | Proposed                 |
| Authors        | [Fiona Waters](@Fiona-Waters)   |
| Supersedes     | N/A                      |
| Superseded by: | N/A                      |
| Issues         | https://github.com/project-codeflare/.github/issues/8    |
| Other docs:    | https://github.com/project-codeflare/.github/pull/14     |

## What
Currently when creating github issues for repositories within Project Codeflare we do not use any templates. The aim of this ADR is to propose adding a defined selection of templates that will allow team members to include all relevant information when creating an issue which will allow others to effectively pick it up and complete it.

## Why

This change will effect all repositories across the stack and therefore all stakeholders should be involved in the process. This ADR aims to document 3 proposed templates for discussion within the team.

## Goals

* Addition of relevant templates for creating issues in any of the Project Codeflare repositories.

## How

The proposal includes the addition of 3 templates via a drop down menu which will be available when creating an issue. The proposed templates are as follows:

1. Feature Request

    ### Name of Feature or Improvement

    TL;DR of what you are requesting...

    ### Description of Problem the Feature Should Solve

    A clear and concise description of the problem the proposal would address. E.g., "I'm frequently frustrated when X happens." "It's too tedious/confusing to do Y."

    ### Describe the Solution You Would Like to See

    Description of the proposed solution.

    ### Describe Alternatives You Have Considered

    Description of any alternative solutions or features you have considered.

    ### Additional Context

    Add any other context, screenshots, console logs, etc. about the request here.


2. Bug Report

    ### Describe the Bug

    A clear and concise description of the bug.

    ### Steps to Reproduce the Bug

    1. Go to '...'
    2. Click on '....'
    3. Scroll down to '....'
    4. See error

    ### What Have You Already Tried to Debug the Issue?

    Provide a clear and concise description of how you tried to debug this issue.

    ### Expected Behavior

    A clear and concise description of what you expected to happen instead.

    ### Screenshots, Console Output, Logs, etc.

    Add screenshots of UIs (like dashboards), etc. that help explain the issue.

    ### Affected Releases

    List any release versions, git commit hashes, or git tags, etc. that you know show the bug. If it is the latest `HEAD` on `main`, just put that.

    ### Additional Context

    Add as applicable and when known:

    * OS: 1) MacOS, 2) Linux, 3) Windows: [1 - 3]
    * OS Version: [e.g. RedHat Linux X.Y.Z, MacOS Monterey, ...]
    * Browser (UI issues): 1) Chrome, 2) Safari, 3) Firefox, 4) Other (describe):  [1 - 4 + description?]
    * Browser Version (UI issues): [e.g. Firefix 97.0]
    * Cloud: 1) AWS, 2) IBM Cloud, 3) Other (describe), or 4) on-premise: [1 - 4 + description?]
    * Kubernetes: 1) OpenShift, 2) Other K8s [1 - 2 + description]
    * OpenShift or K8s version: [e.g. 1.23.1]
    * Other relevant info

    Add any other information you think might be useful here.


3. Other

    ### WHY
    Why is this change being made?
    ### WHAT
    What is being asked for?
    ### HOW
    Suggestions for how this may be solved. [Optional]
    ### TESTS
    List of related tests
    ### DONE
    Bullet point items for what should be completed


## Alternatives

We have not currently considered any alternatives.

## Stakeholder Impacts

| Group                         | Key Contacts       | Date       | Impacted? |
| ----------------------------- | ----------------   | ---------- | --------- |
| CodeFlare SDK                 | Mustafa Eyceoz     | date       | Yes       |
| MCAD                          | Abhishek Malvankar | date       | Yes       |
| InstaScale                    | Abhishek Malvankar | date       | Yes       |
| CodeFlare Operator            | Anish Asthana      | date       | Yes       |

## References

* https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository 

## Reviews

Reviews on the pull request will suffice for the approval process. At least 2 approvals are required prior to this ADR being merged. The ADR must also remain open for at least one week.
