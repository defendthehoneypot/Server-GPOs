#<p align="center">Member Server Group Policy</p>

This policy establishes the basic OS policy, that includes the necessary settings for Security Options, LAPS, Local Users and Groups (Preferences).  This document will be broken down into individual sections so they can be explained further.  As part of applying this GPO we are going to setup role based access to make changing administrative access easier in the future.  Keep in mind some organizations already have a well-designed Active Directory and will not going to require the creation of new groups and OUs, but this document will attempt to provide some guidance if the organization does not already have something in place.  It also helps to create new OUs and groups if you are migrating to a new design for tracking purposes.  All security group and user account naming conventions are an example to help organizations that do not have a standard convention.

1. In Active Directory create a new root level OU named **Security Groups**.  This will contain all of the sub OUs for each of the different systems we are trying to segment.  Now create an OU named “Servers” under **Security Groups**.  In this OU create the following Domain Local security groups below. 
  1. ACL_Servers_LocalAdmins - This security group is an example naming convention.
  2. ACL_Servers_DenyLogonLocally - This security group is an example naming convention.

2. In Active Directory create a new root level OU named **Privileged Accounts**.  Inside this OU create sub OUs for each of sections that will have administrative accounts. i.e. “Server Team”, “Help Desk” or whatever names your company uses.  In the Server Team OU create two more sub OUs.  One named “Groups” and “Users”. 
  1. In the Groups OU create a new Domain Global Group named as below.
    1. USR_ServerTeam_SVAdmins – This security group is an example naming convention.
  2. In the Users OU create a new user account based on company naming convention.
    2. Username.sv – This username is an example naming convention.
  3. Add the username.sv account as a member of USR_ServerTEam_SVAdmins and then add USR_ServerTeam_SVAdmins as a member of ACL_Servers_LocalAdmins.

#Security Options

 Computer Configuration > Polices > Windows Settings > Security Settings > Local Policies > Security Options.  There are no settings to adjust in this section but there are a few settings that need some explanation.

1. Interactive logon: Number of previous logons to cache (in case domain controller is not available) – The STIG requires this to be set at 4 or less.  This GPO sets it to 0 because there is no reason to logon if the network is not available.

2. Network security: Configure encryption types allowed for Kerberos – the STIG has only the highest values set but sometimes environments have older systems that require RC4_HMAC_MD5 and DES_CBC_MD5/CRC.

#Windows Firewall

The firewall is set to not configured due to the complexity of creating rules for servers but I highly suggest you create some rules to at least restrict what subnets can remote the servers.

#Local Account Password Solution (LAPS)

LAPS is a Microsoft solution to manage the password of the local administrator account.  This protects systems by setting a different randomized password on the clients.  It then stores the password in the Active Directory computer object.  Download the LAPS solution from Microsoft here.  LAPS will require the running PowerShell to extend the schema and delegate permissions to allow computer objects to set their password.

1. LAPS can be download from here: https://www.microsoft.com/en-us/download/details.aspx?id=46899

#NOTE: IN ORDER TO CONFIGURE THE LAPS SETTINGS, IT IS ASSUMED THE SOLUTION HAS BEEN DEPLOYED ALREADY.

#Preferences

Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups.  These are the settings to control access to the local groups on the client computers.  Sort the accounts by the Order column.  In each of the settings there are place holders that need to be changed out with your domain groups.

1. Backup Operators – This preference is set to remove all membership.  If your company uses this setting remove the preference.

2. DenyNetworkAccess – This group is set in the Users Rights section of the GPO to deny highly privileged accounts access to the system from the network.  Ensure you remove the place holders and put your Domain Admins and Enterprise Admins group in the preference.

3. Administrators (built-in) – This group is designed to manage the local administrators group.  Modify this preference and ensure to add the security group that contains your server administrators.  Example used in this document ACL_Servers_LocalAdmins

#NOTE:  THE BEST PRACTICE IS TO ADD STANDARD DOMAIN USER ACCOUNTS TO ALLOW RDP ACCESS AND THEN FORCE THEM TO PERFORM A RUN AS TO PERFORM ADMINISTRATIVE TASKS.  SINCE THE LOGON CACHE IS SET TO 0, THIS HELPS MITIGATE THE RISK OF LEAVING ADMINISTRATIVE CACHED CREDENTIALS ON THE CLIENTS. WITH THIS IS MIND IT WOULD BE ACCEPTABLE TO ALLOW THE DOMAIN ACCOUNTS CREATED FOR ADMINISTRATIVE ACCESS TO RDP THE SYSTEMS.

Policy Definitions - I have included all the current policy definitions that I used to create the client and server GPOs, although I highly recommend you download them yourself.

Microsoft Office admx/adml files.

1. Office 2010 - https://www.microsoft.com/en-us/download/details.aspx?id=18968

2. Office 2013 - https://www.microsoft.com/en-us/download/details.aspx?id=35554

3. Office 2016 - https://www.microsoft.com/en-us/download/details.aspx?id=49030

Microsoft MSS Settings - http://serverfault.com/questions/655914/how-can-i-enable-mss-group-policy-settings-windows-server-2012
