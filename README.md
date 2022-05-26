# logs

# Задание

1. в вагранте поднимаем 2 машины web и log
2. на web поднимаем nginx
3. на log настраиваем центральный лог сервер на любой системе на выбор
journald;
rsyslog;
elk.

4. настраиваем аудит, следящий за изменением конфигов нжинкса Все критичные логи с web должны собираться и локально и удаленно. Все логи с nginx должны уходить на удаленный сервер (локально только критичные). Логи аудита должны также уходить на удаленную систему. Формат сдачи ДЗ - vagrant + ansible
развернуть еще машину elk
таким образом настроить 2 центральных лог системы elk и какую либо еще;
в elk должны уходить только логи нжинкса;
во вторую систему все остальное.

# Решение 

Решение первого задания представлено в виде ansible playbook - main.yaml.

После развертывания машин и выполения плейбука логи поступают на сервер. 
Для проверки перехожу по несуществующему пути 192.168.56.10/pp в браузере и меняю права на файл nginx.conf ```chmod +x /etc/nginx/nginx.conf```

```
[root@log ~]# cat /var/log/rsyslog/web/nginx_access.log
May 22 19:37:52 web nginx_access: 192.168.56.1 - - [22/May/2022:19:37:52 +0300] "GET / HTTP/1.1" 200 4833 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:37:53 web nginx_access: 192.168.56.1 - - [22/May/2022:19:37:53 +0300] "GET /img/centos-logo.png HTTP/1.1" 200 3030 "http://192.168.56.10/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:37:53 web nginx_access: 192.168.56.1 - - [22/May/2022:19:37:53 +0300] "GET /img/html-background.png HTTP/1.1" 200 1801 "http://192.168.56.10/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:37:53 web nginx_access: 192.168.56.1 - - [22/May/2022:19:37:53 +0300] "GET /img/header-background.png HTTP/1.1" 404 3650 "http://192.168.56.10/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:37:53 web nginx_access: 192.168.56.1 - - [22/May/2022:19:37:53 +0300] "GET /favicon.ico HTTP/1.1" 404 3650 "http://192.168.56.10/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:38:09 web nginx_access: 192.168.56.1 - - [22/May/2022:19:38:09 +0300] "GET /pp HTTP/1.1" 404 3650 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:38:09 web nginx_access: 192.168.56.1 - - [22/May/2022:19:38:09 +0300] "GET /nginx-logo.png HTTP/1.1" 200 368 "http://192.168.56.10/pp" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
May 22 19:38:09 web nginx_access: 192.168.56.1 - - [22/May/2022:19:38:09 +0300] "GET /poweredby.png HTTP/1.1" 200 368 "http://192.168.56.10/pp" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0"
[root@log ~]# cat /var/log/rsyslog/web/nginx_error.log
May 22 19:37:53 web nginx_error: 2022/05/22 19:37:53 [error] 4369#4369: *2 open() "/usr/share/nginx/html/img/header-background.png" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /img/header-background.png HTTP/1.1", host: "192.168.56.10", referrer: "http://192.168.56.10/"
May 22 19:37:53 web nginx_error: 2022/05/22 19:37:53 [error] 4370#4370: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /favicon.ico HTTP/1.1", host: "192.168.56.10", referrer: "http://192.168.56.10/"
May 22 19:38:09 web nginx_error: 2022/05/22 19:38:09 [error] 4370#4370: *1 open() "/usr/share/nginx/html/pp" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /pp HTTP/1.1", host: "192.168.56.10"
[root@log ~]# grep web /var/log/audit/audit.log
node=web type=DAEMON_START msg=audit(1653239356.175:6564): op=start ver=2.8.5 format=raw kernel=3.10.0-1127.el7.x86_64 auid=4294967295 pid=4892 uid=0 ses=4294967295 subj=system_u:system_r:auditd_t:s0 res=success
node=web type=CONFIG_CHANGE msg=audit(1653239356.324:1693): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=remove_rule key="nginx_conf" list=4 res=1
node=web type=CONFIG_CHANGE msg=audit(1653239356.324:1694): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=remove_rule key="nginx_conf" list=4 res=1
node=web type=CONFIG_CHANGE msg=audit(1653239356.324:1695): audit_backlog_limit=8192 old=8192 auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 res=1
node=web type=CONFIG_CHANGE msg=audit(1653239356.324:1696): audit_failure=1 old=1 auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 res=1
node=web type=CONFIG_CHANGE msg=audit(1653239356.326:1697): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=add_rule key="nginx_conf" list=4 res=1
node=web type=CONFIG_CHANGE msg=audit(1653239356.332:1698): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=add_rule key="nginx_conf" list=4 res=1
node=web type=SERVICE_START msg=audit(1653239356.345:1699): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=auditd comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
node=web type=SYSCALL msg=audit(1653239398.255:1700): arch=c000003e syscall=268 success=yes exit=0 a0=ffffffffffffff9c a1=b4b420 a2=1a4 a3=7ffd13e354a0 items=1 ppid=4849 pid=4917 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="chmod" exe="/usr/bin/chmod" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web type=CWD msg=audit(1653239398.255:1700):  cwd="/root"
node=web type=PATH msg=audit(1653239398.255:1700): item=0 name="/etc/nginx/nginx.conf" inode=33561196 dev=08:01 mode=0100755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PROCTITLE msg=audit(1653239398.255:1700): proctitle=63686D6F64002D78002F6574632F6E67696E782F6E67696E782E636F6E66
node=web type=SYSCALL msg=audit(1653239655.488:1701): arch=c000003e syscall=268 success=yes exit=0 a0=ffffffffffffff9c a1=c66420 a2=1ed a3=7ffe83a77de0 items=1 ppid=4849 pid=4923 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="chmod" exe="/usr/bin/chmod" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web type=CWD msg=audit(1653239655.488:1701):  cwd="/root"
node=web type=PATH msg=audit(1653239655.488:1701): item=0 name="/etc/nginx/nginx.conf" inode=33561196 dev=08:01 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PROCTITLE msg=audit(1653239655.488:1701): proctitle=63686D6F64002B78002F6574632F6E67696E782F6E67696E782E636F6E66
```

Также в плейбук включен этап развертывания еще одном vm с elk. После установки остается только зайти в кибану (пробрасывается порт 5601 на локальную машину) и добавить патерн с логами nginx.

![image](https://user-images.githubusercontent.com/98832702/170556802-dbb3adba-9352-4631-93c6-0ad9afa7adc3.png)

![image](https://user-images.githubusercontent.com/98832702/170556868-b659d4d6-9e51-44f9-b1c9-7dfb2f77baa6.png)



