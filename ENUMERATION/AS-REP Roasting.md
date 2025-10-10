
AS-REP Roasting is the technique that enables the attacker to retrieve password hashes for AD users whose account options have been set to "Do not require Kerberos pre-authentication". This option relies on the old Kerberos authentication protocol, which allows authentication without a password. Once we obtain the hashes, we can try to crack it offline, and finally, if it is crackable, we got a password!

## Creating a list of users

`lookupsid.py` is a tool designed to enumerate Windows Security Identifiers (SIDs) and map them to their corresponding usernames or groups on a remote Windows machine. It facilitates the brute-forcing of SIDs to identify users and groups, which is particularly useful during penetration testing and red teaming exercises.

### Basic Usage

#Attacking_Machine 
The typical command syntax for using lookupsid.py is:

```
python3 lookupsid.py <domain>/<user>:<password>@<target>
```

In your case  create a file with the ourput:

```
/usr/share/doc/python3-impacket/examples/lookupsid.py vulnnet-rst.local/guest@<MACHINE IP > lookupsid_output.txt  
```

![[Screenshot_2025-10-10_05-06-28.png]]


### Extract the usernames

#Attacking_Machine 
Extract the account names (the part after `VULNNET-RST\`) and put them into `users.txt`.

```
 sed -n 's/.*\\\(.*\) (SidType.*/\1/p' lookupsid_output.txt > users.txt
```

![[Screenshot_2025-10-10_05-41-19.png]]


**Next step:** [[Gaining initial access]]