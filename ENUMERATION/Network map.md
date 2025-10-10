Use *nmap* to see the services and ports on the victim machine.
```
sudo nmap -sSVC -p- -oA nmap_full -Pn  <MACHINE IP>
```

![[Screenshot_2025-10-08_05-48-23.png]]

There are many ports opened, so, you need to do more enumeration.

**Next step:** [[enum4linux-ng]]
