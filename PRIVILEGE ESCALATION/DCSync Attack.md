
## DCSync

**DCSync** is a **technique (and legitimate feature)** in **Active Directory (AD)** that allows one domain controller to **synchronize (replicate)** password data and other account information from another domain controller.

In normal operations, this is how domain controllers keep their data consistent.

However — in **offensive security or incident response contexts**, “DCSync” usually refers to **abusing** that replication process to **steal credentials**.

## In Simple Terms

DCSync = _“pretend to be a domain controller and ask another one for all user password hashes.”_

##  Legitimate Function

In a Windows domain, **domain controllers** replicate directory information among themselves using the **Directory Replication Service (DRS)** protocol.  
This includes:

- User accounts
    
- Group memberships
    
- Password hashes (NTLM, Kerberos keys, etc.)
    

They use the **DS-Replication-Get-Changes** and **DS-Replication-Get-Changes-All** permissions for that.

## DCSync Attack (Abuse)

Attackers who compromise a privileged account (like **Domain Admin**, **Enterprise Admin**, or **Administrator**) can **mimic a domain controller** and request replication data directly from another DC — **without touching the DC’s filesystem**.

Tools like:

- 🧰 **Mimikatz** (`lsadump::dcsync`)
    
- 🧰 **Impacket’s secretsdump.py**  
    can perform this attack.
    

It allows extraction of:

- NTLM password hashes
    
- Kerberos keys (krbtgt)
    
- Password history
    


## Why It’s Dangerous

With DCSync, an attacker can:

- Dump _every_ domain user’s credentials remotely
    
- Forge **Golden Tickets** (if they obtain the `krbtgt` hash)
    
- Maintain **persistence** indefinitely
    

All without deploying malware on the domain controller itself.

## Attacking

#Attacking_Machine  
Run Impacket’s `secretsdump.py` against `<MACHINE IP>` using the credentials you Ggo  to extract credential material from the target domain controller.

```
python3 /home/kali/tools/secretsdump.py -just-dc a-whitehat:bNdKVkjv3RR9ht@<MACHINE IP>
```

- `python3 .../secretsdump.py` — runs the Impacket `secretsdump` script (used for dumping credentials/secrets).
    
- `a-whitehat:bNdKVkjv3RR9ht@<MACHINE IP>` — the credentials used to authenticate: **username** `a-whitehat`, **password** `bNdKVkjv3RR9ht`, and **target IP** `10.10.92.47`.
    
- `-just-dc` — an option that tells `secretsdump` to perform actions focused on retrieving **domain controller secrets** (rather than attempting all possible ways of dumping every local or remote SAM/LSA secret). Practically, this usually means using replication/DRSUAPI or similar methods to obtain domain credentials from a DC.

### What `secretsdump` will try to do

Depending on the target and the privileges of `a-whitehat`, `secretsdump` can:

- Use remote APIs (SAMR, LSARPC, DRSUAPI / replication) to extract password material.
    
- Dump NTLM (and sometimes LM) hashes for domain accounts.
    
- Extract Kerberos keys (including the `krbtgt` account’s key) if available — obtaining `krbtgt` enables Golden Ticket attacks.
    
- Enumerate account RIDs, SIDs, and sometimes other metadata.
    

If the supplied account **has replication-like privileges** (or is a Domain Admin / has DCSync rights), `secretsdump` can pull a lot of domain account hashes remotely — similar to the DCSync technique.

### Required privileges / when it works

- If `a-whitehat` is just a normal user **without** elevated rights, `secretsdump -just-dc` will likely fail to extract domain hashes.
    
- If `a-whitehat` is a **privileged account** (Domain Admin, Enterprise Admin, or any account granted `Replicating Directory Changes` / `Replicating Directory Changes All` on the domain), the tool can request replication data and succeed.
    
- On non-DC targets, some techniques may fail; on an actual DC the DRS/replication approaches are most relevant.

![[Screenshot_2025-10-13_06-01-34.png]]

The Administrator's info is the most interesting for you.

![[Screenshot_2025-10-13_06-03-45.png]]

## WinRM

#Attacking_Machine 
Connect with WinRM.

```
evil-winrm -i <MACHINE IP> -u Administrator -H c2597747aa5e43022a3a3049a3c3b09d 
```

![[WinRM conection.png]]

**Next step:** [[System flag]]

