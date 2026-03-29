---
layout: post
filename: 2026-03-27-oracle-26ai-starter-lab-virtualbox.md
title: "Oracle 26ai Starter Lab: Building a Free, License-Compliant VirtualBox Environment"
date: 2099-01-01
#date: 2026-03-27
author: Jeremy Perkins
categories: oracle lab virtualization
robots: noindex
hidden: true
#published: false
---

Building a personal Oracle lab is one of the most valuable things you can do as a DBA, but it must be done correctly to avoid violating licensing.

This post walks through creating a fully functional Oracle 26ai lab using Oracle VirtualBox, while staying compliant with Oracle Technology Network (OTN) licensing.

This lab will serve as the foundation for all future posts (Data Guard, RMAN, RHP, etc).

---

## What This Solves

- Safe Oracle learning environment without licensing risk  
- Repeatable lab build for future testing  
- Foundation for advanced configurations (Data Guard, RAC, etc)  

---

## Licensing (READ THIS FIRST)

Oracle provides software under OTN for **free personal use** with strict limitations:

- Allowed:
  - Personal learning  
  - Non-commercial use  
  - Local lab environments  

- NOT allowed:
  - Production use  
  - Commercial/business use  
  - Any revenue-generating activity  

There is no enforced expiration, but usage must remain within these terms.

---

## Lab Architecture

- Virtualization: Oracle VirtualBox  
- Database: Oracle 26ai  
- Type: Single Instance (non-RAC)  
- Container Database: DBTEST  
- PDBs: PDB1, PDB2  

---

## Install Oracle VirtualBox

Download and install:

https://www.virtualbox.org

---

## Create Virtual Machine

Example configuration:

- Name: OL9-26AI-SI-LAB01  
- Type: Linux  
- Memory: 8GB (minimum)  
- CPU: 2–4 cores  
- Disk: 100GB  

---

## Install Oracle Linux

- Use Oracle Linux 9 ISO  
- Minimal install is sufficient  
- Set hostname:

```
hostnamectl set-hostname ol9-26ai-si-lab01
```

---

## Install Oracle Database 26ai

Run preinstall:

```bash
dnf install -y oracle-database-preinstall-19c
```

Install database software (example path):

```bash
/u01/app/oracle/product/26.0.0/dbhome_1
```

---

## Create Container Database

```sql
CREATE DATABASE DBTEST
ENABLE PLUGGABLE DATABASE;
```

---

## Create PDBs

```sql
CREATE PLUGGABLE DATABASE pdb1 ADMIN USER pdbadmin IDENTIFIED BY password;
CREATE PLUGGABLE DATABASE pdb2 ADMIN USER pdbadmin IDENTIFIED BY password;
```

Open PDBs:

```sql
ALTER PLUGGABLE DATABASE ALL OPEN;
```

---

## Validation

```sql
SELECT name, open_mode FROM v$pdbs;
```

Expected:
- PDB1 open  
- PDB2 open  

---

## VM Backup (Critical Step)

Shut down VM:

```bash
shutdown -h now
```

On host:

- Copy entire VM directory  
OR  
- Use VirtualBox export:

```
File → Export Appliance
```

This allows full restore of your lab environment.

---

## Restore VM

Import appliance:

```
File → Import Appliance
```

Start VM and verify database:

```sql
SELECT name FROM v$database;
```

---

## Naming Convention Strategy

Use consistent naming for all future labs.

### Format

```
<OS>-<DB>-<TYPE>-<FEATURE>-<LAB#>
```

---

### Examples

| Name | Meaning |
|------|--------|
| OL9-26AI-SI-LAB01 | Linux, 26ai, Single Instance |
| OL9-26AI-RAC-LAB01 | RAC cluster |
| OL9-26AI-SI-DG-LAB01 | Data Guard primary |
| OL9-26AI-SI-DG-LAB02 | Data Guard standby |

---

## What Most DBAs Miss

- OTN license is NOT production-safe  
- Lab environments must remain isolated  
- Snapshots are not backups — export VM regularly  
- Naming matters as lab complexity grows  

---

## Real-World Notes

- Always snapshot BEFORE major changes  
- Keep one “clean baseline” VM  
- Clone instead of rebuilding  
- Treat lab like production (naming, structure, discipline)  

---

## Final Thought

A properly built lab is not just a learning tool — it becomes your personal testing platform for every Oracle feature you work with.

Done correctly, this one VM becomes the foundation for everything that follows.
