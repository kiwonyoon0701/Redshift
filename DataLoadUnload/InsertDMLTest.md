## Insert data into redshift using INSERT DML in SQL script

```
postgres@ip-10-100-1-108:~\$ time psql -h redshift-cluster-1.cfdt66dlx6mk.ap-northeast-2.redshift.amazonaws.com -U awsuser -d dev -p 5439 -a -f data10000.sql

Elapsed Time : 48min


root@oracle11g:/root# time psql -h redshift-cluster-2.cfdt66dlx6mk.ap-northeast-2.redshift.amazonaws.com -U awsuser -d dev -p 5439 -a -f data1000.sql  > /dev/null
Password for user awsuser:

real	4m42.394s
user	0m0.070s
sys	0m0.039s

```

redshift-cluster-2.cfdt66dlx6mk.ap-northeast-2.redshift.amazonaws.com:5439/dev

## Insert 1000 row data using INSERT DML in DBeaver

| Node type | # of Nodes | # of Files | Total Filesize | Total Rownum | Elapsed Time | Insert/sec |
| --------- | ---------- | ---------- | -------------- | ------------ | ------------ | ---------- |
| dc2.large | 8          | NA         | NA             | 1000         | 4 min        | 4row / s   |

## Insert 1000/10000 row data using INSERT DML in psql

| Node type | # of Nodes | # of Files | Total Filesize | Total Rownum | Elapsed Time | Insert/sec |
| --------- | ---------- | ---------- | -------------- | ------------ | ------------ | ---------- |
| dc2.large | 8          | NA         | NA             | 1000         | 4 min 42sec  | 4row / s   |
| dc2.large | 8          | NA         | NA             | 10000        | 48 min       | 4row / s   |
