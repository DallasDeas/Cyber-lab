# Active Directory and Windows Administration Lab

**Hyper-V · Windows Server 2022 · Windows 11 · AD DS · DNS · Group Policy · SMB · RDP**

[Back to project overview](../README.md)

## Project overview

I built this lab to practice the Windows administration tasks behind user access, workstation configuration, and shared resources. I started with a domain controller and a workstation, then added department users, security groups, Group Policy, shared drives, and remote administration.

The build gave me a place to troubleshoot the dependencies between those services. When a domain join or drive mapping failed, I worked through DNS, policy results, group membership, and permissions instead of repeatedly changing the same setting.

## Architecture

```mermaid
flowchart TD
    H["Windows 11 Pro host: Hyper-V"] --> S["Internal virtual switch"]
    S --> D["Windows Server 2022: AD DS and DNS"]
    S --> W["Windows 11 Enterprise workstation"]
    D --> P["Users, groups and Group Policy"]
    D --> F["IT, HR and AllStaff shares"]
    P --> W
    F --> W
```

| Component | Role in my lab |
|---|---|
| Windows 11 Pro host | Ran Hyper-V and the two virtual machines |
| Internal virtual switch | Connected the host and VMs on the lab network |
| Windows Server 2022 Standard VM | Hosted AD DS, DNS, and shared folders |
| Windows 11 Enterprise VM | Joined the domain and provided the user-session test environment |
| Department groups | Controlled drive targeting and share access |

I used static addressing and a lab domain. The server combined domain and file services in this small environment, which also made its availability a shared dependency.

## What I implemented and checked

| Area | Work completed | Validation |
|---|---|---|
| Domain foundation | Installed AD DS, promoted the server, configured DNS, and joined the workstation | Computer account and domain login captures |
| Identity administration | Created four department OUs, five users, department security groups, and an All-Staff group | OU and membership captures |
| Account support | Practiced password resets and account disable/enable tasks | Reset, password-policy, and disabled-account evidence |
| Group Policy | Configured screen-saver lock, wallpaper, and removable-storage policies | User and computer policy results |
| Shared resources | Created three shares and group-targeted drive mappings | Share listing, drive preferences, and a department user session |
| Remote administration | Enabled RDP on the workstation and connected from the server | Connection and established-session captures |

## 1. Build the domain foundation

I enabled Hyper-V, created the internal switch, and installed the server and workstation VMs. On the server, I installed Active Directory Domain Services and promoted it to a domain controller. I then configured the workstation to use the lab DNS service and joined it to the domain.

DNS became the first practical troubleshooting lesson. The workstation could not locate the domain controller while name resolution was failing. After restarting the server's DNS service and adjusting the DNS firewall allowance, I checked resolution again before retrying the join.

<details>
<summary>Evidence: virtual machines, domain membership, and sign-in</summary>

![Hyper-V Manager with the lab VMs](Screenshots/Hyper-V-Manager.png)

I used Hyper-V Manager to check the VM states before testing services inside them.

![Workstation computer account](Screenshots/Computer-Container.png)

The computer account records the workstation's presence in Active Directory.

![Workstation domain login](Screenshots/WS01-Logged-in.png)

The workstation capture documents the login stage of the domain setup.

</details>

## 2. Organize users and access groups

I created IT, HR, Finance, and Management OUs, then provisioned five user accounts. I used department groups and an All-Staff group to organize membership for shared resources.

I also practiced password resets and account state changes. A rejected password reset led me to check the Default Domain Policy and use a password that met the configured requirements.

<details>
<summary>Evidence: identity structure and account administration</summary>

![Department OU structure](Screenshots/OU-Structure.png)

The OU structure organizes the department accounts.

![All-Staff membership](Screenshots/All-Staff-Group.png)

The All-Staff group provides a common membership target.

![IT-Staff membership](Screenshots/IT-Staff-Group.png)

I checked group membership because it later affected drive targeting and access.

![Password reset result](Screenshots/Reset-Password-3.png)

The reset capture records the account-support task.

![Disabled account](Screenshots/Disabled-Account.png)

I practiced disabling an account as part of account lifecycle administration.

![Domain password policy](Screenshots/Reset-Password-PasswordPolicy.png)

I inspected the configured policy after the password reset was rejected.

</details>

## 3. Apply and validate Group Policy

I created three policies: a 10-minute password-protected screen-saver policy, a corporate wallpaper policy, and a removable-storage restriction. I refreshed policy on the workstation and checked the user and computer scopes separately.

```powershell
gpupdate /force
gpresult /r
gpresult /r /scope computer
```

The computer-scope query initially returned Access Denied. Running the command prompt as Administrator allowed the query to complete.

