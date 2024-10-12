## Enumerating & Retrieving Password Policies



### Enumerating the Pasword Policy - from Linux - Credentialed


- As stated in the previous section, we can pull the domain password policy in several ways, depending on how the domain is configured and whether or not we have valid domain credentials. With valid domain credentials, the password policy can also be obtained remotely using tools such as [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) or **rpcclient**. 


	$ crackmapexec smb <TARGET IP> -u avazquez -p Password123 --pass-pol

- Running ifconfig on the internal network returns a network adapter name of -ens224 and inet 172.16.5.225 and a netmask of 255.255.254.0 

- Running nmap -sn 172.16.5.0.0/23 -T5 finds another host at 172.16.5.5

- And we get the following password policy information on the domain after running the crackmapexec command.


![Password Policy](/Checking-Password-Policies/images/policy.png) 



### Enumerating the Password Policy - from Linux - SMB NULL Sessions


- Without credentials, we may be able to obtain the password policy via an SMB NULL session or LDAP anonymous bind. 

- The first is via an SMB NULL session. SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. 

- SMB NULL session misconfigurations are often the result of legacy Domain Controllers being upgraded in place, ultimately bringing along insecure configurations, which existed by default in older versions of Windows Server. 

- When creating a domain in earlier versions of Windows Server, anonymous access was granted to certain shares, which allowed for domain enumeration. 

- An SMB NULL session can be enumerated easily. 

- **For enumeration, we can use tools such as enum4linux, CrackMapExec, rpcclient**, etc. 

- We can use [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) to check a Domain Controller for SMB NULL session access. 

- Once connected, we can issue an RPC command such as **querydominfo** to obtain information about the domain and confirm NULL session access. 



#### Using rpcclient


	$ rpcclient -U "" -N 172.16.5.5 





- We can also obtain the password policy. We can see that the password policy is relatively weak, allowing a minimum password of 8 characters. 


#### Obtaining the Password Policy using rpcclient





![Password Policy](/Checking-Password-Policies/images/rpc-1.png) 


![Password Policy](/Checking-Password-Policies/images/rpc-2.png) 




- Let's try this using [enum4linux](https://labs.portcullis.co.uk/tools/enum4linux).  **enum4linux** is a tool built around the [Samba suite of tools](https://www.samba.org/samba/docs/current/man-html/samba.7.html) **nmblookup, net, rpcclient and smbclient** to use for enumeration of windows hosts and domains. 

- It can be found pre-installed on many different penetration testing distros, including Parrot Security Linux. Below we have an example output displaying information that can be provided by **enum4linux**. Here are some common enumeration tools and the ports they use: 


![Password Policy](/Checking-Password-Policies/images/smbtool_ports.png) 



#### Using enum4linux



	$ enum4linux -P 172.16.5.5



![Password Policy](/Checking-Password-Policies/images/enum4linux.png) 


![Password Policy](/Checking-Password-Policies/images/enum4linux-2.png) 


- The tool [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) is a rewrite of **enum4linux** in Python, but has additional features such as the ability to export data as YAML or JSON files which can later be used to process the data further or feed it to other tools. It also supports colored output, among other features. 



	$ enum4linux-ng -P 172.16.5.5 -oA ilfreight



![Password Policy](/Checking-Password-Policies/images/enum4ng.png) 



![Password Policy](/Checking-Password-Policies/images/enum4ng-2.png) 



- Enum4linux-ng provided us with a bit clearer output and handy JSON and YAML output using the -oA flag. 


#### Displaying the contents of ilfreight.json


![Password Policy](/Checking-Password-Policies/images/json.png) 



### Enumerating Null Session - from Windows 


- It is less common to do this type of null session attack from Windows, but we could use the command net use **\\host\ipc$ "" /u:""** to establish a null session from a windows machine and confirm if we can perform more of this type of attack.



#### Establish a null session from Windows


	> net use \\DC01\ipc$ "" /u:""


![Password Policy](/Checking-Password-Policies/images/win.png) 



- We can also use a username/password combination to attempt to connect. Let's see some common errors when trying to authenticate: 



#### Error Account is Disabled



	> net use \\DC01\ipc$ "" /u:guest 



![Password Policy](/Checking-Password-Policies/images/win-2.png) 



#### Error: Password is Incorrect



	> net use \\DC01\ips$ "password" /u:guest



![Password Policy](/Checking-Password-Policies/images/win-3.png) 



#### Error: Account is locked out (Password Policy) 



![Password Policy](/Checking-Password-Policies/images/win-4.png) 



#### Enumerating the Password Policy - from Linux - LDAP Anonymous 



- [LDAP anonymous binds](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) allow unauthenticated attackers to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. 

- This is a legacy configuration, and as of Windows Server 2003, only authenticated users are permitted to initiate LDAP requests. We still see this configuration from time to time as an admin may have needed to set up a particular application to allow anonymous binds and given out more than the intended amount of access, thereby giving unauthenticated users access to all objects in AD. 


- With an LDAP anonymous bind, we can use **LDAP-specific enumeration tools such as windapsearch.py, ldapsearch, ad-ldapdomaindump.py**, etc., to pull the password policy. With [ldapsearch](https://linux.die.net/man/1/ldapsearch), it can be a bit cumbersome but doable. One example command to get the password policy is as follows: 



#### Using ldapsearch



	$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength 


![Password Policy](/Checking-Password-Policies/images/ldap.png) 


Here we can see the minimum password length of 8, lockout threshold of 5, and password complexity is set **(pwdProperties set to 1)**.
