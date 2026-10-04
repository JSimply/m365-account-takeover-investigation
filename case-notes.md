# Case Notes

## Case 001 — Suspected Microsoft 365 Account Takeover

### Initial Alert

A Microsoft 365 user successfully authenticated from Dallas, Texas at 08:42 and then successfully authenticated from Bucharest, Romania at 08:53.

Both sign-ins used the correct password. The Romanian session originated from a device that had not previously accessed the account.

### Initial Analysis

I consider this activity suspicious because getting from Dallas to Romania in about 11 minutes is not physically possible.

My initial concern is that a bad actor could have gotten the affected user's credentials and logged in from another device.

However, this does not necessarily mean there is a bad actor because there could be a reason this is happening, such as tunneling or a VPN.

At this point, I would classify the activity as suspicious until I can gather more information.

### Investigation Plan


1. I would investigate both source IP addresses because that could help determine whether the IP addresses have accessed company resources in the past and whether they are associated with company infrastructure, VPNs, proxies, or suspicious activity.

2. I would check the user's login history to determine whether this is abnormal behavior for this user.

3. I would check to see if there are any VPN or tunneling applications or devices that the company or the user's team uses that could explain this behavior.

4. I would check if this is an isolated incident or if other users show similar behavior during the same timeframe, particularly from the same source IP addresses.

5. I would want to see the OS, application, and resources being accessed and determine whether that activity is normal for the user.

6. I would investigate what authentication methods were used for both successful sign-ins. If MFA was used, I would want to determine what MFA method was used, such as a physical security key, biometrics, or another authentication method, and compare the two sign-ins.

## Evidence Received — Sign-In Details

USER
alex.wilson@contoso.com

SIGN-IN #1
Time: 08:42:17
Location: Dallas, Texas, USA
IP: 73.184.22.91
Device ID: CORP-LT-2841
Browser: Microsoft Edge 141
OS: Windows 11
Application: Microsoft 365
MFA: Satisfied by claim in token
Result: Success

SIGN-IN #2
Time: 08:53:46
Location: Bucharest, Romania
IP: 185.220.101.44
Device ID: Unknown
Browser: Chrome 140
OS: Windows 10
Application: Microsoft 365
MFA: Satisfied by claim in token
Result: Success

USER HISTORY
Last 30 days:
- 87 successful sign-ins
- Locations: Texas only
- Known devices: CORP-LT-2841 and iPhone
- No previous Romanian authentication

## Evidence Analysis

I am more suspicious because the user's last 30 days of sign-in history show activity only from Texas and from two known devices: the corporate laptop and the user's iPhone. The Romanian session came from an unknown device and used a different browser, operating system, location, and IP address than the user's normal activity.

There could still be a legitimate explanation, such as the user receiving a new device that has not yet been registered or using a specialized VPN or tunneling service that changes the observed location.

The MFA field also shows "Satisfied by claim in token" for both sign-ins. I would want to understand whether MFA was actually performed during the Romanian sign-in or whether the session reused a token that already contained an MFA claim.

Next, I would investigate whether other users have authenticated from the Romanian IP address. I would also check whether that IP attempted failed or successful sign-ins against other accounts and review what resources were accessed before and after the successful sign-in.

### Additional Findings

A review of the suspicious IP showed failed login attempts against multiple user accounts before a successful login to Alex Wilson's account. The IP had not previously appeared in the tenant and was associated with a commercial hosting/VPS provider.

After the successful login, the account accessed Exchange Online and SharePoint Online within minutes.

Exchange audit logs showed that the account:

- Accessed 14 mailbox items
- Searched for terms including "invoice," "wire," "bank," and "payment"
- Accessed messages in the Finance shared mailbox
- Created an inbox rule that moved messages containing "invoice" to the RSS Feeds folder
- Added an external forwarding address
- Granted itself FullAccess permissions to the Finance shared mailbox
- Continued accessing messages from the Finance mailbox

Based on this activity, I would classify this as a confirmed account compromise. The financial search terms, access to the Finance mailbox, external forwarding, and unauthorized permission changes show activity that does not match normal user behavior.

My immediate response would be to contain Alex Wilson's account and escalate the incident to the Incident Response team. I would also want active sessions and tokens revoked, the account credentials reset, and the MFA configuration reviewed.

### Permission Investigation

Alex Wilson is not an Exchange Administrator, so I investigated how the account was able to grant itself FullAccess to the Finance shared mailbox.

Alex is a member of the `Finance Operations` group, which has delegated permission to manage access to the Finance shared mailbox. His normal job responsibilities do not require membership in this group.

The group membership was added three months earlier by:

`svc-helpdesk-automation@company.com`

This creates an additional investigation lead. The next step is to determine why the service account added Alex to the group and whether the service account or automation process was also compromised or misconfigured.