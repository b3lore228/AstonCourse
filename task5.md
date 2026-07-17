# Task 5 Ansible

> Этот файл демонстрирует шаги установки и выполнения задания.

---

## Шаг 1. Проверка системы, обновления БД пакетов и версий.

```bash
sudo apt update # Обновляем базу репозиториев
sudo apt install ansible -y # Устанавливаем ansible на "главную машину"
ansible --version # Проверяем установился ли пакет и какой версии
```

Пример вывода:

```plaintext
ansible [core 2.16.3]
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3 (main, Jun 19 2026, 12:46:00) [GCC 13.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

Виртуальная машина в моём случае настроена на 2 сетевых адаптера,
Первый адаптер - Nat подключение, из которого машина берёт интернет:

```plaintext
enp0s3 inet 10.0.2.15/24
```

Второе подключение - Внутренняя сеть, созданное для общения между виртуалками:

```plaintext
enp0s8 inet 192.168.100.20/24
```

Второй адаптер внутренней сети изначально не имеет параметров IP, для конфигурации порта необходимо выполнить следующее:

```bash
ls /etc/netplan/
```

Пример вывода:

```plaintext
50-cloud-init.yaml или 01-network-manager-all.yaml
```

Открываем для редактирования сам файл и вносим изменения:

```bash
nano /etc/netplan/50-cloud-init.yaml
```

Содержимое /etc/netplan/50-cloud-init.yaml:

```plaintext
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.100.20/24
```

Для второй машины всё то же самое за исключением IP

Содержимое /etc/netplan/50-cloud-init.yaml на второй машине:

```plaintext
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.100.10/24
```

Сохраняем и выходим

Теперь у нас всё готово, чтобы приступить к работе с Ansible.

---

## Шаг 2. Создание Inventory и Playbook для ansible.

На машине с Ansible создадим в домашней директории пользователя папку

```bash
cd ~ # Преход в домашнюю директорию
mkdir task5 # Создаём папку task5
mkdir /home/egor/task5/files # Создаём отдельную папку для файла html
```

Пропишем инвентарь, для этого нужно узнать IP адрес двух машин.

```bash
ip a
```

Пример вывода:

```plaintext
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 10000
    link/ether 08:00:27:ac:3f:f6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 81045sec preferred_lft 81045sec
    inet6 fe80::a00:27ff:feac:3ff6/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ea:9a:65 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.20/24 brd 192.168.100.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feea:9a65/64 scope link
       valid_lft forever preferred_lft forever
```

```bash
nano /home/egor/task5/inventory
```

Содержимое /home/egor/task5/inventory:

```plaintext
[web]
aston ansible_host=192.168.100.10

[web:vars]
ansible_user=odmin1
ansible_password=35 Krokodil7

ansible_become=true
ansible_become_method=sudo
ansible_become_password=35 Krokodil7

ansible_python_interpreter=/usr/bin/python3
```

В данном файле прописан адрес машины, которой мы собираемся управлять:

```plaintext
aston ansible_host=192.168.100.10
```

Учётные данные пользователя, а так же пароль для sudo:

```plaintext
ansible_user=odmin1
ansible_password=35 Krokodil7

ansible_become=true
ansible_become_method=sudo
ansible_become_password=35 Krokodil7
```

Сохраняем и выходим

Проверим корректность работы и доступность управляемой ноды

```bash
ansible all -i inventory -m ping
```

Пример вывода:

```plaintext
aston | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Управляемая нода доступна, ansible смог подключиться к ней и авторизоваться.

Перейдём к созданию Playbook.

```bash
nano /home/egor/task5/playbook.yml
```
 
Содержимое /home/egor/task5/playbook.yml:

 ```plaintext
 ---
- name: Configure web server
  hosts: web
  become: true

  tasks:

    - name: Update package cache
      apt:
        update_cache: true

    - name: Upgrade installed packages
      apt:
        upgrade: dist

    - name: Set timezone
      command: timedatectl set-timezone Europe/Moscow

    - name: Install base packages
      apt:
        name:
          - vim
          - curl
          - wget
          - ufw
          - net-tools
          - traceroute
          - mc
        state: present

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Enable nginx service
      service:
        name: nginx
        state: started
        enabled: true

    - name: Copy nginx page
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
```

Разберём, что данный playbook делает.

Имя самого playbook:

