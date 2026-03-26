---
layout: post
title: "Oracle Application Containers 19c: Creating and Managing App Roots and PDBs"
date: 2026-03-25
categories: oracle rdbms dba
tags: oracle multitenant application-container pdb cdb 19c
---

## Summary


Learn how to create and manage Oracle Application Containers in 19c, including application root, seed, and PDB lifecycle using real DBA commands with validation.

---

## Problem Statement

Managing multiple PDBs for the same application introduces challenges:

- Keeping schema versions consistent  
- Deploying changes across many PDBs  
- Avoiding configuration drift  

In real environments (especially SaaS or multi-tenant deployments), manually maintaining consistency across PDBs becomes error-prone and difficult to scale.

Oracle Application Containers solve this problem by centralizing application definition and lifecycle management.

---

## Environment

- Oracle Database: 19c  
- Architecture: Multitenant (CDB)  
- OS: Linux  

---

## Solution Overview

Application Containers introduce three key components:

- **Application Root** – central container where the application is defined  
- **Application Seed** – template used to create consistent PDBs  
- **Application PDBs** – databases that run the application  

This allows controlled deployment and versioning across multiple PDBs.

---

## Step-by-Step Implementation

### Step 1: Create Application Root

```sql
CREATE PLUGGABLE DATABASE app_root AS APPLICATION CONTAINER;
