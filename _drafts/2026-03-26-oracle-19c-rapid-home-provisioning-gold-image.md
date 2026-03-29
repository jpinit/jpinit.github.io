---
layout: post
filename: 2026-03-26-oracle-19c-rapid-home-provisioning-gold-image.md
title: "Oracle 19c Rapid Home Provisioning: Building and Deploying Gold Images"
date: 2026-03-26
author: Jeremy Perkins
categories: oracle rhp gold-image
---

Rapid Home Provisioning (RHP) is one of those features that sounds like overkill until you have to manage multiple Oracle Homes across servers and environments.

At some point, patching stops being a technical task and becomes a consistency problem:
- different patch levels across servers
- manual installs that drift over time
- difficulty reproducing a known good environment

Gold images solve this by allowing you to standardize and deploy Oracle Homes in a controlled and repeatable way.

---

## What This Solves

- Inconsistent Oracle Home patch levels across environments  
- Time-consuming manual installs and patching  
- Difficulty maintaining a known good baseline  
- Environment drift between servers  

---

## How It Works

Rapid Home Provisioning allows you to:
- Capture an existing Oracle Home as a gold image  
- Store it centrally  
- Deploy it to target servers  

---

## Implementation

```bash
rhpctl add image -image db19c_gold_image \
  -path /u01/app/oracle/product/19.0.0/dbhome_1 \
  -imagetype ORACLEDBSOFTWARE
```

```bash
rhpctl query image
```

```bash
rhpctl deploy image -image db19c_gold_image \
  -targetnode server1 \
  -oraclehome /u01/app/oracle/product/19.0.0/dbhome_gold
```

---

## Validation

```bash
opatch lspatches
```

Confirm patch level matches the source Oracle Home.

---

## What Most DBAs Miss

- Gold images do not update automatically  
- You must rebuild after patching  
- Old images can reintroduce issues  

---

## Real-World Notes

- Build images only after validation  
- Use consistent naming  
- Avoid mixing manual and RHP-managed homes  

---

## Final Thought

Rapid Home Provisioning is less about speed and more about control.

Consistency across environments is where the real value comes from.
