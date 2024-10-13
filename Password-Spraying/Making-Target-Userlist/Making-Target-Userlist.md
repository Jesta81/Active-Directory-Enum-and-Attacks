## Password Spraying - Making a Target User List


### Detailed User Enumeration


- To mount a successful password spraying attack, we first need a list of valid domain users to attempt to authenticate with. There are several ways that we can gather a target list of valid users:



1. By leveraging an SMB NULL session to retrieve a complete list of domain users from the domain controller. 

2. Utilizing an LDAP anonymous bind to query LDAP anonymously and pull down the domain user list. 

3. Using a tool such as **Kerbrute** to validate users utilizing a word list from a source such as the [statistically-likely-usernames GitHub repo](https://github.com/insidetrust/statistically-likely-usernames), or gathered by using a tool such as [linkedin2username](https://github.com/initstring/linkedin2username) to create a list of potentially valid users. 

4. Using a set of credentials from a Linux or Windows attack system either provided by our client or obtained through another means such as LLMNR/NBT-NS response poisoning using **Responder** or even a successful password spray using a smaller wordlist



- No matter the method we choose, it is also vital for us to consider the domain password policy. If we have an SMB NULL session, LDAP anonymous bind, or a set of valid credentials, we can enumerate the password policy. Having this policy in hand is very useful because the minimum password length and whether or not password complexity is enabled can help us formulate the list of passwords we will try in our spray attempts. Knowing the account lockout threshold and bad password timer will tell us how many spray attempts we can do at a time without locking out any accounts and how many minutes we should wait between spray attempts. 


- Again, if we do not know the password policy, we can always ask our client, and, if they won't provide it, we can either try one very targeted password spraying attempt as a "hail mary" if all other options for a foothold have been exhausted. 

- We could also try one spray every few hours in an attempt to not lock out any accounts. Regardless of the method we choose, and if we have the password policy or not, we must always keep a log of our activities, including, but not limited to:



1. The accounts targeted
2. Domain Controller used in the attack
3. Time of the spray
4. Date of the spray
5. Password(s) attempted


- This will help us ensure that we do not duplicate efforts. If an account lockout occurs or our client notices suspicious logon attempts, we can supply them with our notes to crosscheck against their logging systems and ensure nothing nefarious was going on in the network. 



### SMB NULL Session to Pull User List 


- If you are on an internal machine but don’t have valid domain credentials, you can look for SMB NULL sessions or LDAP anonymous binds on Domain Controllers. Either of these will allow you to obtain an accurate list of all users within Active Directory and the password policy. If you already have credentials for a domain user or SYSTEM access on a Windows host, then you can easily query Active Directory for this information.


- Some tools that can leverage SMB NULL sessions and LDAP anonymous binds include [enum4linux](https://github.com/portcullislabs/enum4linux), [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html), and [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec), among others. Regardless of the tool, we'll have to do a bit of filtering to clean up the output and obtain a list of only usernames, one on each line. We can do this with **enum4linux** with the **-U** flag. 



#### Using enum4linux



	> $ enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]" 



![enum4linux usernames](/Password-Spraying/Making-Target-Userlist/images/enum4linux.png) 


![enum4linux usernames](/Password-Spraying/Making-Target-Userlist/images/enum4linux-2.png) 

#### Using rpcclient


	> $ rpcclient -U "" -N 172.16.5.5

	> $ enumdomusers


![rpc usernames](/Password-Spraying/Making-Target-Userlist/images/usernames.png) 



#### Using crackmapexec


- Finally, we can use **CrackMapExec with the --users** flag. 

- This is a useful tool that will also show the badpwdcount (invalid login attempts), so we can remove any accounts from our list that are close to the lockout threshold. 

- It also shows the baddpwdtime, which is the date and time of the last bad password attempt, so we can see how close an account is to having its badpwdcount reset. 

- In an environment with multiple Domain Controllers, this value is maintained separately on each one. To get an accurate total of the account's bad password attempts, we would have to either query each Domain Controller and use the sum of the values or query the Domain Controller with the PDC Emulator FSMO role. 



	> $ crackmapexec smb 172.16.5.5 --users 




![crackmapexec usernames](/Password-Spraying/Making-Target-Userlist/images/cme-users.png) 



#### Gathering Users with LDAP Anonymous


- We can use various tools to gather users when we find an LDAP anonymous bind. Some examples include [windapsearch](https://github.com/ropnop/windapsearch) and [ldapsearch](https://linux.die.net/man/1/ldapsearch). If we choose to use **ldapsearch** we will need to specify a valid LDAP search filter. We can learn more about these search filters in the [Active Directory LDAP](https://academy.hackthebox.com/course/preview/active-directory-ldap) module. 


	> $ ldapsearch -h 172.16.5.5 -x -b "DC=inlanefreight,dc=local" -s sub "(&(objectclass=user))" 



![ldapsearch usernames](/Password-Spraying/Making-Target-Userlist/images/ldapsearch.png) 



####Using windapsearch



Tools such as **windapsearch** make this easier (though we should still understand how to create our own LDAP search filters). Here we can specify anonymous access by providing a blank username with the **-u** flag and the **-U** flag to tell the tool to retrieve just users. 



	> $ ./windapserch.py --dc-ip 172.16.5.5 -u "" -U


![windapsearch usernames](/Password-Spraying/Making-Target-Userlist/images/windapsearch.png) 



### Enumerating Usernames with Kerbrute



- As mentioned in the **Initial Enumeration of The Domain** section, if we have no access at all from our position in the internal network, we can use **Kerbrute** to enumerate valid AD accounts and for password spraying.

- This tool uses [Kerberos Pre-Authentication](https://ldapwiki.com/wiki/Wiki.jsp?page=Kerberos%20Pre-Authentication), which is a much faster and potentially stealthier way to perform password spraying. This method does not generate Windows event ID [4625: An account failed to log on](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625), or a logon failure which is often monitored for. 

- The tool sends TGT requests to the domain controller without Kerberos Pre-Authentication to perform username enumeration. If the KDC responds with the error **PRINCIPAL UNKNOWN**, the username is invalid. Whenever the KDC prompts for Kerberos Pre-Authentication, this signals that the username exists, and the tool will mark it as valid. This method of username enumeration does not cause logon failures and will not lock out accounts. 

- However, once we have a list of valid users and switch gears to use this tool for password spraying, failed Kerberos Pre-Authentication attempts will count towards an account's failed login accounts and can lead to account lockout, so we still must be careful regardless of the method chosen.

- Let's try out this method using the [jsmith.txt wordlist](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt) of 48,705 possible common usernames in the format **flast**. The [statistically-likely-usernames GitHub repo](https://github.com/insidetrust/statistically-likely-usernames) is an excellent resource for this type of attack and contains a variety of different username lists that we can use to enumerate valid usernames using **Kerbrute**. 




	> $ kerbrute userenum -d inlanefreight.local --dc 172.16.6.6 /opt/jsmith.txt



![kerbrute](/Password-Spraying/Making-Target-Userlist/images/kerbrute.png) 



- We've checked over 48,000 usernames in just over 12 seconds and discovered 50+ valid ones. 

- Using Kerbrute for username enumeration will generate event ID [4768: A Kerberos authentication ticket (TGT) was requested](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768). This will only be triggered if Kerberos event logging is enabled via Group Policy. Defenders can tune their SIEM tools to look for an influx of this event ID, which may indicate an attack. 

- If we are successful with this method during a penetration test, this can be an excellent recommendation to add to our report. 

- If we are unable to create a valid username list using any of the methods highlighted above, we could turn back to external information gathering and search for company email addresses or use a tool such as [linkedin2username](https://github.com/initstring/linkedin2username) to mash up possible usernames from a company's LinkedIn page. 



### Credentialed Enumeration to Build our User List


- With valid credentials, we can use any of the tools stated previously to build a user list. A quick and easy way is using CrackMapExec. 


#### Using CrackMapExec with Valid Credentials 


	> $ sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users



![cme valid password](/Password-Spraying/Making-Target-Userlist/images/cme-valid-password.png) 
