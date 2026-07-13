# Task 3 Script

> Этот файл демонстрирует скрипт и шаги для выполнения задания.

---

## Шаг 1. Создание скрипта

```bash
cd ~
nano Aston.sh
```

Содержимое `Aston.sh`:

```bash
#!/bin/bash

mkdir -p /opt/app
touch /opt/app/log.txt

while true
do
    LEN=$((RANDOM % 20 + 1))
    cat /dev/urandom | tr -dc 'a-zA-Z0-9' | head -c $LEN >> /opt/app/log.txt
    echo >> /opt/app/log.txt
    sleep 17
done
```

Сохраняем (`Ctrl+S`) и выходим (`Ctrl+X`).

---

## Шаг 2. Делаем скрипт исполняемым и перемещаем в нужную папку

```bash
chmod +x Aston.sh
./Aston.sh          # создание папки при помощи скрипта, условие ТЗ.
mv /root/Aston.sh /opt/app/Aston.sh
```

---

## Шаг 3. Настройка logrotate

Создаём конфиг:

```bash
sudo nano /etc/logrotate.d/aston
```

Содержимое `/etc/logrotate.d/aston`:

```plaintext
/opt/app/log.txt {
    daily
    rotate 5
    compress
    missingok
    copytruncate
}
```

Сохраняем и выходим.

---

## Шаг 4. Добавление в автозагрузку и запуск процесса

Запускаем скрипт в фоне:

```bash
nohup /opt/app/Aston.sh &
```

Проверка процесса:

```bash
ps aux | grep Aston.sh
```

Пример вывода:

```
root     1586715  0.0  0.1   7340  3708 pts/2    S    09:47   0:00 /bin/bash /opt/app/Aston.sh
```

Добавляем в cron для автозапуска при ребуте:

```bash
crontab -e
```

Добавляем строку:

```
@reboot /opt/app/Aston.sh
```