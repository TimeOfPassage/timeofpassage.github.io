# Shell脚本模板

# 容器运行

​`xxx.sh`​

```bash
help() {
        echo 'Usage: ./xxx.sh [start|stop|restart]'
}

start() {
    
}

stop() {
      
}

# start exec program
command=$1
case $command in
        start)
                echo 'starting...'
                start
                echo 'start finished'
                ;;
        stop)
                echo 'stoping...'
                stop
                echo 'stop finished'
                ;;
        restart)
                echo 'restarting...'
                stop
                start
                echo 'restart finished'
                ;;
        *)
                help
                ;;
esac
```
