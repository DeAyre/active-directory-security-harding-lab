# active-directory-security-harding-lab
Hands-on AD lab: least-privilege delegation, password policy GPO, and logon auditing



# Active Directory Security Hardening Lab

A self-hosted Active Directory environment built to test least-privilege delegation, password policy hardening, and security auditing hands-on rather than just on paper. Each step below has a screenshot showing it worked.

![Windows Server](https://img.shields.io/badge/Windows_Server-0078D4?style=for-the-badge&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-5E5E5E?style=for-the-badge)
![Group Policy](https://img.shields.io/badge/Group_Policy-0A66C2?style=for-the-badge)
![VMware](https://img.shields.io/badge/VMware_Fusion-607078?style=for-the-badge&logo=vmware&logoColor=white)

## Environment
| Component | Detail |
|---|---|
| Hypervisor | VMware Fusion |
| Server OS | Windows Server (add version) |
| Domain | lab.local |
| Test accounts | it_user1 (IT_Admins), Finance User1 |

## Skills demonstrated
Active Directory administration, delegation of control, least privilege, Group Policy, password policy, advanced audit policy, Windows Event Viewer, troubleshooting

---

## Part 1: Delegation and least privilege
I delegated control of the **IT** Organizational Unit to the **IT_Admins** security group, granting rights to reset passwords and manage user accounts, scoped only to that OU. Two tests check that boundary.

![Password reset succeeds in IT OU](screenshots/01-delegation-allowed-it-ou.png)
**Test 1: Password reset succeeds inside the IT OU.** Logged in as `it_user1`, a limited domain account, I reset the password for a user in the IT OU. The action completed, confirming the delegated permissions work.

![Access denied in Finance OU](screenshots/02-delegation-denied-finance-ou.png)
**Test 2: The same account is denied in the Finance OU.** Using the same account and tool against a Finance user returned *Access is denied*. Delegation doesn't carry over to other OUs, so a compromised IT account can't touch Finance's accounts.

## Part 2: Password policy hardening
I created a GPO named **IT-Password-Policy** and linked it to the IT OU with stricter settings than the defaults.

![IT-Password-Policy GPO](screenshots/03-password-policy-gpo.png)
- Minimum password length: 14 characters
- Password complexity: enabled
- Maximum password age: 30 days

## Part 3: Security auditing
A control that isn't logged is hard to verify or investigate. I enabled audit policy for logon activity, then triggered a failed and a successful login to confirm events were captured.

![Advanced Audit Policy](screenshots/04-advanced-audit-policy.png)
**Advanced Audit Policy Configuration.** Auditing set to Success and Failure under Account Logon, using the more granular Advanced Audit Policy instead of the legacy node.

![Failed login attempt](screenshots/05-failed-login-attempt.png)
**Test event.** I entered a wrong password for the Administrator account on purpose to generate activity.

![Security log in Event Viewer](screenshots/06-event-viewer-security-log.png)
**Event Viewer.** The Security log shows 4,500+ events, including Event ID 4624 (successful logon) and 4634 (logoff) from testing.

![Event 4624 detail](screenshots/07-event-4624-detail.png)
**Event 4624 detail.** Each event includes the timestamp, source computer, and audit keyword, the details that matter in an investigation.

## Troubleshooting
![Installer error](screenshots/08-installer-error.png)

During the Windows Server install I repeatedly hit *"Windows cannot find the Microsoft Software License Terms."* The cause was VMware Fusion's Easy Install feature injecting a broken unattended-install configuration file. Switching the VM firmware to UEFI Secure Boot and stepping through the installer manually fixed it.

## Why this matters
These are small versions of controls that limit blast radius in real environments. A compromised helpdesk account shouldn't reach Finance's data, weak passwords shouldn't be accepted by default, and when something goes wrong there needs to be a log trail to investigate. Building this and trying to break it made those principles click in a way reading about them didn't.
