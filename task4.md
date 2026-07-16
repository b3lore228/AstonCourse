# Task 4 Nginx

> Этот файл демонстрирует шаги установки и выполнения задания.

---

## Шаг 1. Проверка системы, обновления БД пакетов и версий.

```bash
sudo apt update && apt upgrade

netstat -tulnp | grep 80 # проверка занятости порта, на котором работает реверс прокси.
```
Пример вывода:

```plaintext
tcp        0      0 0.0.0.0:3493            0.0.0.0:*               LISTEN      3380669/upsd
tcp6       0      0 :::80                   :::*                    LISTEN      2550814/apache2
tcp6       0      0 ::1:3493                :::*                    LISTEN      3380669/upsd
```


В случае обнаружения сервиса, который слушает порт, его необходимо остановить.
```bash
sudo systemctl stop apache2 #В моём случае это apache2.
sudo systemctl disable apache2 #Отключаем автозагрузку во избежании проблем.
```
---

## Шаг 2. Установка сервиса и проверка работы

```bash
sudo apt install nginx -y #Ключ -y автоматически соглашается на установку.
systemctl status nginx
```
Пример вывода:

```plaintext
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-07-15 15:08:41 MSK; 7s ago
       Docs: man:nginx(8)
    Process: 2748258 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2748259 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2748261 (nginx)
      Tasks: 5 (limit: 2262)
     Memory: 3.7M (peak: 4.2M)
        CPU: 19ms
     CGroup: /system.slice/nginx.service
             ├─2748261 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─2748262 "nginx: worker process"
             ├─2748263 "nginx: worker process"
             ├─2748264 "nginx: worker process"
             └─2748265 "nginx: worker process"

Jul 15 15:08:41 zabbix systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Jul 15 15:08:41 zabbix systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

В случае вывода иного, например:
```
× nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Wed 2026-07-15 14:47:39 MSK; 1min 51s ago
       Docs: man:nginx(8)
    Process: 2740317 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2740325 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=1/FAILURE)
        CPU: 19ms

Jul 15 14:47:38 zabbix nginx[2740325]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Jul 15 14:47:38 zabbix nginx[2740325]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
Jul 15 14:47:38 zabbix nginx[2740325]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Jul 15 14:47:38 zabbix nginx[2740325]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
Jul 15 14:47:39 zabbix nginx[2740325]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Jul 15 14:47:39 zabbix nginx[2740325]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
Jul 15 14:47:39 zabbix nginx[2740325]: nginx: [emerg] still could not bind()
Jul 15 14:47:39 zabbix systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Jul 15 14:47:39 zabbix systemd[1]: nginx.service: Failed with result 'exit-code'.
Jul 15 14:47:39 zabbix systemd[1]: Failed to start nginx.service - A high performance web server and a reverse proxy server.
```
Следует ещё раз проверить порт, проверить логи системного журнала, внимательно прочитать вывод предыдущей команды

Так же добавим nginx в автозагрузку.

```bash
systemctl enable nginx
```

---
## Шаг 3. Создание домена, конфигурация Nginx.

Создаём локальный домен:

```bash
sudo nano /etc/hosts
```

Содержимое `/etc/hosts`:

```plaintext
127.0.0.1 localhost
127.0.0.1 app.local
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Сохраняем и выходим.

Создаём файл конфигурации нашего домена:

```bash
sudo nano /etc/nginx/conf.d/app.local.conf
```

Содержимое `/etc/hosts`:

```plaintext
server {
    listen 80;
    server_name app.local;

    root /var/www/html;
    index index.html;
}
```

Сохраняем и выходим.

```bash
sudo nginx -t #Выполняем синтаксическую проверку файла без перезапуска сервиса.
```

Пример вывода:

```plaintext
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Данная команда проверяет файл конфигурации nginx.conf, в котором и находится строчка вызова всех конфигураций - include /etc/nginx/conf.d/*.conf.

Далее необходимо выполнить перезагрузку сервиса для вступления изменений в силу.

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

Пример вывода:

```plaintext
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-07-15 16:01:09 MSK; 1min 20s ago
       Docs: man:nginx(8)
    Process: 2767310 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2767312 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2767313 (nginx)
      Tasks: 5 (limit: 2262)
     Memory: 3.7M (peak: 4.0M)
        CPU: 20ms
     CGroup: /system.slice/nginx.service
             ├─2767313 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─2767314 "nginx: worker process"
             ├─2767315 "nginx: worker process"
             ├─2767316 "nginx: worker process"
             └─2767317 "nginx: worker process"
```
---

## Шаг 4. Создание скрипта и проверка работы

Создаём файл:

```bash
nano /opt/app/Aston2.sh
```

Содержимое файла:

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Укажите адрес."
    exit 1
fi

curl -k --silent --head --fail "$1" > /dev/null

if [ $? -eq 0 ]; then
    echo "Ресурс доступен."
    exit 0
else
    echo "Ресурс недоступен."
    exit 1
fi
```

Делаем скрипт исполняемым файлом:

```bash
chmod +x /opt/app/Aston2.sh
```

Выполняем проверку, если все предущие шаги выполнены корректно, то на выходе должны получить "Ресур доступен":

```bash
./Aston2.sh http://app.local
```

Пример вывода:

```plaintext
Ресурс доступен.
```
---

## Шаг 5. Дополнительное задание.

Для получения сертификата воспользуемся open ssl

```bash
sudo openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/ssl/private/app.local.key \
-out /etc/ssl/certs/app.local.crt
```

Разберём построчно, что делает команда:

```bash
sudo openssl req -x509 -nodes -days 365 #Данная строчка создаёт запрос на сертификат
```

Ключ -x509 говорит, что запрос создавать не нужно, сразу сделай сертификат.

Ключ -nodes отключает защиту приватного ключа паролем

Ключ -days буквально означает количество дней, которое будет сертификат действовать.

```bash
-newkey rsa:2048 #Метод шифрования приватного ключа
```

```bash
-keyout /etc/ssl/private/app.local.key #Указывает расположение, где будет храниться ключ
```

```bash
-out /etc/ssl/certs/app.local.crt #Указывает расположение сертификата
```

Далее для првоерки стоит выполнить команды

```bash
 ls /etc/ssl/private/ | grep app.local
```

Пример вывода:

```plaintext
app.local.key
```

```bash
ls /etc/ssl/certs/ | grep app.local
```

Пример вывода:

```plaintext
app.local.crt
```

Теперь необходимо настроить конфигурацию nginx для работы по протоколу HHTPS, а так же проверить занятость 443 порта.

```bash
netstat -tulnp | grep 443
```

Если порт свободен, то в выводе ничего не будет, если занят, то выполняем инструкции из шага 1.

```bash
nano /etc/nginx/conf.d/app.local.conf
```

Добавляем блок для 443 порта.

```plaintext
server {

    listen 443 ssl;

    server_name app.local;

    ssl_certificate /etc/ssl/certs/app.local.crt;
    ssl_certificate_key /etc/ssl/private/app.local.key;

    root /var/www/html;
    index index.html;

}
```

Проверяем синтаксис и перезапускаем.

```bash
sudo nginx -t
```

Пример вывода:

```plaintext
anginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

```bash
sudo systemctl restart nginx
```

Выполняем финальную проверку скрипта на протокол HHTPS

```bash
./Aston2.sh https://app.local
```

Пример вывода:

```plaintext
Ресурс доступен.
```
