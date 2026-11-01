# Cybersecurity, Windows Systems, and Azure Labs

**Active Directory · Windows Server · Hyper-V · Microsoft Entra ID · Azure RBAC · Kibana · Log analysis**

I use this repository to document how I build Windows environments, organize cloud access, and investigate activity in security logs. These projects connect system configuration, identity administration, and evidence-based troubleshooting.

The Active Directory lab follows a Windows domain from deployment through user administration, policy validation, shared drives, and remote access. The Azure lab covers cloud users, security groups, scoped role assignments, and subscription budgeting. The ELK investigation follows a suspicious connection from the initial log search to the relevant host, destination, and request details.

## Projects at a glance

| Project | What I worked on | Where to start |
|---|---|---|
| **Azure identity and governance** | Organized cloud users and security groups, assigned resource-group roles, reviewed inherited access, and configured a monthly budget | [Identity and access walkthrough](Azure-Identity-Governance/README.md) |
| **Active Directory and Windows administration** | Built a two-VM domain, managed users and groups, applied Group Policy, configured department shares, and troubleshot DNS and access problems | [Build and troubleshooting walkthrough](Active-Directory-Lab/README.md) |
| **ELK suspicious connection investigation** | Used Kibana to narrow HTTP logs, inspect a suspicious user-agent, and document the destination and response associated with a training alert | [Investigation report](ELK-PROJECT/README.md) |

## Active Directory: build, test, troubleshoot

I built a Windows Server 2022 domain controller and a Windows 11 workstation in Hyper-V. After joining the workstation to the domain, I created department OUs, user accounts, security groups, and policies for workstation settings and mapped drives.

The useful part was working through the failures. DNS problems prevented the domain join, a computer-policy query required an elevated session, and a drive-mapping policy applied without creating the expected drives. I traced each issue to the layer involved and checked the result from the workstation.

| Work completed | How I checked it |
|---|---|
| Domain join and user sign-in | Computer account in Active Directory and workstation login evidence |
| User and group administration | OU structure, group membership, password reset, and disabled-account records |
| User and computer policies | Separate user-scope and computer-scope `gpresult` output |
| Department drive mappings | Drive preferences and the drives visible in a department user's session |
| Remote administration | RDP connection and established session |

[Read the Active Directory lab →](Active-Directory-Lab/README.md)

## Azure: organize identities and scope access

I created Entra ID accounts and security groups for administration, VM operations, and read-only review. I assigned Reader and Virtual Machine Contributor at the resource-group level, then checked the IAM scope column to distinguish those assignments from inherited subscription access.

I also configured a $25 monthly budget with actual-cost thresholds at 50%, 80%, and 100%. The walkthrough includes five screenshots, an architecture diagram, and a short recorded RBAC review.

[Read the Azure lab →](Azure-Identity-Governance/README.md)

## ELK: investigate a suspicious connection

I worked through the TryHackMe ItsyBitsy scenario using HTTP connection logs already available in Kibana. I started with the data view and time range, filtered the source address, then examined the request fields behind the matching records.

The investigation tied the selected source to a Pastebin request with a `bitsadmin` user-agent and an HTTP 200 response. I documented the recorded fields separately from the questions I would investigate next, including the process that initiated the request and whether any downloaded content executed.

[Read the ELK investigation →](ELK-PROJECT/README.md)

## How I approach troubleshooting

1. **Define the problem.** Identify the failed behavior or the question raised by an alert.
2. **Check the dependencies.** For the Windows lab, that meant DNS, domain membership, policy scope, group membership, and permissions.
3. **Inspect the evidence.** Read the actual policy results, session behavior, or event fields.
4. **Validate the conclusion.** A policy appearing in a report and a mapped drive appearing in a session answer different questions.
5. **Document the next check.** Keep follow-up work separate from completed results.

## Related project

In my [Infrastructure Monitoring and Incident Investigation Lab](https://github.com/DallasDeas/enterprise-infrastructure-monitoring-lab), I extended this work into Zabbix monitoring, automated Gmail alerts, Windows event correlation, and recovery validation across Windows Server and Ubuntu.

## Repository guide

- [Azure identity and governance](Azure-Identity-Governance/README.md): architecture, users, groups, role assignments, and budget configuration.
- [Azure screenshots](Azure-Identity-Governance/Screenshots): five supporting captures.
- [Active Directory walkthrough](Active-Directory-Lab/README.md): architecture, build sequence, validation, and troubleshooting.
- [Active Directory screenshots](Active-Directory-Lab/Screenshots): the original supporting captures.
- [ELK investigation](ELK-PROJECT/README.md): scope, search process, findings, and annotated evidence.

[My GitHub profile](https://github.com/DallasDeas) · [My portfolio](https://dallasdeas.github.io/)
