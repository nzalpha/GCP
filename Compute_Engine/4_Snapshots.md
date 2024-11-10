# Snapshots
Primary backup mechanism, when instances/disk are crashed these backups are needed to restore to the previous state.

Priodic/Point in Time Backup: Taking backup at particular point in time. Example take backup at 11:10 am pst, you can restore to 11:10 am data.

**Backups are Incremental Backup**
   - Diskname   dsize   dusage      snapshot
      d1        10gb    1gb         s1
      d1        10gb    2gb         s2
      d1        10gb    2.5gb       s3
      In S2 the extra 1gb is taken as backup
      In S3 the extra .5gb is taken as backup

If S2 is deleted, S3 will get snapshot of S2 as well.


## you can create Snapshots from Compute Engine Service

We create Snapshot1 of disk → us-central1; the snapshot size is 1.5 gb
After couple of days the disk will be increasing to 2 gb
Now we create Snapshot2 of same disk-> us-central1 the snapshot size will be .5 gb
Lets assume we are creating snapshot3 of same disk in asia-southeast in different region, the snapshot size will be 3.5 gb