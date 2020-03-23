# Introduction

The goal of the `WDACPolicies` repo is to highlight the work and methodology that goes into building [Windows Defender Application Control](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control) (WDAC) code integrity policies and performing software package baselines.

Why am I doing this? All too often, I hear apprehension around deploying application control solutions because "it's too hard", "it's not realistic", or "it requires a high level of maturity to implement." By documenting the process and code involved and offering you the resulting code integrity policy, my aim is to:

1) Demonstrate that this is absolutely within your reach.
2) Allow you to be a more informed judge of whether or not application control is a viable solution to prevent at least one component in nearly all forms of post-exploitation tradecraft.
3) Highlight how getting into a consistent process of software baselining can enable you to achieve a more consistent gold image in your organization that lends itself better to the application of application control policies.
4) Offer a peak behind the curtain of what I look for when baselining a software installation.
5) Give you a sense of what good software builds look like and what bad software builds look like.

# Repository Layout

The `Policies` directory consists of markdown (`.md`) and XML (`.xml`) files. The filename corresponds to the software respective installation. The markdown file contains the documentation of the configuration process and an assessment of the overall hygeine of the software package. The XML file is the code integrity policy that was built for the software package. Both documents reflect a partocular methodology I employ for configuring application control policies - one of striking a balance between security, maintainability, and auditability. You are certainly free to merge the included policies into your own respective code integrity policy but be aware that they reflect my personal methodology and they reflect a snapshot in time.

# Selection Criteria for New Application Policies

Currently, the applications added to this repo are ones that I've taken a personal interest in, have a need for, and feel willing to document the process for. I am open to requests but I will prioritize accordingly as this is a personal project.