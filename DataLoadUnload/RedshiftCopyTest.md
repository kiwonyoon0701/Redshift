## Copy data into redshift from S3

| Node type | # of Nodes | # of Files | Total Filesize | Total Rownum | DIST Key | Sort key | Elapsed Time |
| --------- | ---------- | ---------- | -------------- | ------------ | -------- | -------- | ------------ |
| dc2.large | 8          | 1          | 1.8G           | 9937969      | N/A      | N/A      | 44 sec       |
| dc2.large | 8          | 2          | 1.8G           | 9937969      | N/A      | N/A      | 22 sec       |
| dc2.large | 8          | 4          | 1.8G           | 9937969      | N/A      | N/A      | 12 sec       |
| dc2.large | 8          | 12         | 1.8G           | 9937969      | N/A      | N/A      | 9 sec        |

| Node type | # of Nodes | # of Files | Total Filesize | Total Rownum | DIST Key | Sort key | Elapsed Time |
| --------- | ---------- | ---------- | -------------- | ------------ | -------- | -------- | ------------ |
| dc2.large | 2          | 10         | 18G            | 99379690     | N/A      | N/A      | 220 sec      |
| dc2.large | 2          | 120        | 18G            | 99379690     | N/A      | N/A      | 214 sec      |
| dc2.large | 8          | 10         | 18G            | 99379690     | N/A      | N/A      | 87 sec       |
| dc2.large | 8          | 120        | 18G            | 99379690     | N/A      | N/A      | 59 sec       |

```
root@ip-10-100-1-108:/data/2015# ls -alh ./original/
total 1.8G
drwxr-xr-x 2 root root 4.0K Aug 31 15:19 .
drwxr-xr-x 3 root root 4.0K Aug 31 15:20 ..
-rw-r--r-- 1 root root  43M Apr 13  2015 201501-citibike-tripdata.csv
-rw-r--r-- 1 root root  30M Apr 13  2015 201502-citibike-tripdata.csv
-rw-r--r-- 1 root root  51M Jun 23  2015 201503-citibike-tripdata.csv
-rw-r--r-- 1 root root 121M Jun 23  2015 201504-citibike-tripdata.csv
-rw-r--r-- 1 root root 177M Jun 23  2015 201505-citibike-tripdata.csv
-rw-r--r-- 1 root root 140M Jul  9  2015 201506-citibike-tripdata.csv
-rw-r--r-- 1 root root 200M Sep 23  2015 201507-citibike-tripdata.csv
-rw-r--r-- 1 root root 217M Sep 23  2015 201508-citibike-tripdata.csv
-rw-r--r-- 1 root root 238M Oct  1  2015 201509-citibike-tripdata.csv
-rw-r--r-- 1 root root 227M Nov 11  2015 201510-citibike-tripdata.csv
-rw-r--r-- 1 root root 185M Dec  2  2015 201511-citibike-tripdata.csv
-rw-r--r-- 1 root root 151M Jan 14  2016 201512-citibike-tripdata.csv


root@ip-10-100-1-108:/data/2015# find ./original/ -type f|sort > ./filelist.txt
root@ip-10-100-1-108:/data/2015# cat filelist.txt
./original/201501-citibike-tripdata.csv
./original/201502-citibike-tripdata.csv
./original/201503-citibike-tripdata.csv
./original/201504-citibike-tripdata.csv
./original/201505-citibike-tripdata.csv
./original/201506-citibike-tripdata.csv
./original/201507-citibike-tripdata.csv
./original/201508-citibike-tripdata.csv
./original/201509-citibike-tripdata.csv
./original/201510-citibike-tripdata.csv
./original/201511-citibike-tripdata.csv
./original/201512-citibike-tripdata.csv

```

