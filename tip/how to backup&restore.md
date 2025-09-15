### Backup

```
 > BACKUP TABLE `default`.trips TO Disk('backups', 'trips.zip');
 
 > BACKUP TABLE `default`.trips TO Disk('backups', 'async_trips.zip') ASYNC;

 > Drop table trips;

 > RESTORE TABLE `default`.trips FROM Disk('backups', 'trips.zip')
   SETTINGS allow_non_empty_tables=true;
```

### Restore with a new renamed table

```
 > RESTORE TABLE `default`.trips AS `default`.trips2 FROM Disk('backups', 'trips.zip');
```

### Incremental backup

```
 > BACKUP TABLE `default`.trips TO Disk('backups', 'incremental-a3.zip')
   SETTINGS base_backup = Disk('backups', 'trips.zip')
```
  
### Restore table from base+incremental

```
 > RESTORE TABLE `default`.trips AS `default`.ted FROM Disk('backups', 'incremental-a3.zip');
 
  
 > select count(*) from `default`.trips;
 > select count(*) from `default`.ted;
```

### Check the backup status and progress

```
 > SELECT * from system.backups where id='f0c6186a-d951-41ee-ac12-b9c940764cfe' format Vertical;
```

### Backup to Object Storage like(S3, minio,etc)

```
CREATE TABLE data
(
    `key` Int,
    `value` String,
    `array` Array(String)
)
ENGINE = MergeTree
ORDER BY tuple();

INSERT INTO data SELECT *
FROM generateRandom('key Int, value String, array Array(String)')
LIMIT 1000;

select * from `default`.data;
```

### Backup to S3 or comparable storage like minio

```
 > BACKUP TABLE `default`.data TO S3('http://1.1.1.1:9199/[name-of-ted-bucket]/ted_backup', 'aaa', 'bbbb');

 > INSERT INTO data SELECT *
   FROM generateRandom('key Int, value String, array Array(String)')
   LIMIT 100
   
 > BACKUP TABLE `default`.data TO S3('http://1.1.1.1:9199/[name-of-ted-bucket]/ted_incremental', 'aaa', 'bbb') 
   SETTINGS base_backup = S3('http://1.1.1.1:9199/[name-of-ted-bucket]/ted_backup', 'aaa', 'bbbb');
   
```
 
### Restore from S3 where was being backup by incremental

```
 > RESTORE TABLE [database].[table] AS data3 FROM S3('http://1.1.1.1:9199/[name-of-ted-bucket]/ted_incremental', 'aaaa', 'bbb') 

 > SELECT count(*) from `default`.data;

 > SELECT count(*) from `default`.data3;

 > DROP table `default`.data3
 
 > SELECT count(*) from `default`.data3;
```
 
