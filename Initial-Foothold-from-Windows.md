## LLMNR/NBT-NS Poisoning - from Windows

LLMNR & NBT-NS poisoning is possible from a Windows host as well. In the last section, we utilized Responder to capture hashes. This section will explore the tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) and attempt to capture another set of credentials.


### Inveigh - Overview

> 1. If we end up with a Windows host as our attack box, our client provides us with a Windows box to test from, or we land on a Windows host as a local admin via another attack method and would like to look to further our access, the tool Inveigh works similar to Responder, but is written in PowerShell and C#. Inveigh can listen to IPv4 and IPv6 and several other protocols, including **LLMNR, DNS, mDNS, NBNS, DHCPv6, ICMPv6, HTTP, HTTPS, SMB, LDAP, WebDAV, and Proxy Auth**. The tool is available in the C:\Tools directory on the provided Windows attack host.

> 2. We can get started with the PowerShell version as follows and then list all possible parameters. There is a [wiki](https://github.com/Kevin-Robertson/Inveigh/wiki/Parameters) that lists all parameters and usage instructions.


### Using Inveigh