```
root@ip-10-100-1-108:/data/2015# ls -alh ./original/
total 1.8G
drwxr-xr-x 2 root root 4.0K Aug 31 15:19 .
drwxr-xr-x 3 root root 4.0K Aug 31 15:20 ..
-rw-r--r-- 1 root root 43M Apr 13 2015 201501-citibike-tripdata.csv
-rw-r--r-- 1 root root 30M Apr 13 2015 201502-citibike-tripdata.csv
-rw-r--r-- 1 root root 51M Jun 23 2015 201503-citibike-tripdata.csv
-rw-r--r-- 1 root root 121M Jun 23 2015 201504-citibike-tripdata.csv
-rw-r--r-- 1 root root 177M Jun 23 2015 201505-citibike-tripdata.csv
-rw-r--r-- 1 root root 140M Jul 9 2015 201506-citibike-tripdata.csv
-rw-r--r-- 1 root root 200M Sep 23 2015 201507-citibike-tripdata.csv
-rw-r--r-- 1 root root 217M Sep 23 2015 201508-citibike-tripdata.csv
-rw-r--r-- 1 root root 238M Oct 1 2015 201509-citibike-tripdata.csv
-rw-r--r-- 1 root root 227M Nov 11 2015 201510-citibike-tripdata.csv
-rw-r--r-- 1 root root 185M Dec 2 2015 201511-citibike-tripdata.csv
-rw-r--r-- 1 root root 151M Jan 14 2016 201512-citibike-tripdata.csv

root@ip-10-100-1-108:/data/2015# cat merge.sh
#!/bin/bash
# i : incremental var
# j : modular var for rotating target file
# k : var for filename

function do_merge()
{
MERGE_UNIT=$1
	DIR_NAME=$2

    DIR_PATH=/data/2015/s3/$DIR_NAME
    MYFILEPATH=$DIR_PATH/file
    rm -rf $DIR_PATH
    mkdir -p $DIR_PATH

    exec<filelist.txt
    i=1
    k=1
    while read LINE
    do
    	j=$((i%$MERGE_UNIT))
    	if [ $j -eq 0 ]
    	then
    		#cat $LINE >> $MYFILEPATH$k.csv
    		tail -n +2 $LINE >> $MYFILEPATH$k.csv # excludes first header line
    		echo 'FILE '$LINE' merge into file'$k'.csv'
    		k=$((k+1))
    	else
    		#cat $LINE >> $MYFILEPATH$k.csv
    		tail -n +2 $LINE >> $MYFILEPATH$k.csv # excludes first header line
    		echo 'FILE '$LINE' merge into file'$k'.csv'
    	fi
    i=$((i+1))
    done

}

do_merge 1 file-num-12
do_merge 2 file-num-6
do_merge 4 file-num-3
do_merge 12 file-num-1

root@ip-10-100-1-108:/data/2015# ./merge.sh
FILE ./original/201501-citibike-tripdata.csv merge into file1.csv
FILE ./original/201502-citibike-tripdata.csv merge into file2.csv
FILE ./original/201503-citibike-tripdata.csv merge into file3.csv
FILE ./original/201504-citibike-tripdata.csv merge into file4.csv
FILE ./original/201505-citibike-tripdata.csv merge into file5.csv
FILE ./original/201506-citibike-tripdata.csv merge into file6.csv
FILE ./original/201507-citibike-tripdata.csv merge into file7.csv
FILE ./original/201508-citibike-tripdata.csv merge into file8.csv
FILE ./original/201509-citibike-tripdata.csv merge into file9.csv
FILE ./original/201510-citibike-tripdata.csv merge into file10.csv
FILE ./original/201511-citibike-tripdata.csv merge into file11.csv
FILE ./original/201512-citibike-tripdata.csv merge into file12.csv
FILE ./original/201501-citibike-tripdata.csv merge into file1.csv
FILE ./original/201502-citibike-tripdata.csv merge into file1.csv
FILE ./original/201503-citibike-tripdata.csv merge into file2.csv
FILE ./original/201504-citibike-tripdata.csv merge into file2.csv
FILE ./original/201505-citibike-tripdata.csv merge into file3.csv
FILE ./original/201506-citibike-tripdata.csv merge into file3.csv
FILE ./original/201507-citibike-tripdata.csv merge into file4.csv
FILE ./original/201508-citibike-tripdata.csv merge into file4.csv
FILE ./original/201509-citibike-tripdata.csv merge into file5.csv
FILE ./original/201510-citibike-tripdata.csv merge into file5.csv
FILE ./original/201511-citibike-tripdata.csv merge into file6.csv
FILE ./original/201512-citibike-tripdata.csv merge into file6.csv
FILE ./original/201501-citibike-tripdata.csv merge into file1.csv
FILE ./original/201502-citibike-tripdata.csv merge into file1.csv
FILE ./original/201503-citibike-tripdata.csv merge into file1.csv
FILE ./original/201504-citibike-tripdata.csv merge into file1.csv
FILE ./original/201505-citibike-tripdata.csv merge into file2.csv
FILE ./original/201506-citibike-tripdata.csv merge into file2.csv
FILE ./original/201507-citibike-tripdata.csv merge into file2.csv
FILE ./original/201508-citibike-tripdata.csv merge into file2.csv
FILE ./original/201509-citibike-tripdata.csv merge into file3.csv
FILE ./original/201510-citibike-tripdata.csv merge into file3.csv
FILE ./original/201511-citibike-tripdata.csv merge into file3.csv
FILE ./original/201512-citibike-tripdata.csv merge into file3.csv
FILE ./original/201501-citibike-tripdata.csv merge into file1.csv
FILE ./original/201502-citibike-tripdata.csv merge into file1.csv
FILE ./original/201503-citibike-tripdata.csv merge into file1.csv
FILE ./original/201504-citibike-tripdata.csv merge into file1.csv
FILE ./original/201505-citibike-tripdata.csv merge into file1.csv
FILE ./original/201506-citibike-tripdata.csv merge into file1.csv
FILE ./original/201507-citibike-tripdata.csv merge into file1.csv
FILE ./original/201508-citibike-tripdata.csv merge into file1.csv
FILE ./original/201509-citibike-tripdata.csv merge into file1.csv
FILE ./original/201510-citibike-tripdata.csv merge into file1.csv
FILE ./original/201511-citibike-tripdata.csv merge into file1.csv
FILE ./original/201512-citibike-tripdata.csv merge into file1.csv
root@ip-10-100-1-108:/data/2015# ls -alh ./s3/\*
./s3/file-num-1:
total 1.8G
drwxr-xr-x 2 root root 4.0K Aug 31 15:22 .
drwxr-xr-x 6 root root 4.0K Aug 31 15:22 ..
-rw-r--r-- 1 root root 1.8G Aug 31 15:22 file1.csv

./s3/file-num-12:
total 1.8G
drwxr-xr-x 2 root root 4.0K Aug 31 15:22 .
drwxr-xr-x 6 root root 4.0K Aug 31 15:22 ..
-rw-r--r-- 1 root root 43M Aug 31 15:22 file1.csv
-rw-r--r-- 1 root root 227M Aug 31 15:22 file10.csv
-rw-r--r-- 1 root root 185M Aug 31 15:22 file11.csv
-rw-r--r-- 1 root root 151M Aug 31 15:22 file12.csv
-rw-r--r-- 1 root root 30M Aug 31 15:22 file2.csv
-rw-r--r-- 1 root root 51M Aug 31 15:22 file3.csv
-rw-r--r-- 1 root root 121M Aug 31 15:22 file4.csv
-rw-r--r-- 1 root root 177M Aug 31 15:22 file5.csv
-rw-r--r-- 1 root root 140M Aug 31 15:22 file6.csv
-rw-r--r-- 1 root root 200M Aug 31 15:22 file7.csv
-rw-r--r-- 1 root root 217M Aug 31 15:22 file8.csv
-rw-r--r-- 1 root root 238M Aug 31 15:22 file9.csv

./s3/file-num-3:
total 1.8G
drwxr-xr-x 2 root root 4.0K Aug 31 15:22 .
drwxr-xr-x 6 root root 4.0K Aug 31 15:22 ..
-rw-r--r-- 1 root root 243M Aug 31 15:22 file1.csv
-rw-r--r-- 1 root root 733M Aug 31 15:22 file2.csv
-rw-r--r-- 1 root root 800M Aug 31 15:22 file3.csv

./s3/file-num-6:
total 1.8G
drwxr-xr-x 2 root root 4.0K Aug 31 15:22 .
drwxr-xr-x 6 root root 4.0K Aug 31 15:22 ..
-rw-r--r-- 1 root root 72M Aug 31 15:22 file1.csv
-rw-r--r-- 1 root root 171M Aug 31 15:22 file2.csv
-rw-r--r-- 1 root root 317M Aug 31 15:22 file3.csv
-rw-r--r-- 1 root root 417M Aug 31 15:22 file4.csv
-rw-r--r-- 1 root root 465M Aug 31 15:22 file5.csv
-rw-r--r-- 1 root root 336M Aug 31 15:22 file6.csv

root@ip-10-100-1-108:/data/2015# aws s3 sync ./s3/ s3://citybike1-kiwony
upload: s3/file-num-12/file1.csv to s3://citybike1-kiwony/file-num-12/file1.csv
upload: s3/file-num-12/file10.csv to s3://citybike1-kiwony/file-num-12/file10.csv
upload: s3/file-num-12/file11.csv to s3://citybike1-kiwony/file-num-12/file11.csv
upload: s3/file-num-12/file2.csv to s3://citybike1-kiwony/file-num-12/file2.csv
upload: s3/file-num-12/file12.csv to s3://citybike1-kiwony/file-num-12/file12.csv
upload: s3/file-num-12/file4.csv to s3://citybike1-kiwony/file-num-12/file4.csv
upload: s3/file-num-12/file5.csv to s3://citybike1-kiwony/file-num-12/file5.csv
upload: s3/file-num-12/file6.csv to s3://citybike1-kiwony/file-num-12/file6.csv
upload: s3/file-num-12/file7.csv to s3://citybike1-kiwony/file-num-12/file7.csv
upload: s3/file-num-12/file8.csv to s3://citybike1-kiwony/file-num-12/file8.csv
upload: s3/file-num-12/file9.csv to s3://citybike1-kiwony/file-num-12/file9.csv
upload: s3/file-num-3/file1.csv to s3://citybike1-kiwony/file-num-3/file1.csv
upload: s3/file-num-1/file1.csv to s3://citybike1-kiwony/file-num-1/file1.csv
upload: s3/file-num-3/file3.csv to s3://citybike1-kiwony/file-num-3/file3.csv
upload: s3/file-num-6/file1.csv to s3://citybike1-kiwony/file-num-6/file1.csv
upload: s3/file-num-12/file3.csv to s3://citybike1-kiwony/file-num-12/file3.csv
upload: s3/file-num-6/file2.csv to s3://citybike1-kiwony/file-num-6/file2.csv
upload: s3/file-num-6/file3.csv to s3://citybike1-kiwony/file-num-6/file3.csv
upload: s3/file-num-6/file4.csv to s3://citybike1-kiwony/file-num-6/file4.csv
upload: s3/file-num-6/file6.csv to s3://citybike1-kiwony/file-num-6/file6.csv
upload: s3/file-num-3/file2.csv to s3://citybike1-kiwony/file-num-3/file2.csv
upload: s3/file-num-6/file5.csv to s3://citybike1-kiwony/file-num-6/file5.csv

root@ip-10-100-1-108:/data/2015# tail -1 ./s3/file-num-1/file1.csv
"395","12/31/2015 21:49:19","12/31/2015 21:55:55","3242","Schermerhorn St & Court St","40.69102925677968","-73.99183362722397","418","Front St & Gold St","40.70224","-73.982578","23411","Subscriber","1968","1"

create schema citybike;
drop table citybike.bikeinfo12;

create table citybike.bikeinfo12(
tripduration INTEGER,
startime TIMESTAMP,
endtime TIMESTAMP,
start_station INTEGER,
start_station_name VARCHAR(100),
start_latitude DECIMAL,
start_longtitude DECIMAL,
end_station INTEGER,
end_station_name VARCHAR(100),
end_latitude DECIMAL,
end_longtitiude DECIMAL,
bikeid INTEGER,
usertype VARCHAR(15),
birthyear VARCHAR(10),
gender SMALLINT
);

copy citybike.bikeinfo12
from 's3://citybike1-kiwony/file-num-12/'
credentials 'aws_access_key_id=;aws_secret_access_key='
format
delimiter ','
null as ''
TIMEFORMAT AS 'MM/DD/YYYY HH24:MI:SS'
REMOVEQUOTES;

18.823s
9sec

select query, elapsed, substring from svl_qlog order by query desc limit 5;
select query,starttime,endtime,elapsed,substring, concurrency_scaling_status_txt from svl_qlog where query=3313 ;

---

query starttime endtime elapsed substring concurrency_scaling_status_txt
3,313 2020-08-31 15:36:38 2020-08-31 15:36:47 9,179,381 copy citybike.bikeinfo12 from 's3://citybike1-kiwony/file-nu 0 - Ran on the main cluster

## select \* from svl_query_summary where query = 3313 order by stm, seg, step;

userid query stm seg step maxtime avgtime "rows" bytes rate_row rate_byte label is_diskbased workmem is_rrscan is_delayed_scan rows_pre_filter
100 3313 0 0 0 9095893 4326137 9937969 1841794397 1104218.0 204643821 scan tbl=109753 name=S3 f 0 f f 0
100 3313 0 0 1 9095893 4326137 9937969 0 1104218.0 0.0 parse f 0 f f 0
100 3313 0 0 4 9095893 4326137 9937969 1569371148 1104218.0 174374572 dist f 0 f f 0
100 3313 0 1 0 9115458 9112555 9937969 1570270692 1104218.0 174474521 scan tbl=14896 name=Internal Worktable f 0 f f 0
100 3313 0 1 2 9115458 9112555 9937969 0 1104218.0 0.0 project f 0 f f 0
100 3313 0 1 3 9115458 9112555 9937969 0 1104218.0 0.0 insert tbl=109753 f 0 f f 0
100 3313 0 1 4 9115458 9112555 16 128 1.0 14.0 aggr tbl=762 f 0 f f 0
100 3313 1 2 0 74 63 16 128 scan tbl=762 name=Internal Worktable f 0 f f 0
100 3313 1 2 1 74 63 16 0 return f 0 f f 0
100 3313 1 3 0 2004 2004 16 128 scan tbl=14901 name=Internal Worktable f 0 f f 0
100 3313 1 3 1 2004 2004 1 16 aggr tbl=766 f 0 f f 0
100 3313 2 4 0 44 44 1 16 scan tbl=766 name=Internal Worktable f 0 f f 0
100 3313 2 4 1 44 44 0 0 return f 0 f f 0

copy citybike.bikeinfo6
from 's3://citybike1-kiwony/file-num-6/'
credentials 'aws_access_key_id=;aws_secret_access_key='
format
delimiter ','
null as ''
TIMEFORMAT AS 'MM/DD/YYYY HH24:MI:SS'
REMOVEQUOTES;

12sec

select query, elapsed, substring from svl_qlog order by query desc limit 5;
select query,starttime,endtime,elapsed,substring, concurrency_scaling_status_txt from svl_qlog where query=3470 ;

---

query starttime endtime elapsed substring concurrency_scaling_status_txt
3,470 2020-08-31 15:46:20 2020-08-31 15:46:32 11,503,511 copy citybike.bikeinfo6 from 's3://citybike1-kiwony/file-num 0 - Ran on the main cluster

## select \* from svl_query_summary where query = 3313 order by stm, seg, step;

userid query stm seg step maxtime avgtime rows bytes rate_row rate_byte label is_diskbased workmem is_rrscan is_delayed_scan rows_pre_filter
100 3,470 0 0 0 11,421,316 2,839,928 9,937,969 1,841,794,397 903,451 167,435,854 scan tbl=109758 name=S3 f 0 f f 0
100 3,470 0 0 1 11,421,316 2,839,928 9,937,969 0 903,451 0 parse f 0 f f 0
100 3,470 0 0 4 11,421,316 2,839,928 9,937,969 1,569,890,448 903,451 142,717,313 dist f 0 f f 0
100 3,470 0 1 0 11,442,262 11,438,795 9,937,969 1,570,270,692 903,451 142,751,881 scan tbl=15664 name=Internal Worktable f 0 f f 0
100 3,470 0 1 2 11,442,262 11,438,795 9,937,969 0 903,451 0 project f 0 f f 0
100 3,470 0 1 3 11,442,262 11,438,795 9,937,969 0 903,451 0 insert tbl=109758 f 0 f f 0
100 3,470 0 1 4 11,442,262 11,438,795 16 128 1 11 aggr tbl=762 f 0 f f 0
100 3,470 1 2 0 93 68 16 128 [NULL] [NULL] scan tbl=762 name=Internal Worktable f 0 f f 0
100 3,470 1 2 1 93 68 16 0 [NULL] [NULL] return f 0 f f 0
100 3,470 1 3 0 1,142 1,142 16 128 [NULL] [NULL] scan tbl=15665 name=Internal Worktable f 0 f f 0
100 3,470 1 3 1 1,142 1,142 1 16 [NULL] [NULL] aggr tbl=766 f 0 f f 0
100 3,470 2 4 0 40 40 1 16 [NULL] [NULL] scan tbl=766 name=Internal Worktable f 0 f f 0
100 3,470 2 4 1 40 40 0 0 [NULL] [NULL] return f 0 f f 0

copy citybike.bikeinfo3
from 's3://citybike1-kiwony/file-num-3/'
credentials 'aws_access_key_id=;aws_secret_access_key='
format
delimiter ','
null as ''
TIMEFORMAT AS 'MM/DD/YYYY HH24:MI:SS'
REMOVEQUOTES;

22sec

select query, elapsed, substring from svl_qlog order by query desc limit 5;
select query,starttime,endtime,elapsed,substring, concurrency_scaling_status_txt from svl_qlog where query=3521 ;

---

query starttime endtime elapsed substring concurrency_scaling_status_txt
3,521 2020-08-31 15:49:01 2020-08-31 15:49:23 22,006,382 copy citybike.bikeinfo3 from 's3://citybike1-kiwony/file-num 0 - Ran on the main cluster

## select \* from svl_query_summary where query = 3521 order by stm, seg, step;

userid query stm seg step maxtime avgtime rows bytes rate_row rate_byte label is_diskbased workmem is_rrscan is_delayed_scan rows_pre_filter
100 3,521 0 0 0 21,920,405 3,327,890 9,937,969 1,841,794,397 473,236 87,704,495 scan tbl=109763 name=S3 f 0 f f 0
100 3,521 0 0 1 21,920,405 3,327,890 9,937,969 0 473,236 0 parse f 0 f f 0
100 3,521 0 0 4 21,920,405 3,327,890 9,937,969 1,570,085,340 473,236 74,765,968 dist f 0 f f 0
100 3,521 0 1 0 21,941,451 21,937,517 9,937,969 1,570,270,692 473,236 74,774,794 scan tbl=15933 name=Internal Worktable f 0 f f 0
100 3,521 0 1 2 21,941,451 21,937,517 9,937,969 0 473,236 0 project f 0 f f 0
100 3,521 0 1 3 21,941,451 21,937,517 9,937,969 0 473,236 0 insert tbl=109763 f 0 f f 0
100 3,521 0 1 4 21,941,451 21,937,517 16 128 0 6 aggr tbl=762 f 0 f f 0
100 3,521 1 2 0 87 70 16 128 [NULL] [NULL] scan tbl=762 name=Internal Worktable f 0 f f 0
100 3,521 1 2 1 87 70 16 0 [NULL] [NULL] return f 0 f f 0
100 3,521 1 3 0 1,247 1,247 16 128 [NULL] [NULL] scan tbl=15937 name=Internal Worktable f 0 f f 0
100 3,521 1 3 1 1,247 1,247 1 16 [NULL] [NULL] aggr tbl=766 f 0 f f 0
100 3,521 2 4 0 40 40 1 16 [NULL] [NULL] scan tbl=766 name=Internal Worktable f 0 f f 0
100 3,521 2 4 1 40 40 0 0 [NULL] [NULL] return f 0 f f 0

copy citybike.bikeinfo1
from 's3://citybike1-kiwony/file-num-1/'
credentials 'aws_access_key_id=;aws_secret_access_key='
format
delimiter ','
null as ''
TIMEFORMAT AS 'MM/DD/YYYY HH24:MI:SS'
REMOVEQUOTES;

44sec

select query, elapsed, substring from svl_qlog order by query desc limit 5;
select query,starttime,endtime,elapsed,substring, concurrency_scaling_status_txt from svl_qlog where query=3560 ;

---

query starttime endtime elapsed substring concurrency_scaling_status_txt
3,560 2020-08-31 15:50:43 2020-08-31 15:51:27 43,833,405 copy citybike.bikeinfo1 from 's3://citybike1-kiwony/file-num 0 - Ran on the main cluster

## select \* from svl_query_summary where query = 3560 order by stm, seg, step;

userid query stm seg step maxtime avgtime rows bytes rate_row rate_byte label is_diskbased workmem is_rrscan is_delayed_scan rows_pre_filter
100 3,560 0 0 0 43,749,451 2,734,992 9,937,969 1,841,794,397 231,115 42,832,427 scan tbl=109768 name=S3 f 0 f f 0
100 3,560 0 0 1 43,749,451 2,734,992 9,937,969 0 231,115 0 parse f 0 f f 0
100 3,560 0 0 4 43,749,451 2,734,992 9,937,969 1,570,190,156 231,115 36,516,050 dist f 0 f f 0
100 3,560 0 1 0 43,771,544 43,767,770 9,937,969 1,570,270,692 231,115 36,517,923 scan tbl=16136 name=Internal Worktable f 0 f f 0
100 3,560 0 1 2 43,771,544 43,767,770 9,937,969 0 231,115 0 project f 0 f f 0
100 3,560 0 1 3 43,771,544 43,767,770 9,937,969 0 231,115 0 insert tbl=109768 f 0 f f 0
100 3,560 0 1 4 43,771,544 43,767,770 16 128 0 2 aggr tbl=762 f 0 f f 0
100 3,560 1 2 0 119 69 16 128 [NULL] [NULL] scan tbl=762 name=Internal Worktable f 0 f f 0
100 3,560 1 2 1 119 69 16 0 [NULL] [NULL] return f 0 f f 0
100 3,560 1 3 0 1,171 1,171 16 128 [NULL] [NULL] scan tbl=16140 name=Internal Worktable f 0 f f 0
100 3,560 1 3 1 1,171 1,171 1 16 [NULL] [NULL] aggr tbl=766 f 0 f f 0
100 3,560 2 4 0 42 42 1 16 [NULL] [NULL] scan tbl=766 name=Internal Worktable f 0 f f 0
100 3,560 2 4 1 42 42 0 0 [NULL] [NULL] return f 0 f f 0

select tab.table_schema,
tab.table_name,
tinf.tbl_rows as rows
from svv_tables tab
join svv_table_info tinf
on tab.table_schema = tinf.schema
and tab.table_name = tinf.table
where tab.table_type = 'BASE TABLE'
and tab.table_schema not in('pg_catalog','information_schema')
and tinf.tbl_rows > 1
order by tinf.tbl_rows desc;

---

table_schema table_name rows
citybike bikeinfo12 9,937,969
citybike bikeinfo6 9,937,969
citybike bikeinfo3 9,937,969
citybike bikeinfo1 9,937,969
pg_internal redshift_auto_health_check_197793 4,000

```
