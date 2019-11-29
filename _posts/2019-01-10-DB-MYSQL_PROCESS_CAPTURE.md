---
layout: post
title:  "MySQL process capture script"
date:  2019-01-10
categories:
- DB
tags:
- MySQL
published: true
---
#### 간단히 사용할 수 있는 MySQL 프로세스 캡쳐 스크립트
- CPU 상태 확인 중 임계 값 이상 사용중이면 MySQL 프로세스를 로깅

```bash
#!/bin/sh
# DB Process List Capture for CPU Issue Check
# @Author : 윤순혁
SETUSER="root"
RUNNER=`whoami`
if [ $RUNNER != $SETUSER ] ;
   then echo "Deny Access : [ $RUNNER ]. Not $SETUSER" ;
   exit 0 ;
fi
# Variables
# CPU usage threshold
THRESHOLD=50.0
while true
do
        # CPU Usage
        STAT_TEMP=`mpstat 1 1 | tail -1`
        CURR_CPU_USAGE=`echo "$STAT_TEMP" | awk '{print 100-$11}'`
        CURR_IOWAIT_USAGE=`echo "$STAT_TEMP" | awk '{print $6}'`
        echo "CPU IDLE: $CURR_CPU_USAGE, I/OWAIT: $CURR_IOWAIT_USAGE"
        compare_result=`echo "$CURR_CPU_USAGE > $THRESHOLD" | bc`
        if [ $compare_result -gt 0 ]; then
                echo "WARN: CPU usage larger than threshhold(Usage: $CURR_CPU_USAGE, I/O WAIT: $CURR_IOWAIT_USAGE)" >> /root/capture_cpu/process_capture.log
                echo "Process date : `date +%F\ %H:%M:%S`" >> /root/capture_cpu/process_capture.log
                TOP_PROCESS_LIST=`ps -e -o %cpu,%mem,cmd -orss=,args= | sort -k1,1n -b | tail -n 4`
                echo "Top Usage: $TOP_PROCESS_LIST" >> /root/capture_cpu/process_capture.log
                echo "DB Process List" >> /root/capture_cpu/process_capture.log
                mysql -uroot -pPASSWORD -e "show full processlist;" >> /root/capture_cpu/process_capture.log
                echo "" >> /root/capture_cpu/process_capture.log
                echo "Top I/O: " >> /root/capture_cpu/process_capture.log
                dstat --nocolor --top-io --top-bio 1 1 >> /root/capture_cpu/process_capture.log
                echo "" >> /root/capture_cpu/process_capture.log
        fi
done
```