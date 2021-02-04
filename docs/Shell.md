# Shell

运行shell脚本

```shell
second supply 脚本: 
 ./run restart
run 脚本文件（shell文件）

cd-water 项目脚本：
sh shell.sh
```



公司服务器上面的 run shell 脚本 

```shell
#!/bin/bash
source /etc/profile

path=$(cd `dirname $0`; pwd)
dir_name=${path##/*/}
jar_name=`find $path -name *.jar`
logdir=/var/log/$dir_name
debugport=53221

start(){
    mkdir -p $logdir
    if [ ! -d $path/logs ];then
      ln -s $logdir $path/logs
    fi
    nohup java -jar $jar_name   --spring.profiles.active=dev  >$logdir/log.log 2>&1 &
    echo "start boot $dir_name"
}

debug(){
    if [ ! $debugport ]; then
      echo "未获取到debugport"
    else
      mkdir -p $logdir
      if [ ! -d $path/logs ];then
        ln -s $logdir $path/logs
      fi
      nohup java -jar -Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=4791 $jar_name  --spring.profiles.active=dev >$logdir/log.log 2>&1 &
    fi
}

stop(){
        ps -ef | grep "$jar_name" | grep -v grep |awk '{print $2}'| while read pid
        do
           kill  $pid
           sleep 3
        a=`jps -l | grep "$dir_name" | wc -l`
        if [ -n "$a" ] && [ "$a" = "1" ]
                then
                echo $a
                if [ $a == 1 ]
                        then
                        kill -9 $pid
                fi
        fi
        done
        echo "stop $dir_name"
}

status(){
    echo `ps -ef | grep "$dir_name" | grep -v grep`
}

update(){
    mkdir -p $path/update/.lib-tmp/BOOT-INF/lib
    cp $path/update/*.jar $path/update/.lib-tmp/BOOT-INF/lib
    jar -uvf0 $jar_name -C $path/update/.lib-tmp/ . && rm -rf $path/update/.lib-tmp
    echo "-------${jar_name##/*/} update success-------"
}

case "$1" in
    start)
        start
        ;;
    debug)
        debug
        ;;
    stop)
        stop
        ;;
    restart)
        stop
        sleep 3
        start
        ;;
    status)
        status
        ;;
    update)
        update
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status|update}"
        RETVAL=1
esac


```

