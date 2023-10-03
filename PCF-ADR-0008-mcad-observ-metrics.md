# Project CodeFlare - MCAD Custom Metrics Emission


|                |                                                                                   |
| -------------- | --------------------------------------------------------------------------------- |
| Date           | 10/03/2023                                                                        |
| Scope          |                                                                                   |
| Status         | implementable                                                                     |
| Authors        | [Ronen Schaffer](@ronensc), [Rachel Brill](@rachelt44), [Eran Raichstein](@eranra)|
| Supersedes     | N/A                                                                               |
| Superseded by: | N/A                                                                               |
| Issues         |                                                                                   |
| Other docs:    | none                                                                              |

## What

Emit MCAD custom metrics such as total allocatable CPU, GPU and memory.

## Why

MCAD custom metrics information is important for enabling generation of an overall up-to-date observablity view of the running app wrappers and connecting to other stack layers.

## Goals

* Emit MCAD custom metrics 

## Non-Goals

The following are not included in this ADR:
* Emit metrics of other components
* Connect MCAD metrics to metrics of other components

## How

Register collected metrics with the runtime controller of the CodeFlare Operator


## Alternatives

Given the CodeFlare operator re-design that enables off-the-shelf exposure of metrics, we have not currently considered any alternatives.


## Stakeholder Impacts

| Group                  | Key Contacts                          | Date | Impacted? |
| ---------------------- | --------------------------------------| ---- | --------- |
| CodeFlare Operator     | Anish Asthana, Antonin Stefanutti     |      | yes       |
| CodeFlare SDK          | Mustafa Eyceoz, Dimitri Saridakis     |      | no        |
| Dashboard              | Mohammed Abdi                         |      | yes       |
| MCAD                   | Abhishek Malvankar, Antonin Stefanutti|      | yes       |


## References



## Reviews

Reviews on the pull request will suffice for the approval process. At least 2 approvals are required prior to this ADR being merged. The ADR must also remain open for at least one week.
