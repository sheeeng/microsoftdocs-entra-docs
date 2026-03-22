---
title: Password Policy Overview and Frequently Asked Questions
description: This document explains password policies in a clear and easy-to-understand way.
ms.topic: faq
ms.date: 03/22/2026
ms.reviewer: ruimurakami
adobe-target: true
# Customer intent: As a Microsoft Entra Administrator, I want to understand how password policies apply to different types of users
---
# Password Policy Overview and Frequently Asked Questions

This document provides answers to frequently asked questions about password policies and password expiration, with a focus on how these policies apply to different user types in Microsoft Entra ID.

## Password Policy Quick Reference by User Type
 [![Password policy quick reference](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)

## Policies Evaluated During Password Change or Reset (Password Length and Complexity)

### Synced Users
The on-premises Active Directory (AD) password policy that applies to synced users can be modified under Account Policies → Password Policy, just like the password expiration settings.
For example, by changing the minimum password length, you can relax the requirement to six characters or fewer, or enforce a stricter policy such as ten characters or more.
 [![Password length](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)

If the password writeback option in Microsoft Entra ID Connect is enabled, synced users can change or reset their passwords directly from Entra ID.
In this case, the password is evaluated by Entra ID before the on-premises AD password policy is applied, which prevents users from setting easily guessable passwords.
Depending on the environment, synced users may also be subject to Entra ID restrictions such as the global banned password list. Please refer to the FAQ section below for more information.

### Cloud-Only Users
For cloud users, the Entra ID password policy cannot be customized, except for password expiration.
For detailed information about the Entra ID password policy, please refer to [the relevant public documentation](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sspr-policy?tabs=ms-powershell#microsoft-entra-password-policies).
Although Entra ID does not provide the same granular password complexity settings as on-premises AD, it does include a global banned password list and a custom banned password list.
The global banned password list is enabled for all tenants and cannot be disabled.
It blocks weak passwords such as admin or baseball.
The custom banned password list allows organizations to register words such as the company name or abbreviations and prevent them from being used in passwords.
For details, see [the related public documentation](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-password-ban-bad#global-banned-password-list).

### Guest Users
Guest users are subject to the settings of their home tenant, where they are originally registered as members.
If you need to confirm which password policies apply to a guest user, please contact an administrator of the home tenant.

## Policies Evaluated During Authentication (Password Expiration)
Password expiration specifies the maximum number of days a single password can be used.
When the expiration date is reached, users are required to change their password the next time they sign in.
When considering password expiration, it is helpful to look not only at whether the user is a synced user or a cloud user, but also whether authentication occurs on-premises or in the cloud.
Each scenario is described in detail below. Please refer to the section that matches the user type and environment you want to review

### Synced Users with Password Hash Synchronization
It is sometimes misunderstood that the on-premises AD password expiration is directly synchronized to Entra ID.
However, password expiration values for synced users are stored separately in on-premises AD and in Entra ID.
Because password information exists in both environments, the applied expiration policy depends on where the user signs in (where authentication occurs).
For example:

When signing in to a domain-joined client device, authentication occurs in on-premises AD, so the on-premises AD password expiration applies.
When signing in to the Azure portal or Microsoft 365, the Entra ID password expiration policy applies.

When the on-premises AD password expires, the user is prompted to change the password during the next on-premises sign-in.
After the password is changed, it is synchronized to Entra ID, allowing the user to sign in to Entra ID with the new password.
By default, for password hash–synchronized users, the Entra ID password expiration is set to never expire.
As a result, even after the on-premises AD password expires, users can still sign in to Entra ID using the expired password.
If you do not want users to sign in to Entra ID with an expired on-premises password, enable the
CloudPasswordPolicyForPasswordSyncedUsersEnabled option so that Entra ID does not treat passwords as non-expiring.

### Synced Users Authenticated On-Premises (Pass-through Authentication or AD FS)
When using Pass-through Authentication or AD FS, authentication for Azure portal and Microsoft 365 sign-ins is performed in on-premises AD, not in Entra ID.
Therefore, the on-premises AD password expiration policy applies even when users sign in to the Azure portal.
To check password expiration settings, review the on-premises AD password policy.
You can view the on-premises AD password policy by editing the Default Domain Policy in the Group Policy Management Console and navigating to
Account Policies → Password Policy.
Note that account policies can be defined in only one GPO linked to the domain.
For this reason, configure these settings by editing the Default Domain Policy.

 [![Password length](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)


### Cloud-Only Users Created Directly in Entra ID
For cloud users created directly in Entra ID, the password expiration period specified in the Entra ID password policy applies.
After the configured number of days has passed, users are prompted to change their password when signing in to Entra ID.
Password expiration in Entra ID can be changed using the Microsoft 365 admin center or PowerShell.

1. Access to [Password expiration policy setting](https://admin.cloud.microsoft/?#/Settings/SecurityPrivacy/:/Settings/L1/PasswordPolicy)
1. In the image below, the password expiration is set to Never expire.
 [![Password expiration never](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)

If you want to configure a password expiration, uncheck the checkbox as shown below.
[![Password expiration uncheck](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)

> [!Hint]
> Even in environments where the password expiration is set to 90 days using the above setting, there may be scenarios where you want to set the password to never expire for only specific users, such as system accounts.
> In this case, in addition to the above setting, you can individually configure the account to never expire by setting DisablePasswordExpiration using the Update-MgUser command for that specific account. For more details, please refer to [Setting an individual user’s password to never expire](https://learn.microsoft.com/en-us/microsoft-365/admin/misc/password-policy-recommendations?view=o365-worldwide).

### Guest Users
Password expiration for guest users is managed by the organization in the guest’s home tenant.
The resource tenant that invited the guest does not manage this setting.
If the guest user uses a Microsoft account such as user@outlook.com, the password policy defined by that service applies.

## FAQ

### In a password hash synchronization environment, users can still access Entra ID with their old password after the on-premises AD password has expired. Can the Entra ID password expiration for synced users be changed from “never expire”?

Yes. This can be configured by enabling the CloudPasswordPolicyForPasswordSyncedUsersEnabled option at the tenant level.
To allow users to change their passwords from Entra ID, password writeback must also be enabled in Entra ID Connect.
By running the following commands, you can enable password expiration for synced users in Entra ID.

$OnPremSync = Get-MgDirectoryOnPremiseSynchronization
$OnPremSync.Features.CloudPasswordPolicyForPasswordSyncedUsersEnabled = $true

Update-MgDirectoryOnPremiseSynchronization `
  -OnPremisesDirectorySynchronizationId $OnPremSync.Id `
  -Features $OnPremSync.Features

> [!Warning]
> Password expiration values for synced users are not synchronized from on-premises AD to Entra ID. After enabling this feature, the on-premises password policy applies when signing in on-premises, and the Entra ID password policy applies when authentication occurs in Entra ID. As a result, the applicable password expiration policy depends on where the user signs in.
> If different expiration periods are configured (for example, 30 days on-premises and 90 days in Entra ID), it can become difficult for users to determine when their password will expire. For this reason, it is recommended to configure the same expiration period for both on-premises AD and Entra ID.
For example, if both environments require a password change after 90 days, the password will expire at roughly the same time.
> When a password is changed either on-premises or in Entra ID, the change is synchronized through password hash synchronization and password writeback, and the expiration timer is reset.

### When “User must change password at next logon” is set on-premises, users see an “incorrect password” error when signing in to Entra ID. Can users change their password in Entra ID instead?

Yes. This is possible by enabling the UserForcePasswordChangeOnLogonEnabled option at the tenant level.
To allow password changes from Entra ID, password writeback must be enabled in Entra ID Connect.

```powershell
$OnPremSync = Get-MgDirectoryOnPremiseSynchronization
$OnPremSync.Features.UserForcePasswordChangeOnLogonEnabled = $true

Update-MgDirectoryOnPremiseSynchronization
 -OnPremisesDirectorySynchronizationId $OnPremSync.Id
 -Features $OnPremSync.FeaturesShow more lines
```

By default, this feature is disabled. Without enabling it, users must change their password on-premises first, otherwise sign-in fails. After enabling this option, users are prompted to change their password in Entra ID. Please also refer to [the related public documentation](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-password-hash-synchronization#enforcecloudpasswordpolicyforpasswordsyncedusers) for details.

### What is the default password expiration period shown in the Microsoft 365 admin center?
Previously, the default was 90 days. However, for tenants created around spring 2021 or later, the default value is never expire.
As a result, some tenants have a 90-day expiration, while others are configured with no expiration by default.

### The Entra ID password policy is now set to never expire. Is there a way to check who changed it?
If the policy was changed recently, you can check the Microsoft Entra ID audit logs.
Filter by the Set password policy activity and review the user listed as the Initiator (Actor).

### When running Get-MgDomain in PowerShell, the password expiration shows 2147483647 days. What does this mean?
This value indicates that the password expiration for the domain is set to never expire.

### I edited the password policy in the Microsoft 365 admin center or via PowerShell, but I do not receive expiration notifications.
Previously, notifications were displayed using a bell icon in the Microsoft 365 portal.
However, this notification feature has been retired, and “password about to expire” notifications are no longer sent.

### I can sign in to a Microsoft Entra–joined device even though my password has expired. When will I be required to change it?
You can sign in to the device and reach the Windows desktop even if the password has expired.
You are prompted to change your password when signing in to Entra ID–integrated cloud resources such as the Azure portal, Microsoft 365 admin center, or Exchange Online.

###  Is there a way to check the password expiration date for individual users rather than the tenant-wide setting?
There is no simple way to check this in the Azure portal.
You must calculate it using values obtained from PowerShell.
To check the tenant password expiration policy:

1. Run Connect-MgGraph
1. Run Get-MgContext to confirm you are connected to the correct tenant
1. Run the following command:

```powershell
Get-MgDomain -DomainId contoso.onmicrosoft.com |  select Id, PasswordNotificationWindowInDays, PasswordValidityPeriodInDays
```

Example output:

```powershell
Id                            PasswordNotificationWindowInDays PasswordValidityPeriodInDays
--                            -------------------------------- ----------------------------
contoso.onmicrosoft.com                                   14                   90
```

1. To retrieve the date when a user last changed their password:

```powershell
Get-MgUser -UserId admin@contoso.onmicrosoft.com `  -Property UserPrincipalName, LastPasswordChangeDateTime |  fl UserPrincipalName, LastPasswordChangeDateTime
```
Example output:

```powershell
UserPrincipalName          : admin@M365x61971868.onmicrosoft.com
LastPasswordChangeDateTime : 2024/02/27 4:51:12
```

1. Compare the last password change date with the expiration period to determine the expiration date.

### Is “change password” the same as “reset password”?
No. These terms represent different actions.
[![Password change and Password reset](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)

Password change refers to a scenario where the user changes their password to a new one when they already know their current password.
When performing a password change, the user is required to enter their current (old) password. For example, the prompt shown below is also considered a password change. In addition, even if the password has not yet expired, the user can explicitly change their password from the My Account page.

On the other hand, a password reset does not necessarily require the user to know their current password. While a password reset can still be performed even if the user knows the password, it is most commonly used in scenarios where the password is unknown or has been forgotten. (There are also scenarios where a password reset is required as a response to detected risks or security events.)
When a user performs a password reset by themselves, the “Recover your account” screen, as shown below, is displayed.
[![SSPR screen](media/tutorial-enable-sspr-writeback/xxxxx.png)](media/tutorial-enable-sspr-writeback/xxxxx.png#lightbox)

### Is there a way to prevent users from changing their password?
No. Users can always change their own password.
However, administrators can disable self-service password reset (SSPR).

### Can I prevent specific words from being used in passwords?
Yes. You can add words such as your company name to the custom banned password list.
Common weak passwords such as Admin or Password are already blocked by the global banned password list, which is automatically applied to all tenants.

### When are global and custom banned passwords evaluated?
They are evaluated during password change and password reset operations.
Depending on the password evaluation results described in [the public documentation](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-password-ban-bad#how-are-passwords-evaluated), a password containing a custom banned word may still be accepted in some cases.

### When synced users change their password from Entra ID, the screen always shows “Please wait a few minutes.” Why?
This occurs when the password meets the on-premises AD password policy but does not meet the Entra ID password policy.
For example, this can happen if:
* The on-premises minimum password length is seven characters or fewer
* Password complexity is disabled on-premises

If the password meets both the Entra ID and on-premises policies (for example, at least eight characters and three of four character types), the password is updated immediately and the message does not appear. If only the on-premises policy is satisfied, the password is accepted on-premises and synchronized to Entra ID during the next password hash synchronization cycle (up to two minutes).
This is expected behavior and does not require remediation.

### The same “Please wait a few minutes” message appears when resetting a password from Entra ID. Why?
The reason and behavior are the same as during password change, although the displayed screens differ slightly.

### Does Microsoft provide recommended password policy guidance?
[Microsoft’s recommended password policy](https://learn.microsoft.com/en-us/microsoft-365/admin/misc/password-policy-recommendations?view=o365-worldwide) includes the following principles:

* Set a minimum password length of 8 characters
* Do not require specific character composition (such as mandatory symbols)
* Do not force periodic password changes
* Block commonly used or weak passwords
* Do not reuse organizational passwords for non-business purposes

Research has shown that enforcing frequent password changes or complex symbol requirements can actually reduce security, as users tend to reuse or simplify passwords.
Microsoft recommends modern password practices based on these guidelines rather than outdated assumptions.