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

### Service Account Investigation

I reviewed the activity of `svc-helpdesk-automation@company.com` around the time Alex Wilson was added to the `Finance Operations` group.

The service account normally signs in from the company's internal automation server in Texas, and no unusual or foreign sign-ins were identified during the relevant timeframe.

However, there was no helpdesk ticket requesting that Alex be added to `Finance Operations`.

Further review showed that the service account made several unrelated access changes within the same 30-minute period:

- Added Alex Wilson to `Finance Operations`
- Added Sarah Patel to `HR Shared Mailbox Users`
- Added Michael Grant to `Executive Reports Readers`
- Removed Daniel Kim from `VPN-Standard`
- Added Priya Shah to `Payroll Reports`

Each affected user had a legitimate helpdesk ticket open that day, but none of the tickets requested the access change that was actually made.

This suggests the activity may involve manipulation of the helpdesk or automation workflow rather than a direct compromise of the service account.

### Helpdesk Investigation

I reviewed the ticket history for the affected users and found that all five tickets were handled by the same helpdesk technician:

`jason.reed@company.com`

I also reviewed the sign-in history of the other affected users.

No foreign or obviously suspicious sign-ins similar to Alex Wilson's were identified for Sarah Patel, Michael Grant, Priya Shah, or Daniel Kim.

At this point, Jason Reed becomes an important investigation lead because he is the common factor across all of the tickets associated with unauthorized access changes.

The next step is to investigate Jason Reed's account, workstation, permissions, and ticketing activity to determine whether his account was compromised or whether the workflow was intentionally manipulated.

### Jason Reed Account Investigation

I reviewed Jason Reed's sign-in activity to determine whether his account may have been compromised remotely.

Jason's normal activity is from the corporate network in Texas using a registered Windows 11 device and Microsoft Edge.

Three days before Alex Wilson was added to `Finance Operations`, Jason's account had an unusual successful sign-in with the following characteristics:

- Source IP located in the Netherlands
- IP associated with a commercial VPS provider
- Linux operating system
- Google Chrome
- Unregistered device
- MFA marked as "previously satisfied in token"

The sign-in was preceded by two failed password attempts and then succeeded two minutes later.

The session remained active for approximately six hours and accessed:

- Outlook
- SharePoint
- Teams
- The company helpdesk portal

No new MFA method or device was registered during this session.

### Phishing Investigation

Review of Jason Reed's email activity found that