```plaintext
- name: Configure web server
```

Какую группу хостов из файла inventory будет использовать playbook для выполнения:

```plaintext
hosts: web
```

Права с которыми будет работать ansible, в данном случае - root:

```plaintext
become: true
```

Далее пререходим к task'ам.

Таск выполнения apt update (обновление базы данных репозиториев) на управляемой ноде

```plaintext
 - name: Update package cache
      apt:
        update_cache: true
```

Таск выполнения apt upgrade (установка компонентов из репозитория последней версии) на управляемой ноде

```plaintext
 - name: Upgrade installed packages
      apt:
        upgrade: dist
```

Таск установки временной зоны, необходимо для корректного отображения времени в логах и не только

```plaintext
 - name: Set timezone
      command: timedatectl set-timezone Europe/Moscow
```

Таск установки базовых компонентов, необходимые для "первичной настройки" сервера

```plaintext
 - name: Install base packages
      apt:
        name:
          - vim
          - curl
          - wget
          - ufw
          - net-tools
          - traceroute
          - mc
        state: present
```

Таск установки сервиса nginx на управляемую ноду

```plaintext
  - name: Install nginx
      apt:
        name: nginx
        state: present
```

Таск запуска сервиса nginx и добавление его в автозагрузку

```plaintext
 - name: Enable nginx service
      service:
        name: nginx
        state: started
        enabled: true
```

Таск копирования нашего файла index.html на управляемую ноду.

```plaintext
 - name: Copy nginx page
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
```

Теперь у нас готов playbook, который выполнит установку компонентов, необходимых для дальнейшего взаимодействия с сервером и диагностики.

Создадим простой файл, который бы заменил собой базовую приветственную страницу nginx

```bash
nano /home/egor/task5/files/index.html
```

Содержимое /home/egor/task5/files/index.html:

```plaintext
<!DOCTYPE html>

<html>

<head>
    <title>ASTON Task 5</title>
</head>

<body>

<h1>Hello from Ansible!</h1>

<p>Nginx configured successfully.</p>

</body>

</html>
```

---

## Шаг 3. Запуск и проверка работы.

```bash
cd /home/egor/task5 # Смена директории на ту, где лежит playbook
ansible-playbook -i inventory playbook.yml # Запуск playbook
```

Если всё выполнено правильно, управляющая нода подключится к управляемой и приступит к работе, которая описана в playbook

Пример вывода:

```plaintext
PLAY [Configure web server] **************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************
ok: [aston]

TASK [Update package cache] **************************************************************************************************************************************************************************
changed: [aston]

TASK [Upgrade installed packages] **************************************************************************************************************************************************************************
ok: [aston]

TASK [Set timezone] **************************************************************************************************************************************************************************
changed: [aston]

TASK [Install base packages] **************************************************************************************************************************************************************************
changed: [aston]

TASK [Install nginx] **************************************************************************************************************************************************************************
changed: [aston]

TASK [Enable nginx service] **************************************************************************************************************************************************************************
changed: [aston]

TASK [Copy nginx page] **************************************************************************************************************************************************************************
changed: [aston]

PLAY RECAP **************************************************************************************************************************************************************************
aston                      : ok=8    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

Теперь стоит проверить управляемую ноду, заходим на нёё и выполняем команды:

```bash
systemctl status nginx
```

Пример вывода:

```plaintext
nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-07-17 14:13:58 MSK; 1h 1min ago
       Docs: man:nginx(8)
   Main PID: 2568 (nginx)
      Tasks: 2 (limit: 1055)
     Memory: 1.7M (peak: 1.9M)
        CPU: 11ms
     CGroup: /system.slice/nginx.service
             ├─2568 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─2569 "nginx: worker process"

Jul 17 14:13:58 aston systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Jul 17 14:13:58 aston systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

```bash
ls /var/www/html/
```

Пример вывода:

```plaintext
index.html  index.nginx-debian.html
```

```bash
traceroute 192.168.100.20
```

Пример вывода:

```plaintext
traceroute to 192.168.100.20 (192.168.100.20), 30 hops max, 60 byte packets
 1  192.168.100.20 (192.168.100.20)  0.258 ms  0.134 ms  0.203 ms
```

---

## Шаг 4. Дополнительное задание.

На управляющей ноде ставим pipx

