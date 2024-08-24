## LLMNR/NBT-NS Poisoning - from Windows

LLMNR & NBT-NS poisoning is possible from a Windows host as well. In the last section, we utilized Responder to capture hashes. This section will explore the tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) and attempt to capture another set of credentials.


### Inveigh - Overview

> 1. If we end up with a Windows host as our attack box, our client provides us with a Windows box to test from, or we land on a Windows host as a local admin via another attack method and would like to look to further our access, the tool Inveigh works similar to Responder, but is written in PowerShell and C#. Inveigh can listen to IPv4 and IPv6 and several other protocols, including **LLMNR, DNS, mDNS, NBNS, DHCPv6, ICMPv6, HTTP, HTTPS, SMB, LDAP, WebDAV, and Proxy Auth**. The tool is available in the C:\Tools directory on the provided Windows attack host.

> 2. We can get started with the PowerShell version as follows and then list all possible parameters. There is a [wiki](https://github.com/Kevin-Robertson/Inveigh/wiki/Parameters) that lists all parameters and usage instructions.


### Using Inveigh


	PS C:\Tools> Import-Module .\Inveigh.ps1
	PS C:\Tools> (Get-Command Invoke-Inveigh).Parameters

	Key                     Value
	---                     -----
	ADIDNSHostsIgnore       System.Management.Automation.ParameterMetadata
	KerberosHostHeader      System.Management.Automation.ParameterMetadata
	ProxyIgnore             System.Management.Automation.ParameterMetadata
	PcapTCP                 System.Management.Automation.ParameterMetadata
	PcapUDP                 System.Management.Automation.ParameterMetadata
	SpooferHostsReply       System.Management.Automation.ParameterMetadata
	SpooferHostsIgnore      System.Management.Automation.ParameterMetadata
	SpooferIPsReply         System.Management.Automation.ParameterMetadata
	SpooferIPsIgnore        System.Management.Automation.ParameterMetadata
	WPADDirectHosts         System.Management.Automation.ParameterMetadata
	WPADAuthIgnore          System.Management.Automation.ParameterMetadata
	ConsoleQueueLimit       System.Management.Automation.ParameterMetadata
	ConsoleStatus           System.Management.Automation.ParameterMetadata
	ADIDNSThreshold         System.Management.Automation.ParameterMetadata
	ADIDNSTTL               System.Management.Automation.ParameterMetadata
	DNSTTL                  System.Management.Automation.ParameterMetadata
	HTTPPort                System.Management.Automation.ParameterMetadata
	HTTPSPort               System.Management.Automation.ParameterMetadata
	KerberosCount           System.Management.Automation.ParameterMetadata
	LLMNRTTL                System.Management.Automation.ParameterMetadata
	mDNSTTL                 System.Management.Automation.ParameterMetadata
	NBNSTTL                 System.Management.Automation.ParameterMetadata
	NBNSBruteForcePause     System.Management.Automation.ParameterMetadata
	ProxyPort               System.Management.Automation.ParameterMetadata
	RunCount                System.Management.Automation.ParameterMetadata
	RunTime                 System.Management.Automation.ParameterMetadata
	WPADPort                System.Management.Automation.ParameterMetadata
	SpooferLearningDelay    System.Management.Automation.ParameterMetadata
	SpooferLearningInterval System.Management.Automation.ParameterMetadata
	SpooferThresholdHost    System.Management.Automation.ParameterMetadata
	SpooferThresholdNetwork System.Management.Automation.ParameterMetadata
	ADIDNSDomain            System.Management.Automation.ParameterMetadata
	ADIDNSDomainController  System.Management.Automation.ParameterMetadata
	ADIDNSForest            System.Management.Automation.ParameterMetadata
	ADIDNSNS                System.Management.Automation.ParameterMetadata
	ADIDNSNSTarget          System.Management.Automation.ParameterMetadata
	ADIDNSZone              System.Management.Automation.ParameterMetadata
	HTTPBasicRealm          System.Management.Automation.ParameterMetadata
	HTTPContentType         System.Management.Automation.ParameterMetadata
	HTTPDefaultFile         System.Management.Automation.ParameterMetadata
	HTTPDefaultEXE          System.Management.Automation.ParameterMetadata
	HTTPResponse            System.Management.Automation.ParameterMetadata
	HTTPSCertIssuer         System.Management.Automation.ParameterMetadata
	HTTPSCertSubject        System.Management.Automation.ParameterMetadata
	NBNSBruteForceHost      System.Management.Automation.ParameterMetadata
	WPADResponse            System.Management.Automation.ParameterMetadata
	Challenge               System.Management.Automation.ParameterMetadata
	ConsoleUnique           System.Management.Automation.ParameterMetadata
	ADIDNS                  System.Management.Automation.ParameterMetadata
	ADIDNSPartition         System.Management.Automation.ParameterMetadata
	ADIDNSACE               System.Management.Automation.ParameterMetadata
	ADIDNSCleanup           System.Management.Automation.ParameterMetadata
	DNS                     System.Management.Automation.ParameterMetadata
	EvadeRG                 System.Management.Automation.ParameterMetadata
	FileOutput              System.Management.Automation.ParameterMetadata
	FileUnique              System.Management.Automation.ParameterMetadata
	HTTP                    System.Management.Automation.ParameterMetadata
	HTTPS                   System.Management.Automation.ParameterMetadata
	HTTPSForceCertDelete    System.Management.Automation.ParameterMetadata
	Kerberos                System.Management.Automation.ParameterMetadata
	LLMNR                   System.Management.Automation.ParameterMetadata
	LogOutput               System.Management.Automation.ParameterMetadata
	MachineAccounts         System.Management.Automation.ParameterMetadata
	mDNS                    System.Management.Automation.ParameterMetadata
	NBNS                    System.Management.Automation.ParameterMetadata
	NBNSBruteForce          System.Management.Automation.ParameterMetadata
	OutputStreamOnly        System.Management.Automation.ParameterMetadata
	Proxy                   System.Management.Automation.ParameterMetadata
	ShowHelp                System.Management.Automation.ParameterMetadata
	SMB                     System.Management.Automation.ParameterMetadata
	SpooferLearning         System.Management.Automation.ParameterMetadata
	SpooferNonprintable     System.Management.Automation.ParameterMetadata
	SpooferRepeat           System.Management.Automation.ParameterMetadata
	StatusOutput            System.Management.Automation.ParameterMetadata
	StartupChecks           System.Management.Automation.ParameterMetadata
	ConsoleOutput           System.Management.Automation.ParameterMetadata
	Elevated                System.Management.Automation.ParameterMetadata
	HTTPAuth                System.Management.Automation.ParameterMetadata
	mDNSTypes               System.Management.Automation.ParameterMetadata
	NBNSTypes               System.Management.Automation.ParameterMetadata
	Pcap                    System.Management.Automation.ParameterMetadata
	ProxyAuth               System.Management.Automation.ParameterMetadata
	Tool                    System.Management.Automation.ParameterMetadata
	WPADAuth                System.Management.Automation.ParameterMetadata
	KerberosHash            System.Management.Automation.ParameterMetadata
	FileOutputDirectory     System.Management.Automation.ParameterMetadata
	HTTPDirectory           System.Management.Automation.ParameterMetadata
	HTTPIP                  System.Management.Automation.ParameterMetadata
	IP                      System.Management.Automation.ParameterMetadata
	NBNSBruteForceTarget    System.Management.Automation.ParameterMetadata
	ProxyIP                 System.Management.Automation.ParameterMetadata
	SpooferIP               System.Management.Automation.ParameterMetadata
	WPADIP                  System.Management.Automation.ParameterMetadata
	ADIDNSCredential        System.Management.Automation.ParameterMetadata
	KerberosCredential      System.Management.Automation.ParameterMetadata
	Inspect                 System.Management.Automation.ParameterMetadata
	invalid_parameter       System.Management.Automation.ParameterMetadata
	Verbose                 System.Management.Automation.ParameterMetadata
	Debug                   System.Management.Automation.ParameterMetadata
	ErrorAction             System.Management.Automation.ParameterMetadata
	WarningAction           System.Management.Automation.ParameterMetadata
	InformationAction       System.Management.Automation.ParameterMetadata
	ErrorVariable           System.Management.Automation.ParameterMetadata
	WarningVariable         System.Management.Automation.ParameterMetadata
	InformationVariable     System.Management.Automation.ParameterMetadata
	OutVariable             System.Management.Automation.ParameterMetadata
	OutBuffer               System.Management.Automation.ParameterMetadata
	PipelineVariable        System.Management.Automation.ParameterMetadata
	
![PowerShell Inveigh Usage](Active-Directory-Enum-and-Attacks/images-windows/invoke-inveigh.png)


> - Let's start Inveigh with LLMNR and NBNS spoofing, and output to the console and write to a file. We will leave the rest of the defaults, which can be seen [here](https://github.com/Kevin-Robertson/Inveigh#parameter-help).

![Inveigh Output](Active-Directory-Enum-and-Attacks/images-windows/inveigh-pwsh-output.png) 

> - We can see that we immediately begin getting LLMNR and mDNS requests. The below animation shows the tool in action. 


### CSharp Inveigh (InveighZero)

> - The PowerShell version of Inveigh is the original version and is no longer updated. The tool author maintains the C# version, which combines the original PoC C# code and a C# port of most of the code from the PowerShell version. Before we can use the C# version of the tool, we have to compile the executable. To save time, we have included a copy of both the PowerShell and compiled executable version of the tool in the **C:\Tools** folder on the target host in the lab, but it is worth walking through the exercise (and best practice) of compiling it yourself using Visual Studio.

> - Let's go ahead and run the C# version with the defaults and start capturing hashes.

![Inveigh PE](Active-Directory-Enum-and-Attacks/images-windows/inveigh-PE.png) 

> - As we can see, the tool starts and shows which options are enabled by default and which are not. The options with a **[+]** are default and enabled by default and the ones with a **[ ]** before them are disabled. The running console output also shows us which options are disabled and, therefore, responses are not being sent (mDNS in the above example). We can also see the message **Press ESC to enter/exit interactive console**, which is very useful while running the tool. The console gives us access to captured credentials/hashes, allows us to stop Inveigh, and more.

> - We can hit the **esc** key to enter the console while Inveigh is running.

> - After typing **HELP** and hitting enter, we are presented with several options:

![Inveigh PE HELP](Active-Directory-Enum-and-Attacks/images-windows/inveight-PE-Help.png) 

> - We can quickly view unique captured hashes by typing **GET NTLMV2UNIQUE**.

![Inveigh Hashes](Active-Directory-Enum-and-Attacks/images-windows/inveigh-hashes.png) 

> - We can type in **GET NTLMV2USERNAMES** and see which usernames we have collected. This is helpful if we want a listing of users to perform additional enumeration against and see which are worth attempting to crack offline using Hashcat.


### Cracking Hashes with JTR

	cat svc-qualys.hash                   
	svc_qualys::INLANEFREIGHT:76310A5311765D0C:8289F0B7D517B6D755C7B222837B41DF:0101000000000000760B904664F6DA015B78E4A1907A81090000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0007000800760B904664F6DA01060004000200000008003000300000000000000000000000003000008F3E69EBDDAA730DF7827ECA69C435C1785A863EF7F58D2AAEABB17F20B7D2EB0A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000
	

	$ john --list=FORMATS | grep ntlm
	414 formatsnet-md5, netntlmv2, netntlm, netntlm-naive, net-sha1, nk, notes, md5ns, 
	(149 dynamic formats shown as just "dynamic_n" here)

	$ john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt svc-qualys.hash 
	Using default input encoding: UTF-8
	Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
	Will run 2 OpenMP threads
	Press 'q' or Ctrl-C to abort, almost any other key for status
	security#1       (svc_qualys)     
	1g 0:00:00:03 DONE (2024-08-24 15:38) 0.3115g/s 1222Kp/s 1222Kc/s 1222KC/s seemis27..secill
	Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
	Session completed.
	
### svc-qualys password

> Username: svc-qualys
> Password: security#1


## Remediation

1. > - Mitre ATT&CK lists this technique as ID: **T1557.001, Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay**.

2. > - There are a few ways to mitigate this attack. To ensure that these spoofing attacks are not possible, we can disable LLMNR and NBT-NS. As a word of caution, it is always worth slowly testing out a significant change like this to your environment carefully before rolling it out fully. As penetration testers, we can recommend these remediation steps, but should clearly communicate to our clients that they should test these changes heavily to ensure that disabling both protocols does not break anything in the network.

3. > - We can disable LLMNR in Group Policy by going to Computer Configuration --> Administrative Templates --> Network --> DNS Client and enabling "Turn OFF Multicast Name Resolution."


![Disalbe LLMNR](Active-Directory-Enum-and-Attacks/images-windows/disable-dns-client.png) 


4. > - NBT-NS cannot be disabled via Group Policy but must be disabled locally on each host. We can do this by opening **Network and Sharing Center under Control Panel**, clicking on **Change adapter settings**, right-clicking on the adapter to view its properties, selecting **Internet Protocol Version 4 (TCP/IPv4)**, and clicking the **Properties** button, then clicking on **Advanced** and selecting the **WINS** tab and finally selecting **Disable NetBIOS over TCP/IP**.

![Disable Netbios](Active-Directory-Enum-and-Attacks/images-windows/disable-netbios.png) 

4. > - While it is not possible to disable NBT-NS directly via GPO, we can create a PowerShell script under Computer Configuration --> Windows Settings --> Script (Startup/Shutdown) --> Startup with something like the following:


	$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
	Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}cd 
