**Пишем своего демона для Linux**

1) Создаем по  пути **/opt/checker.sh** скрипт с содержимым:

_#!/bin/bash_

_while true_

_ do_

_LIMIT=0.02_

_LAST=&quot;$(uptime | tail -c 5)&quot;_

_if (( $(echo &quot;$LAST &gt;= $LIMIT&quot; | bc -l) )) ; then_

_ echo &quot;ALERT at $(date)&quot; &gt;&gt; /var/log/checker_

_fi_

_sleep 10_

_done_

2) Превращаем его в исполняемый файл и проверяем работу:

**chmod u+x /opt/checker.sh**

**/opt/checker.sh**

**touch /var/log/checker**

**tail /var/log/checker**

3) Пишу в файл /etc/init.d/checker скрипт для демона:

_#!/bin/bash_

_# chkconfig: 2345 20 80_

_# description: checking load_

_# Source function library._

_. /etc/init.d/functions_

_ _

_case &quot;$1&quot; in_

_start)_

_   echo &quot;$(date) service checker started&quot; &gt;&gt; /var/log/checker_

_   /opt/checker.sh &amp;_

_   echo $!&gt;/var/run/checker.pid_

_   ;;_

_stop)_

_   echo &quot;$(date) service checker stopped&quot; &gt;&gt; /var/log/checker_

_   kill `cat /var/run/checker.pid`_

_   rm /var/run/checker.pid_

_   ;;_

_restart)_

_   $0 stop_

_   $0 start_

_   ;;_

_status)_

_   if [-e /var/run/checker.pid]; then_

_      echo checker is running, pid=`cat /var/run/checker.pid`_

_   else_

_      echo checker is NOT running_

_      exit 1_

_   fi_

_   ;;_

_\*)_

_   echo &quot;Usage: $0 {start|stop|status|restart}&quot;_

_esac_

_ _

_exit 0_

4) Делаю скрипт исполняемым и добавляю в автозапуск и проверяю его работу:

**sudo chmod u+x /etc/init.d/checker**

**chkconfig checker on**

**service checker start**

**service checker status**