```bash
sudo apt install pipx -y
pipx ensurepath
```

После установки для того чтобы пути прописались необходимо перезапустить терминал, либо выполнить команду

```bash
source ~/.bashrc
```

Теперь устанавливаем через pipx ansible-lint

```bash
pipx install ansible-lint
```

Проверим версию, чтобы убедиться, что ansible-lint установился.

```bash
ansible-lint --version
```

Пример вывода:

```plaintext
ansible-lint 26.6.0 using ansible-core:2.21.2 ansible-compat:26.6.0 ruamel-yaml:0.19.1 ruamel-yaml-clib:0.2.15
```

Теперь можем проверить наш playbook на ошибки, а так же соответствие стандартам и хорошим практикам.

```bash
ansible-lint playbook.yml | tee ansible-lint1.log
```

После выполнения видим такую картину:

```plaintext
WARNING  Listing 9 violation(s) that are fatal
Read documentation for instructions on how to ignore specific rule violations.

# Rule Violation Summary

  1 risky-file-permissions profile:safety tags:unpredictability
  1 no-changed-when profile:safety tags:command-shell,idempotency
  7 fqcn profile:safety tags:formatting

Failed: 9 failure(s), 0 warning(s) in 1 files processed of 1 encountered. Last profile that met the validation criteria was 'moderate'. Rating: 2/5 star
fqcn[action-core]: Use FQCN for builtin module actions (apt).
playbook.yml:9:7 Use `ansible.builtin.apt` or `ansible.legacy.apt` instead.

fqcn[action-core]: Use FQCN for builtin module actions (apt).
playbook.yml:13:7 Use `ansible.builtin.apt` or `ansible.legacy.apt` instead.

no-changed-when: Commands should not change things if nothing needs doing.
playbook.yml:16 Task/Handler: Set timezone

fqcn[action-core]: Use FQCN for builtin module actions (command).
playbook.yml:17:7 Use `ansible.builtin.command` or `ansible.legacy.command` instead.

fqcn[action-core]: Use FQCN for builtin module actions (apt).
playbook.yml:20:7 Use `ansible.builtin.apt` or `ansible.legacy.apt` instead.

fqcn[action-core]: Use FQCN for builtin module actions (apt).
playbook.yml:32:7 Use `ansible.builtin.apt` or `ansible.legacy.apt` instead.

fqcn[action-core]: Use FQCN for builtin module actions (service).
playbook.yml:37:7 Use `ansible.builtin.service` or `ansible.legacy.service` instead.

risky-file-permissions: File permissions unset or incorrect.
playbook.yml:42 Task/Handler: Copy nginx page

fqcn[action-core]: Use FQCN for builtin module actions (copy).
playbook.yml:43:7 Use `ansible.builtin.copy` or `ansible.legacy.copy` instead.
```

В соответствии с замечаниями lint выполним исправление в playbook

```bash
nano /home/egor/task5/playbook.yml
```

Содержимое файла:

```plaintext
---
- name: Configure web server
  hosts: web
  become: true

  tasks:

    - name: Update package cache
      ansible.builtin.apt:
        update_cache: true

    - name: Upgrade installed packages
      ansible.builtin.apt:
        upgrade: dist

    - name: Set timezone
      ansible.builtin.command: timedatectl set-timezone Europe/Moscow
      changed_when: false
    
    - name: Install base packages
      ansible.builtin.apt:
        name:
          - vim
          - curl
          - wget
          - ufw
          - net-tools
          - traceroute
          - mc
        state: present

    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Enable nginx service
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Copy nginx page
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: "0644"
```

Что изменилось:

Все команды выполняются встроеными модулями ansible

```plaintext
ansible.builtin.command
ansible.builtin.apt
ansible.builtin.service
ansible.builtin.copy:
```

Теперь есть подтверждение внесения изменений в таске timezone

```plaintext
changed_when: false
```

Теперь у файла index.html прописан владелец, группа-владелец и права

```plaintext
owner: root
group: root
mode: "0644"
```

Пробуем снова, после всех корректировок выполнить

```bash
ansible-lint playbook.yml | tee ansible-lint2.log
```

Пример вывода:

```plaintext
Passed: 0 failure(s), 0 warning(s) in 1 files processed of 1 encountered. Last profile that met the validation criteria was 'production'.
```

Лог файлы сохранены в папке task5.