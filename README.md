# OTUS PRO Homework 18 SELinux

## Домашняя работа 18: SELinux

### Домашнее задание:
1. Подготовка рабочего места   
2. Запустить nginx на нестандартном порту 3-мя разными способами:  
     - переключатели setsebool;
     - добавление нестандартного порта в имеющийся тип;
     - формирование и установка модуля SELinux.
3. Обеспечить работоспособность приложения при включенном selinux.
      - выяснить причину неработоспособности механизма обновления зоны;
      - предложить решение (или решения) для данной проблемы;
      - выбрать одно из решений для реализации, предварительно обосновав выбор;
      - реализовать выбранное решение и продемонстрировать его работоспособность.
---
## Выполнение задания:
### 1. Подготовка рабочего места:
Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: **/opt/otus/SELinux**):
```
mkdir -p /opt/otus/SELinux ; cd /opt/otus/SELinux
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_18.git;
```
  - В результате, после выполнения предыдущей команды получим 2 папки с подготовленными vagrant стендами для заданий 2 и 3
    
    **В нашем примере:**
  -  в папке **/opt/otus/SELinux/nginx** находится Vagrant-стенд для выполнения **ДЗ 2** (nginx на нестандартном порту)
  -  в папке **/opt/otus/SELinux/dns** находится Vagrant-стенд для выполнения **ДЗ 3** (работоспособность приложения при включенном selinux)   
Далее, будет достаточно зайти в нужную папку и выполнить команду для запуска нужного vagrant-box:
```
vagrant up
``` 
---
### 2. Запустить nginx на нестандартном порту 3-мя разными способами
Зайдите в папку с проектом ДЗ 2 (в нашем примере **/opt/otus/SELinux/nginx**)
```
cd /opt/otus/SELinux/nginx
```
**Запустите проект из папки:**
```
vagrant up
```
Результатом выполнения команды vagrant up станет созданная виртуальная машина с установленным nginx, который работает на порту TCP 4881.
Порт TCP 4881 уже проброшен до хоста. SELinux включен.   

<details>
<summary> Во время развёртывания стенда попытка запустить nginx завершится с ошибкой: </summary>
  
```
    selinux: Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
    selinux: ● nginx.service - The nginx HTTP and reverse proxy server
    selinux:    Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
    selinux:    Active: failed (Result: exit-code) since Tue 2024-11-19 16:41:28 UTC; 14ms ago
    selinux:   Process: 13765 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
    selinux:   Process: 13764 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    selinux:
    selinux: Nov 19 16:41:28 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
    selinux: Nov 19 16:41:28 selinux nginx[13765]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    selinux: Nov 19 16:41:28 selinux nginx[13765]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
    selinux: Nov 19 16:41:28 selinux nginx[13765]: nginx: configuration file /etc/nginx/nginx.conf test failed
    selinux: Nov 19 16:41:28 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
    selinux: Nov 19 16:41:28 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
    selinux: Nov 19 16:41:28 selinux systemd[1]: Unit nginx.service entered failed state.
```
</details>

  - Примечание: Данная ошибка появляется из-за того, что SELinux блокирует работу nginx на нестандартном порту.
    
**Зайдите в виртуальную машину (box):**
```
vagrant ssh
```
  - Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя:
```
sudo -i
```
**Посмотрим статус сервиса Nginx**
```
systemctl status nginx.service
```

<details>
<summary> результат выполнения команды: </summary>

```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Tue 2024-11-19 16:41:28 UTC; 28min ago
  Process: 13765 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 13764 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)

Nov 19 16:41:28 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 19 16:41:28 selinux nginx[13765]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 19 16:41:28 selinux nginx[13765]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Nov 19 16:41:28 selinux nginx[13765]: nginx: configuration file /etc/nginx/nginx.conf test failed
Nov 19 16:41:28 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Nov 19 16:41:28 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Nov 19 16:41:28 selinux systemd[1]: Unit nginx.service entered failed state.
Nov 19 16:41:28 selinux systemd[1]: nginx.service failed.
```
</details>

**Проверим, что в ОС отключен файервол:**
```
systemctl status firewalld
```

<details>
<summary> результат выполнения команды: </summary>

```
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```
</details>

**Проверим, что конфигурация nginx корректна:**
```
nginx -t
```

<details>
<summary> результат выполнения команды: </summary>

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
</details>

**Проверим режим работы SELinux:**
```
getenforce
```

<details>
<summary> результат выполнения команды: </summary>

```
Enforcing
```
</details>

  - Примечание: Режим **Enforcing** означает, что SELinux будет блокировать запрещенную активность.   
   
**Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setsebool:**   
   
Находим в логах (**/var/log/audit/audit.log**) информацию о блокировании порта **4881** (порт из статуса сервиса nginx):
```
cat /var/log/audit/audit.log | grep 4881
```

