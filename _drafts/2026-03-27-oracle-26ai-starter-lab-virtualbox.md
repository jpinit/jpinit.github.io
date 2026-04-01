---
layout: post
filename: 2026-03-27-oracle-26ai-starter-lab-virtualbox.md
title: "Oracle 26ai Starter Lab: Building a Free, License-Compliant VirtualBox Environment"
date: 2026-03-27
author: Jeremy Perkins
categories: oracle lab virtualization
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

Oracle provides software under OTN for free personal use with strict limitations:

- Allowed:
  - Personal learning  
  - Non-commercial use  
  - Local lab environments  

- NOT allowed:
  - Production use  
  - Commercial/business use  
  - Any revenue-generating activity  

This lab must remain isolated and cannot be used for any business purpose.

The OTN license does not expire, but usage restrictions apply indefinitely.

Cloned VMs, snapshots, and backups are all subject to the same licensing terms.

---

## Lab Architecture

- Virtualization: Oracle VirtualBox  
- Database: Oracle 26ai  
- Type: Single Instance (non-RAC)  
- Container Database: DBTEST  
- PDBs: PDB1, PDB2  

---

## Create Virtual Machine

Example configuration:

- Name: OL9-26AI-SI-LAB01  
- Type: Linux  
- Memory: 8GB  
- CPU: 2–4 cores  
- Disk: 100GB (VDI, dynamically allocated)  
- Network: NAT  

---

## Install Oracle Linux

- Oracle Linux 9 minimal install  
- Set hostname:

```bash
hostnamectl set-hostname ol9-26ai-si-lab01
```

---

## Prepare OS for Oracle

There is currently no 26ai-specific preinstall package, so we use the 19c preinstall package for OS configuration.

```bash
dnf install -y oracle-database-preinstall-19c
```

---

## Install Oracle Database 26ai

Install software into:

```
/u01/app/oracle/product/26.0.0/dbhome_1
```

---

## Create Database Using DBCA

Use DBCA to create a container database:

```bash
dbca
```

Configuration:
- CDB name: DBTEST  
- Enable PDB creation  
- Create PDB1 and PDB2  

---

## Validate Database

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
- Export appliance via VirtualBox  

---

## Restore VM

Import appliance and start VM.

Validate database:

```sql
SELECT name FROM v$database;
```

---

## Naming Convention Strategy

Use consistent naming for all future labs.

Format:

```
<OS>-<DB>-<TYPE>-<FEATURE>-<ROLE>-<LAB#>
```

---

### Naming Matrix

| Component | Values |
|----------|--------|
| OS | OL8, OL9 |
| DB | 19C, 23AI, 26AI |
| TYPE | SI, RAC |
| FEATURE | NONE, DG, RHP |
| ROLE | PRIMARY, STANDBY, NODE1, NODE2 |
| LAB# | LAB01, LAB02 |

---

### Examples

| Name | Meaning |
|------|--------|
| OL9-26AI-SI-NONE-LAB01 | Base lab |
| OL9-26AI-SI-DG-PRIMARY-LAB01 | Data Guard primary |
| OL9-26AI-SI-DG-STANDBY-LAB02 | Data Guard standby |
| OL9-26AI-RAC-NODE1-LAB01 | RAC node 1 |
| OL9-26AI-RAC-NODE2-LAB01 | RAC node 2 |

---

## What Most DBAs Miss

- OTN license is not production-safe  
- Lab environments must remain isolated  
- Snapshots are not backups — export regularly  
- Naming becomes critical as labs scale  

---

## Real-World Notes

- Snapshot before major changes  
- Keep a clean baseline image  
- Clone instead of rebuild  
- Treat lab environments like production  

---

## Final Thought

A properly built lab becomes the foundation for everything that follows.

Build it once, build it right, and reuse it for every future Oracle test scenario.
