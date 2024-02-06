# Push data received via SFTP to a S3 raw bucket

## Overview

* Know your final end state
* Describe current state
* Identify data starting point(s) and end point(s)
* Identify data intermediate boundary points

## Planning

### Current State

1. EC2 Instance running GlobalScape SFTP Server on Windows Server 2019 
2. Staff member copies files from the shared server folder to the fileshares raw data folder (OsmisShare). This is the archived/historic data. Years worth!
3. Staff member copies ONLY the new files for the month to be processed by SSIS to the fileshare folder for staging (OsmisStaging). This folder is read by SSIS for import.

Two risks that have been realized copying to staging
1. Staff copies to many files. SSIS doesn't look at files to load, it just loads everything. No error prevention with that aspect.
2. Staff forgets to copy data to OsmisStaging or misses selecting a file. This has happened, unfortunately, and was identified years later after processing had been completed.

