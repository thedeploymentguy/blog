---
title:  "Executing a program for N seconds then exiting:"
description: "Executing a program for N seconds then exiting"
date:   2017-08-08 16:30:00
categories: [scripts]
tags: [concepts]
---

If you need to execute a given command for a specified number of seconds Linux system. Then there is no command available in Linux for this purpose.

You can use following script for this purpose.

```sh
#!/bin/bash

cmd=$1
seconds=$2

echo "Executing: ${cmd} for $seconds seconds"
$cmd&

cmdpid=$!
sleep $seconds

if [ -d /proc/$cmdpid ]
then
  echo "terminating program PID:$cmdpid"
  kill -9 $cmdpid
fi

```

If you save this into a file called run_command_for_seconds_exits.sh you use it with:

``` sh
bash run_command_for_seconds_exits.sh "hdfs dfs -put 20170207020*.gz /data/TEST_1/" 50

#The above will run the command “hdfs dfs -put 20170207020*.gz /data/TEST_1/” 50″  for 50 seconds, then it will exit.
```

### Example

``` sh
[admin@GVSGLBNN-2 TEST]# bash /data/scripts/run_command_for_seconds_exits.sh “mv /data/HdfsDownloader/retention/20170315/20170315*.gz . ” 60

bash run_command_for_seconds_exits.sh “hdfs dfs -put 20170207020*.gz /data/TEST_1/” 50
```

You can download this script [run_command_for_seconds_exits.sh](https://github.com/thedeploymentguy/scriptsby-SK/blob/master/run_command_for_seconds_exits.sh)
