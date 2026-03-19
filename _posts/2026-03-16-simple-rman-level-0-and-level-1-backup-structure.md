---
layout: post
title: "A Simple RMAN Level 0 and Level 1 Backup Script Structure"
date: 2026-03-16
categories: oracle rman backup
---
I've put together a set of simple RMAN level 0 and level 1 scripts that can be put in cron.
No need for a complicated backup to get started.

In my RMAN scripts repository, I use a straightforward structure with separate files for level 0 and level 1 backups. 
The directory includes these files:
- `backup_level_0.sh`
- `backup_level_0.rman`
- `backup_level_1.sh`
- `backup_level_1.rman`

This layout keeps the backup logic easy to follow and setup in cron.

Full script examples used in this article are available here: 
[JPInIT GitHub Repository – RMAN Scripts](https://github.com/jpinit/mystuff/tree/main/rman_scripts)



## Why separate the shell and RMAN files

A split between `.sh` and `.rman` files makes maintenance easier.

The shell scripts provide the wrapper around the backup job containing
- environment variables
- log file naming
- calling RMAN
- emailing

  TIP:  Setting OS parameter NLS_DATE_FORMAT will change the date format in RMAN output. Default is mm/dd/yyyy.

### backup_level_0.sh script: 

```
#!/bin/sh

# Specify a valid path to Oracle installation
export ORACLE_HOME=/app/oracle/product/19.3.0/dbhome_1
export PATH=/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:$ORACLE_HOME/bin

# Oracle instance (SID) name that is being exported
export ORACLE_SID=testdb

export DATE=`date '+%Y-%m-%d'`
export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"

export HOST=`uname -n`

REPORT_RECIPIENT="\
test@gmail.com\
"

rman target=/ log=/app/oracle/work/scripts/log/level0_output/testdb_full_bck_${DATE}.log @/app/oracle/work/scripts/rman/backup_level_0.rman

cp /app/oracle/work/scripts/log/level0_output/testdb_full_bck_${DATE}.log /backup/oracle/testdb/rman/full


# --------------------------------------------------
# Email log file
cat /app/oracle/work/scripts/log/level0_output/testdb_full_bck_${DATE}.log | \
mailx -s "$HOST: RMAN Backup Level-0 Report" $REPORT_RECIPIENT

exit
```

### backup_level_1.sh script: 

```
#!/bin/sh

# Specify a valid path to Oracle installation
export ORACLE_HOME=/app/oracle/product/19.3.0/dbhome_1
export PATH=/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:$ORACLE_HOME/bin

# Oracle instance (SID) name that is being exported
export ORACLE_SID=testdb

export DATE=`date '+%Y-%m-%d'`
export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"

export HOST=`uname -n`

REPORT_RECIPIENT="\
test@gmail.com\
"

rman target=/ log=/app/oracle/work/scripts/log/level1_output/testdb_incr_bck_${DATE}.log @/app/oracle/work/scripts/rman/backup_level_1.rman

cp /app/oracle/work/scripts/log/level1_output/testdb_incr_bck_${DATE}.log /backup/oracle/testdb/rman/incremental


# --------------------------------------------------
# Email log file
cat /app/oracle/work/scripts/log/level1_output/testdb_incr_bck_${DATE}.log | \
mailx -s "$HOST: RMAN Backup Level-1 Report" $REPORT_RECIPIENT

exit
```
<br>
<br>
<br>
<br>
The RMAN command file contains the actual backup commands. 

### RMAN Level 0 script: 
```
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

### RMAN Level 1 script:
```
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

## Why use backup as copy
Having a text backup of your spfile and controlfile can be invaluable in certain restore scenarios.
What if you loss your entire server, and cannot remember the layout of your database files.  Is it on /u0??
The backup controlfile will provide you the list of directories you need to recreate for the restore.
Other valuable information,  location of redo logs, archivelogs, and initialization parameters values.


## level 0 and level 1 backups explained
A level 0 backup establishes the baseline FULL backup set. 
A level 1 backup is the INCREMENTAL used to capture changes since the level 0 or the previous incremental.

I keep these in separate files so that backup intent is obvious at a glance.
Less time mentally reverse engineering the scripts later when troubleshooting or making modifications.


## Benefits of using this simple structure

This kind of layout has a few practical advantages:
- easy to troubleshoot
- easy to hand off to another DBA
- simple to schedule from cron or enterprise schedulers
- clear separation between OS script and RMAN commands


## Final thought

A good RMAN backup does not need to be complex. 
This backup process is easy to operate, review, and maintain.
Tweak as needed.

