---
layout: post
title: "A Simple RMAN Level 0 and Level 1 Backup Script Structure"
date: 2026-03-16
categories: oracle rman backup
---
I've put together a set of simple RMAN level 0 and level 1 scripts that can be put in cron.
No need for a complicated backup to get started.

In my RMAN scripts repository, I use a straightforward structure with separate files for level 0 and level 1 backups. 
The directory currently includes these files:

- `backup_level_0.sh`
- `backup_level_0.rman`
- `backup_level_1.sh`
- `backup_level_1.rman`

This layout keeps the backup logic easy to follow and setup in cron.

Full script examples used in this article are available here:
    JPinIT GitHub Repository  
    https://github.com/jpinit/mystuff/tree/main/rman_scripts


## Why separate the shell and RMAN files

A split between `.sh` and `.rman` files makes maintenance easier.

The shell scripts provide the wrapper around the backup job containing

- environment variables
- log file naming
- calling RMAN
- emailing

The RMAN command file contains the actual backup commands. 

RMAN Level 0 script: 
```sql
run {
    allocate channel disk1 device type disk MAXPIECESIZE 20G format '/backup/oracle/testdb/rman/full/%U_%T';
    crosscheck archivelog all;
    delete noprompt expired archivelog all;
    backup incremental level 0 database
    tag 'Full_Tlb_Dfile_0_testdb'
    archivelog from time 'SYSDATE-1'
    tag 'Full_Archive_testdb';
    backup as copy spfile format '/backup/oracle/testdb/rman/full/spfile%d_C_%T_%u';
    backup as copy current controlfile format '/backup/oracle/testdb/rman/full/controlfile_full_ctl%d_C_%T_%u';
}
```
RMAN Level 1 script:
```sql
run {
    allocate channel disk1 device type disk MAXPIECESIZE 20G format '/backup/oracle/testdb/rman/incremental/%U_%T';
    crosscheck archivelog all;
    delete noprompt expired archivelog all;
    backup incremental level 1 database
    tag 'Incr_Tlb_Dfile_1_testdb'
    archivelog from time 'SYSDATE-2'
    tag 'Incr_Archive_testdb';
    backup as copy spfile format '/backup/oracle/testdb/rman/incremental/spfile%d_C_%T_%u';
    backup as copy current controlfile format '/backup/oracle/testdb/rman/incremental/controlfile_incr_ctl%d_C_%T_%u';
}
```

## Why separate level 0 and level 1 backups

There is also value in keeping level 0 and level 1 backups in separate files.

A level 0 backup establishes the baseline backup set. A level 1 backup is then used to capture changes since the baseline or the previous incremental, depending on the backup strategy in use.

By separating them into different files, the backup intent is obvious at a glance. This reduces guesswork when reviewing jobs, logs, or scheduler entries later.

## Benefits of a simple structure

This kind of layout has a few practical advantages:

- easy to troubleshoot
- easy to hand off to another DBA
- simple to schedule from cron or enterprise schedulers
- clear separation between operating system logic and RMAN logic

It is not meant to be a complete enterprise backup framework. It is meant to be understandable and maintainable.

## Final thought

A good RMAN design does not always need to be complex. 
In many cases, a clean and predictable script structure is more valuable
than an overengineered solution.


If the backup files are named clearly and the shell wrapper is kept separate from the RMAN commands, the result is a backup process that is easier to operate, review, and maintain.
