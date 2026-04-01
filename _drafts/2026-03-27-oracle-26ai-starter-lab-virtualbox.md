---
layout: post
filename: 2026-03-27-oracle-26ai-starter-lab-virtualbox.md
title: "Oracle 26ai Starter Lab: Building a Free, License-Compliant VirtualBox Environment"
date: 2026-03-27
author: Jeremy Perkins
categories: oracle lab virtualization
---

Do you wish you had an Oracle environment to call your own?
Owning a personal Oracle database lab is a valuable tool for a DBA.

But can you have such a thing?  Yes.
Oracle Technology Network (OTN) provides non-commercial licensing.
Reference: https://www.oracle.com/downloads/licenses/standard-license.html

Walk-through : creating an Oracle 26ai lab using Oracle VirtualBox.

## Lab Architecture

- Virtualization: Oracle VirtualBox 7.2.6
- Operating System : Oracle Linux 9.7
- Database: Oracle 26ai  (v23.26)
- Type: Single Instance (non-RAC non-ASM)  
- Container Database: DBTEST  
- PDBs: PDB1, PDB2  


## Section I : Create Virtual Machine

Example configuration:

- Name: OL9-26AI-LAB01  
- Type: Linux  
- Memory: 8GB  
- CPU: 2–4 cores  
- Disk: 50GB (VDI, dynamically allocated)  
- Network: NAT  

1) Install Oracle VirtualBox
<img src="/assets/images/blog/starter-lab/1.1-Screenshot 2026-03-31 225724.png" width="600">

2) create VM  configuration

---

## Section II : Install Oracle Linux

- Oracle Linux 9 minimal install  
- Set hostname:

```bash
hostnamectl set-hostname ol9-26ai-lab01
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

---

## Final Thought

A properly built lab is a great foundation for DBAs to hone skills.
Take advantage of the VM snapshotting technology 
Build it once, build it right, and reuse it often.


- Backup your VMs (do not rely solely on snapshots)
- Keep a clean baseline image of your VM  
- Clone your base VM instead of rebuilding each time  
- Snapshot before making major changes.  
- VM Naming becomes critical as number of Vms grow