<details>
<summary> результат выполнения команды: </summary>

```
type=AVC msg=audit(1732034488.921:1114): avc:  denied  { name_bind } for  pid=13765 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
```
</details>

Возьмем время из результата предыдущей команды (в которое был записан этот лог) и, с помощью утилиты audit2why смотрим:

```
grep 1732034488.921:1114 /var/log/audit/audit.log | audit2why
```

<details>
<summary> результат выполнения команды: </summary>

```
type=AVC msg=audit(1732034488.921:1114): avc:  denied  { name_bind } for  pid=13765 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly.
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
```
</details>

  - Примечание: Утилита **audit2why** покажет почему трафик блокируется. Исходя из вывода утилиты, мы видим, что нам нужно поменять параметр **nis_enabled**   

Включим параметр **nis_enabled** и перезапустим nginx:

```
setsebool -P nis_enabled 1
```
```
systemctl restart nginx
```
```
systemctl status nginx
```

<details>
<summary> результат выполнения команд: </summary>

```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2024-11-19 17:27:16 UTC; 5s ago
  Process: 32730 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 32728 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 32727 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 32732 (nginx)
   CGroup: /system.slice/nginx.service
           ├─32732 nginx: master process /usr/sbin/nginx
           └─32734 nginx: worker process

Nov 19 17:27:16 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 19 17:27:16 selinux nginx[32728]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 19 17:27:16 selinux nginx[32728]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Nov 19 17:27:16 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```

</details>

#### Видим, что Nginx теперь работает   

  - Примечание: Проверить статус параметра можно с помощью команды:
<details>
<summary> getsebool -a | grep nis_enabled </summary>

```
nis_enabled --> on
```

</details>
   

**Вернём запрет работы nginx на порту 4881 обратно. Для этого отключим nis_enabled:**
```
setsebool -P nis_enabled 0
```

**Пробуем запустить Nginx (он снова не запустится):**

```
systemctl restart nginx.service; systemctl status nginx.service
```

<details>
<summary> результат выполнения команд: </summary>

```
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Tue 2024-11-19 17:37:21 UTC; 13ms ago
  Process: 32730 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 339 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 336 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 32732 (code=exited, status=0/SUCCESS)

Nov 19 17:37:21 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 19 17:37:21 selinux nginx[339]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 19 17:37:21 selinux nginx[339]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Nov 19 17:37:21 selinux nginx[339]: nginx: configuration file /etc/nginx/nginx.conf test failed
Nov 19 17:37:21 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Nov 19 17:37:21 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Nov 19 17:37:21 selinux systemd[1]: Unit nginx.service entered failed state.
Nov 19 17:37:21 selinux systemd[1]: nginx.service failed.
```

</details>

**Теперь разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип:**   

**Выполним поиск имеющегося типа, для http трафика:**
```
semanage port -l | grep http
```

<details>
<summary> результат выполнения команд: </summary>

```
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```

</details>

Добавим порт в тип **http_port_t** и проверим, что порт добавился:
```
semanage port -a -t http_port_t -p tcp 4881
```
```
semanage port -l | grep  http_port_t
```

<details>
<summary> результат выполнения команд: </summary>

```
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
```

</details>

**Попробуем запустить Nginx и проверим статус службы:**

```
systemctl restart nginx.service; systemctl status nginx.service
```

<details>
<summary> результат выполнения команды: </summary>

```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2024-11-19 17:45:00 UTC; 12ms ago
  Process: 366 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 364 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 363 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 369 (nginx)
   CGroup: /system.slice/nginx.service
           ├─369 nginx: master process /usr/sbin/nginx
           └─373 nginx: master process /usr/sbin/nginx

Nov 19 17:45:00 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 19 17:45:00 selinux nginx[364]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 19 17:45:00 selinux nginx[364]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Nov 19 17:45:00 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```

</details>

#### Видим, что Nginx снова работает 
   
**Удалим нестандартный порт (4881) из имеющегося типа и проверим, что порт удалился:**

```
semanage port -d -t http_port_t -p tcp 4881
```
```
semanage port -l | grep  http_port_t
```

<details>
<summary> результат выполнения команд: </summary>

```
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
```

</details>

**Пробуем запустить Nginx (он снова не запустится):**

```
systemctl restart nginx.service; systemctl status nginx.service
```

<details>
<summary> результат выполнения команд: </summary>

