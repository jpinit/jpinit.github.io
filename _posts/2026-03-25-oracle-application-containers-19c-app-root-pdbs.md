---
layout: post
filename: 2026-03-25-oracle-application-containers-19c-app-root-pdbs.md
title: "Oracle Application Containers 19c: Creating and Managing App Roots and PDBs"
date: 2099-01-01
#date: 2026-03-25
author: Jeremy Perkins
categories: oracle multitenant pdb
robots: noindex
hidden: true
#published: false
---

Oracle Application Containers are one of those features that sound simple on paper but are often misunderstood in practice.

If you are managing multiple PDBs that support the same application, keeping everything aligned becomes a problem:
- schema drift
- inconsistent deployments
- manual updates across environments

Application Containers solve this by allowing you to define the application once and propagate changes across PDBs.

This post walks through a simple working structure to create and manage:
- Application Root
- Application Seed
- Application PDB

---

## Creating the Application Root

The application root is where the application definition lives.

```sql
CREATE PLUGGABLE DATABASE app_root AS APPLICATION CONTAINER;
```

Open the new container:

```sql
ALTER PLUGGABLE DATABASE app_root OPEN;
```

Switch into the container:

```sql
ALTER SESSION SET CONTAINER=app_root;
```

---

## Creating the Application Seed

The application seed acts as a template for all application PDBs.

```sql
CREATE PLUGGABLE DATABASE app_seed FROM app_root AS SEED;
```

Open the seed:

```sql
ALTER PLUGGABLE DATABASE app_seed OPEN;
```

---

## Creating an Application PDB

Now create a PDB from the application seed.

```sql
CREATE PLUGGABLE DATABASE app_pdb1 FROM app_seed;
```

Open it:

```sql
ALTER PLUGGABLE DATABASE app_pdb1 OPEN;
```

---

## Installing the Application

All application objects must be created in the application root.

```sql
ALTER SESSION SET CONTAINER=app_root;

ALTER PLUGGABLE DATABASE app_root 
BEGIN INSTALL 'my_app' VERSION '1.0';

CREATE TABLE app_table (
  id NUMBER,
  name VARCHAR2(100)
);

ALTER PLUGGABLE DATABASE app_root 
END INSTALL;
```

---

## Synchronizing the Application PDB

The PDB will not automatically receive the application objects.

You must run SYNC:

```sql
ALTER SESSION SET CONTAINER=app_pdb1;

ALTER PLUGGABLE DATABASE app_pdb1 SYNC;
```

---

## Verifying the Application

Check application registration:

```sql
SELECT app_name, app_version, app_status 
FROM dba_applications;
```

Confirm container context:

```sql
SELECT sys_context('USERENV','CON_NAME') FROM dual;
```

Verify objects exist in the PDB:

```sql
SELECT table_name 
FROM user_tables 
WHERE table_name = 'APP_TABLE';
```

---

## Important Behavior Notes

A few things that are not obvious until you work with this:

- The application PDB will NOT have objects until SYNC is executed  
- PDBs can run different versions of the same application  
- All application changes must originate in the application root  
- Creating objects directly inside the PDB breaks consistency  

---

## Why This Matters

Application Containers are designed for:
- SaaS environments
- multi-tenant deployments
- controlled application versioning

Instead of managing each PDB independently, you define once and propagate.

---

## Final Thought

This is one of those features that becomes more valuable as your environment scales.

On a single PDB system, it may feel unnecessary.  
On a multi-PDB system, it becomes essential.

Like most Oracle features — simple concept, but execution matters.
