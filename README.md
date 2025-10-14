## Quick summary of exploited weaknesses

- **ASREP-Roasting**: accounts marked _Do not require Kerberos pre-authentication_ — these allow attackers to request an AS-REP response that can be cracked offline.
    
- **Plaintext credentials** (scripts stored on SMB shares) — attackers find passwords in files and escalate.
    
- **Exposed SMB / lax share permissions** — accessible shares allow enumeration and file downloads.

## Mitigations (priority + concrete steps)

### 1) Fix ASREP-Roasting — remove the “Do not require preauthentication” flag

Nobody should have this flag set except highly controlled service accounts (and even then, avoid it if possible).

PowerShell — list vulnerable accounts:

```
# list accounts with 'Do not require preauthentication' (flag 0x00400000 = 4194304) 
Get-ADUser -Filter * -Properties userAccountControl |   Where-Object { ($_.userAccountControl -band 4194304) -ne 0 } |   
Select-Object SamAccountName,UserPrincipalName
```


PowerShell — disable the flag on a user:

```
# example: disable on account 't-skid' 
$u = Get-ADUser t-skid -Properties userAccountControl 
Set-ADUser t-skid -Replace @{userAccountControl = ($u.userAccountControl -band (-bnot 4194304))}
```

Recommendation: inventory all accounts with the flag, then remove it en masse or review service accounts individually.

---

### 2) Enforce strong passwords and MFA for admins/service accounts

- Strong password policy (length, complexity, banned lists).
    
- Require MFA/2FA for privileged accounts.
    
- Rotate service account passwords and rotate the `krbtgt` account periodically if appropriate.
    

---

## 3) Remove plaintext credentials / audit repositories and shares

- Search file shares for patterns (`password`, `passwd`, `pwd`, `user`) in scripts and text files.
    
```
# quick example on a Linux file server 
grep -RIn --exclude-dir={.git,node_modules} -e "password" -e "pwd" /path/to/share_root
```


- Replace embedded credentials with managed identities, gMSAs, or a secrets manager (HashiCorp Vault, Azure Key Vault).
    
- Deploy DLP or automated scanning to catch credentials in files.
    

---

### 4) Harden SMB and share permissions

- Remove anonymous/guest shares; disable guest access.
    
- Apply least-privilege ACLs on shares (only allow read/write to required users).
    
- Enable SMB signing and, where supported, SMB encryption.
    
- Block unnecessary SMB access network-wide with firewalls (don’t expose DCs to general user networks).
    

---

### 5) Restrict and audit remote services (WinRM, RPC, LDAP)

- Limit WinRM to approved admin hosts.
    
- Ensure LDAP is restricted and use LDAPS where possible; avoid anonymous LDAP queries.
    
- Close RPC/NetBIOS services on perimeter segments unless required.
    

---

### 6) Audit and remove unnecessary AD permissions and delegations

- Audit AD ACLs for who has `Replicating Directory Changes` and related replication permissions.
    
- Remove overly-broad delegations; use tightly controlled admin groups.
    
- Use BloodHound (internally) to identify risky privilege paths.
    

---

### 7) Detection & logging (what to watch)

- Enable advanced Kerberos and Directory Service auditing to log auths and directory access.
    
- Monitor for AS-REP indicators (requests/responses that suggest preauth bypass attempts).
    
- Monitor SMB for unusual activity: mass downloads, access to `NETLOGON`, `SYSVOL`, or script directories.
    
- SIEM rules to flag:
    
    - Rapid LDAP enumeration activity.
        
    - Non-admin accounts accessing script shares.
        
    - Use of known enumeration tools (e.g., `crackmapexec`, `rpcclient`) from non-admin hosts.
        
- Useful Windows event IDs to correlate: Kerberos events (4768/4769/4770/4771), Directory Service Access (4662), Account Logon (4624/4625). Complement with Sysmon for process/network telemetry.
    

---

### 8) Network segmentation & DC hardening

- Place DCs in separate VLANs and strictly control which hosts can reach Kerberos/LDAP/SMB ports.
    
- Use jump hosts for admin tasks; keep management hosts isolated.
    
- Keep Windows Server and DCs patched.
    

---

### 9) Post-compromise inventory & remediation

- If credentials are found compromised: rotate immediately, force resets, and perform retrospective log review.
    
- Look for persistence: rogue accounts, scheduled tasks, services using compromised credentials.
    
- If `krbtgt` is exposed or suspected, follow Microsoft’s two-step `krbtgt` reset process to invalidate Golden Tickets.