```
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Tue 2024-11-19 17:51:34 UTC; 13ms ago
  Process: 366 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 392 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 391 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 369 (code=exited, status=0/SUCCESS)

Nov 19 17:51:34 selinux systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Nov 19 17:51:34 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 19 17:51:34 selinux nginx[392]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 19 17:51:34 selinux nginx[392]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Nov 19 17:51:34 selinux nginx[392]: nginx: configuration file /etc/nginx/nginx.conf test failed
Nov 19 17:51:34 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Nov 19 17:51:34 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Nov 19 17:51:34 selinux systemd[1]: Unit nginx.service entered failed state.
Nov 19 17:51:34 selinux systemd[1]: nginx.service failed.
```

</details>

**Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux:**  

Попробуем снова запустить nginx:

```
systemctl restart nginx.service; systemctl status nginx.service
```

<details>
<summary> результат выполнения команды: </summary>

```
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
```

</details>

Nginx не запуститься, так как SELinux продолжает его блокировать.   
**Посмотрим логи SELinux, которые относятся к nginx:**

```
systemctl restart nginx.service; systemctl status nginx.service
```

<details>
<summary> результат выполнения команды: </summary>

```
type=SOFTWARE_UPDATE msg=audit(1732034488.421:1112): pid=13691 uid=0 auid=1000 ses=2 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='sw="nginx-filesystem-1:1.20.1-10.el7.noarch" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python2.7" hostname=? addr=? terminal=? res=success'
type=SOFTWARE_UPDATE msg=audit(1732034488.656:1113): pid=13691 uid=0 auid=1000 ses=2 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='sw="nginx-1:1.20.1-10.el7.x86_64" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python2.7" hostname=? addr=? terminal=? res=success'
type=AVC msg=audit(1732034488.921:1114): avc:  denied  { name_bind } for  pid=13765 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1732034488.921:1114): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=55957336c8b8 a2=10 a3=7fff68147590 items=0 ppid=1 pid=13765 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=SERVICE_START msg=audit(1732034488.921:1115): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'
type=SERVICE_START msg=audit(1732037236.975:1165): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_STOP msg=audit(1732037805.155:1170): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=AVC msg=audit(1732037805.186:1171): avc:  denied  { name_bind } for  pid=32756 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1732037805.186:1171): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=55a4d751e8b8 a2=10 a3=7ffee020fc40 items=0 ppid=1 pid=32756 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=SERVICE_START msg=audit(1732037805.186:1172): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'
type=AVC msg=audit(1732037841.651:1173): avc:  denied  { name_bind } for  pid=339 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1732037841.651:1173): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=5573d69808b8 a2=10 a3=7fff6370c2d0 items=0 ppid=1 pid=339 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=SERVICE_START msg=audit(1732037841.651:1174): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'
type=SERVICE_START msg=audit(1732038300.470:1178): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_STOP msg=audit(1732038694.206:1182): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=AVC msg=audit(1732038694.238:1183): avc:  denied  { name_bind } for  pid=392 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1732038694.238:1183): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=561a5fb338b8 a2=10 a3=7ffe0d8da3a0 items=0 ppid=1 pid=392 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=SERVICE_START msg=audit(1732038694.248:1184): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'
type=AVC msg=audit(1732038865.736:1185): avc:  denied  { name_bind } for  pid=406 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1732038865.736:1185): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=562ec69518b8 a2=10 a3=7ffea0dc5a20 items=0 ppid=1 pid=406 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=SERVICE_START msg=audit(1732038865.736:1186): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'
```

</details>

Воспользуемся утилитой **audit2allow** для того, чтобы на основе логов SELinux сделать модуль, разрешающий работу nginx на нестандартном порту:
```
grep nginx /var/log/audit/audit.log | audit2allow -M nginx
```

<details>
<summary> результат выполнения команд: </summary>

```
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp
```

</details>

**Audit2allow** сформировал модуль и сообщил нам команду, с помощью которой можно применить данный модуль: **semodule -i nginx.pp**   

**Выполним предложенную команду:**
```
semodule -i nginx.pp
```

**Попробуем запустить Nginx и проверим статус службы:**

```
systemctl restart nginx.service; systemctl status nginx.service
```

<details>
<summary> результат выполнения команды: </summary>

```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2024-11-19 18:07:22 UTC; 12ms ago
  Process: 451 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 449 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 448 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 453 (nginx)
   CGroup: /system.slice/nginx.service
           ├─453 nginx: master process /usr/sbin/nginx
           └─456 nginx: master process /usr/sbin/nginx

Nov 19 18:07:22 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 19 18:07:22 selinux nginx[449]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 19 18:07:22 selinux nginx[449]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Nov 19 18:07:22 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```

</details>

#### Видим, что Nginx снова работает 

Доп.информация:
  - При использовании модуля изменения сохранятся после перезагрузки.
  - Просмотр всех установленных модулей: `semodule -l`
  - Для удаления модуля воспользуемся командой: `semodule -r nginx`
---
### 3. Обеспечение работоспособности приложения при включенном SELinux
