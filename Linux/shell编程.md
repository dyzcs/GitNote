# Shell 编程

## 1. Linux

**准备**

* 所有机器完成 ssh 安装和启用
* 所有机器上安装响应 JDK
* 所有机器上软件的安装位置应该相同

使用 centos 7 系统，在集群操作时需要在多台机器上使用展示信息，为了方便起见，完善一些 shell 脚本。

一下为我个人集群样例：s183，s184，s185。s183 为主，s184 和 s185 为从。

1. 文件传输

    首先下载 rsync

    ```shell
    sudo yum install rsync -y
    ```

    ```shell
    xsync.sh [filepath]
    
    #!/bin/bash
    #1 获取输入参数个数，如果没有参数，直接退出
    pcount=$#
    if((pcount==0)); then
    echo no args;
    exit;
    fi
    
    #2 获取文件名称
    p1=$1
    fname=`basename $p1`
    #echo fname=$fname
    
    #3 获取上级目录到绝对路径
    pdir=`cd -P $(dirname $p1); pwd`
    #echo pdir=$pdir
    
    #4 获取当前用户名称
    user=`whoami`
    
    #5 循环
    for((host=184; host<=185; host++)); do
    	echo ----------------- s$host -----------------
    	rsync -rvl $pdir/$fname $user@s$host:$pdir
    done
    ```

2. 在所有机器上执行同一个命令

    **例如**：在所有机器上完成 cat、ls、tar、jps 等操作。

    ```shell
    xcall.sh [params]
    
    #!/bin/bash
    
    params=$@
    for (( i=183 ; i<=184 ; i=$i+1 )) ; do
        echo ----------------- s$i $params -----------------
        ssh s$i "$params"
    done
    ```

    大数据集群部署在所有机器上，当检测集群是否启动成功或者需要显示所有机器上运行的任务时，可以使用 jps 进行查看，但是 jps 即使完成 JDK 环境变量安装，仍然不能使用 xcall.sh jps 进行查看。

    **针对 jps 进行完善**

    ```shell
    1.切换到root用户
    	$> su root
    
    2.创建符号链接 [/soft/jdk/bin/jps 为 jdk 安装位置]
    	$> xcall.sh "ln -sfT /soft/jdk/bin/jps /usr/local/bin/jps"
    	
    3.修改jps符号链接的owner [centos 是操纵集群的用户]
    	$> xcall.sh "chown -h centos:centos /usr/local/bin/jps"
    	
    4.查看所有主机的进程id
    	$> xcall.sh jps
    ```

## 2. Hadoop

Hadoop 启动、停止都需要输入若干条命令，这里将其写入一个脚本。

```shell
hdp.sh start / stop

#!/bin/bash
if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi
case $1 in
"start")
        echo "=================== start hadoop cluster ==================="

        echo "--------------- start hdfs ---------------"
        ssh s183 "/soft/hadoop/sbin/start-dfs.sh"
        echo "--------------- start yarn ---------------"
        ssh s183 "/soft/hadoop/sbin/start-yarn.sh"
        echo "--------------- start historyserver ---------------"
        ssh s184 "/soft/hadoop/sbin/mr-jobhistory-daemon.sh start historyserver"
;;
"stop")
        echo "=================== stop hadoop cluster ==================="

        echo "--------------- stop historyserver ---------------"
        ssh s184 "/soft/hadoop/sbin/mr-jobhistory-daemon.sh stop historyserver"
        echo "--------------- stop yarn ---------------"
        ssh s183 "/soft/hadoop/sbin/stop-yarn.sh"
        echo "--------------- stop hdfs ---------------"
        ssh s183 "/soft/hadoop/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac
```

**/soft/hadoop**：这个目录为 Hadoop 安装目录

在启动 HDFS 和 Yarn 的同时，还开启了 Hadoop historyserver -- 历史服务器，可根据自己需求和集群性能按需使用。

## 3. Zookeeper

Zookeeper 同样安装在不同的机器上，而且启动和停止均要到指定机器上进行操作，为了简化操作，将 Zookeeper 操作写入 zk.sh 脚本。