The local-account wallpaper test also helped me understand scope. The domain user's wallpaper setting applied in the domain session. A local user did not receive that same user-targeted setting. Computer Configuration policies still apply to the domain-joined computer; a local sign-in is not a general bypass of domain policy. [Microsoft's Group Policy overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview).

<details>
<summary>Evidence: policy configuration and results</summary>

![Linked Group Policy Objects](Screenshots/GPO-Creation.png)

The management view records the configured policies and their links.

![User policy results](Screenshots/GPResult.png)

I checked the applied user policies after refreshing the workstation.

![Computer policy results](Screenshots/GPResult-scope.png)

The elevated computer-scope result lists Disable-USB-Storage as applied. A removable-device test would provide a separate check of the resulting behavior.

</details>

## 4. Configure department shares and drive mappings

I created IT, HR, and AllStaff shares on the server and configured share permissions around the corresponding security groups. I then used Group Policy Preferences with item-level targeting to map drives by group membership.

| Drive | Share | Intended membership |
|---|---|---|
| X: | HR | HR-Staff |
| Y: | IT | IT-Staff |
| Z: | AllStaff | All-Staff |

The policy initially appeared in the workstation's results, but the expected drives were missing. I checked the account's group membership, added the missing memberships, then signed out and back in. The drives appeared in the refreshed session.

![Configured drive mappings](Screenshots/Drive-Mapping-GPO.png)

The preferences define the three drive letters and share paths.

![Department user session with HR and AllStaff drives](Screenshots/Sarah-Johnson-Drives.png)

The department user session shows HR and AllStaff, with no IT drive displayed. That validates the visible mapping result. Direct share-access testing is a separate check of authorization.

<details>
<summary>Additional evidence: share creation and server-side listing</summary>

![Shared folders on the server](Screenshots/Shares-Created.png)

I created the folders before configuring the drive mappings.

![Net share output](Screenshots/Net-Share.png)

The server-side share listing confirmed the published shares.

</details>

## 5. Test remote administration

I enabled RDP on the workstation, configured access, and connected from the server using `mstsc`. This let me practice administering the workstation through a remote session.

<details>
<summary>Evidence: RDP connection and session</summary>

![RDP connection setup](Screenshots/RDP-Connection.png)

The connection capture records the remote-access setup.

![Established RDP session](Screenshots/RDP.png)

The session capture documents remote workstation access.

</details>

## Troubleshooting record

| Problem | What I changed or checked | Result and lesson |
|---|---|---|
| Windows installation failed at boot or requirements checks | Adjusted VM security settings and enabled the virtual TPM; my original build notes also record disabling Secure Boot | Installation proceeded. For a rebuild, I would verify the installation media and Secure Boot template before changing that protection |
| DNS queries timed out and domain join failed | Restarted DNS, adjusted the DNS firewall allowance, and checked resolution from the workstation | The domain join succeeded after name resolution was restored |
| Password reset was rejected | Reviewed the domain password policy and supplied a compliant password | The reset succeeded |
| Computer-scope policy query returned Access Denied | Opened an elevated command prompt | The query returned the applied computer policies |
| Drive policy applied but drives were missing | Corrected group membership and started a new user session | The expected mappings appeared |
| Administrative account could not access the HR share | Reviewed the share permissions and added the administrative group in this lab | Access was restored. Share permissions and NTFS permissions both need review when investigating effective access |
| Domain and share services were unavailable with the server powered off | Brought the server online before repeating the dependent checks | The lab had one server providing DNS, directory, policy, and file services |

The server-off test showed the dependency on live domain services and server-hosted shares. Windows can also allow cached domain sign-in, depending on prior logons and configuration, so I distinguish local sign-in from access to resources that require domain validation. [Microsoft's cached domain logon guidance](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/cached-domain-logon-information).

## What I learned

The biggest lesson was to check the result at the layer where the problem appeared. A successful policy refresh did not guarantee that a drive preference matched the user. A computer account in Active Directory did not replace testing the workstation session. An administrative account still needed the appropriate share access.

These checks gave me a more deliberate workflow for support and systems troubleshooting: confirm the dependency, inspect the applied configuration, test the user-visible result, and record the change that resolved the issue.

## Next validation steps

- Add CSV-based bulk user provisioning with PowerShell.
- Test allowed and denied direct share access with standard department accounts.
- Test removable-storage behavior alongside the policy-result evidence.
- Separate routine administration from privileged accounts and narrow permissions.
- Add a second domain controller and test directory/DNS availability separately from file-share availability.

[Back to project overview](../README.md) · [ELK investigation](../ELK-PROJECT/README.md)
