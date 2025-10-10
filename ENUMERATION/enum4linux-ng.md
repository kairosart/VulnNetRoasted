`enum4linux-ng` is the modern replacement for the classic *enum4linux*, written in Python and much cleaner + faster. It’s one of the best tools for **Active Directory and SMB enumeration** on Linux.

#Attacking_Machine
Run the following command:

```
enum4linux-ng -A <MACHINE IP>
```

The most interesting information is:

![[Screenshot_2025-10-08_06-00-11.png]]

You’ve confirmed the box is exposing **LDAP (389)**, **LDAPS (636)** and **SMB (139/445)**. Those services give you a lot of enumeration and foothold opportunities.

![[Screenshot_2025-10-08_06-04-14.png]]

This gives us useful context. You’ve confirmed the host is a domain member in the `vulnnet-rst.local` domain with machine `WIN-2BO8M1OE1M1`. That opens a clear, practical path for AD-focused enumeration and common escalation techniques.

## Adding the domain to /etc/hosts

Add the domain to your hosts file on Kali.
```
sudo nano /etc/hosts
```

`<MACHINE IP>     vulnnet-rst.local`


**Next step:** [[AS-REP Roasting]]