```shell
zk.sh start / stop / status

#!/bin/bash

case $1 in
"start"){
	for i in s183 s184 s185
	do
        echo ---------- zookeeper $i start ------------
		ssh $i "/soft/zookeeper/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in s183 s184 s185
	do
        echo ---------- zookeeper $i stop ------------    
		ssh $i "/soft/zookeeper/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in s183 s184 s185
	do
        echo ---------- zookeeper $i status ------------    
		ssh $i "/soft/zookeeper/bin/zkServer.sh status"
	done
};;
esac
```

## 4. Kafka

Kafka 启动和停止均要到指定机器上进行，且需要指定相关配置文件，完成 kf.sh 脚本，配置文件路径可以自行指定，方便完成 kafka 操作。

```shell
kf.sh start / stop

#! /bin/bash

case $1 in
"start"){
    for i in s183 s184 s185
    do
        echo " --------start $i Kafka-------"
        ssh $i "/soft/kafka/bin/kafka-server-start.sh -daemon /soft/kafka/config/server.properties"
    done
};;
"stop"){
    for i in s183 s184 s185
    do
        echo " --------stop $i Kafka-------"
        ssh $i "/soft/kafka/bin/kafka-server-stop.sh stop"
    done
};;
esac
```

## 5. Flume

```shell
f1.sh start / stop

#! /bin/bash

case $1 in
"start"){
        for i in s183 s184 s185
        do
                echo " --------start $i flume-------"
                ssh $i "nohup /soft/flume/bin/flume-ng agent --conf-file /soft/flume/conf/file-flume-kafka.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/soft/flume/log1.txt 2>&1  &"
        done
};;	
"stop"){
        for i in s183 s184 s185
        do
                echo " --------stop $i flume-------"
                ssh $i "ps -ef | grep file-flume-kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
        done

};;
esac
```

## 6. SuperSet

superset 为 Python 语言的一款可视化框架，需要机器上安装有 Python 3 环境，为了方便管理，在 centos 上可以安装 miniconda 3，方便管理 Python 版本和模块安装卸载。

此脚本完全针对 superset，如有需要可自行定制开发。

```shell
superset.sh start / stop / status

#! /bin/bash

case $1 in
"start"){
        for i in s183 s184 s185
        do
                echo " --------start $i flume-------"
                ssh $i "nohup /soft/flume/bin/flume-ng agent --conf-file /soft/flume/conf/file-flume-kafka.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/soft/flume/log1.txt 2>&1  &"
        done
};;	
"stop"){
        for i in s183 s184 s185
        do
                echo " --------stop $i flume-------"
                ssh $i "ps -ef | grep file-flume-kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
        done

};;
esac

[centos@s183 /home/centos/bin]$cat superset.sh 
#!/bin/bash

superset_status(){
    result=`ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | wc -l`
    if [[ $result -eq 0 ]]; then
        return 0
    else
        return 1
    fi
}
superset_start(){
        # 该段内容取自~/.bashrc，所用是进行conda初始化
        # >>> conda initialize >>>
        # !! Contents within this block are managed by 'conda init' !!
        __conda_setup="$('/soft/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
        if [ $? -eq 0 ]; then
            eval "$__conda_setup"
        else
            if [ -f "/soft/miniconda3/etc/profile.d/conda.sh" ]; then
                . "/soft/miniconda3/etc/profile.d/conda.sh"
            else
                export PATH="/soft/miniconda3/bin:$PATH"
            fi
        fi
        unset __conda_setup
        # <<< conda initialize <<<
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            conda activate superset ; gunicorn --workers 5 --timeout 120 --bind s183:8787 --daemon 'superset.app:create_app()'
        else
            echo "superset running"
        fi

}

superset_stop(){
    superset_status >/dev/null 2>&1
    if [[ $? -eq 0 ]]; then
        echo "superset not run"
    else
        ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | xargs kill -9
    fi
}


case $1 in
    start )
        echo "Superset start"
        superset_start
    ;;
    stop )
        echo "Superset stop"
        superset_stop
    ;;
    restart )
        echo "Superset restart"
        superset_stop
        superset_start
    ;;
    status )
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            echo "superset not run"
        else
            echo "superset running"
        fi
esac
```

