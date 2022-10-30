---
layout: post
title: "CSAW CTF Quals '18 Ldab - Web 50 Write-Up (LDAP Injection)"
excerpt: "CSAW CTF Qualification Round 2018 Web 50 Ldab Write-up - LDAP Injection"
categories: [hacking-write-ups]
comments: true
tag: ['cybersecurity', 'pentesting', 'infosec', 'csaw', 'write-up', 'LDAP injection']
image:
  feature: csaw-quals/csawlogo.png
  show: false
---

![ldap_chal][1]{:class="post-image center"}

This is a straight-forward company directory. First thing came to our mind was SQL injection without thinking too much about the challenge name (because it was a 50-pt challenge...). After trying a bunch of SQL injection quickly we suddenly realized this is LDAP injection (dah), so we put ```*``` as search query to confirmed this should be LDAP injection.

**LDAP Injection**:
> LDAP Injection is an attack technique used to exploit web sites that construct LDAP statements from user-supplied input.
> 
> Lightweight Directory Access Protocol (LDAP) is an open-standard protocol for both querying and manipulating X.500 directory services. 

The web app looks like this...
![chal_screenshot][2]{:class="post-image center"}

Since we weren't  really familiar with LDAP injection, a quick Google search gave us the answer. Magic.

**Payload: ```*)(uid=*))(|(uid=*```**

Source: [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LDAP%20injection][3]

![payload][4]{:class="post-image center"}

<br>
Then we got the flag.
![flag][5]{:class="post-image center"}

FLAG: flag{ld4p_inj3ction_i5_a_th1ng} indeed


[1]: {{ "/assets/upload/images/csaw-quals/ldab-chal.png" | absolute_url }}
[2]: {{ "/assets/upload/images/csaw-quals/ldab-app.png" | absolute_url }}
[3]: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LDAP%20injection
[4]: {{ "/assets/upload/images/csaw-quals/ldab-payload.png" | absolute_url }}
[5]: {{ "/assets/upload/images/csaw-quals/ldab-flag.png" | absolute_url }}
