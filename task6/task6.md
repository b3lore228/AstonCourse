# Task 6 Docker

> Этот файл демонстрирует шаги установки и выполнения задания.

Ссылка на рапозиторий GitLab:
https://gitlab.com/aston-learning
Ссылка на backend:
https://gitlab.com/aston-learning/kanban-backend
Ссылка на frontend:
https://gitlab.com/aston-learning/kanban-frontend

---

## Шаг 1. Клонирование репозиториев.

Для начала нам необходимо склонировать исходные репозитории

```bash
git clone https://gitlab.com/astn-dvps/kanban-backend.git
```

Пример вывода:

```plaintext
Cloning into 'kanban-backend'...
remote: Enumerating objects: 60, done.
remote: Total 60 (delta 0), reused 0 (delta 0), pack-reused 60 (from 1)
Receiving objects: 100% (60/60), 13.90 KiB | 1.74 MiB/s, done.
Resolving deltas: 100% (7/7), done.
```

```bash
git clone https://gitlab.com/astn-dvps/kanban-frontend.git
```

Пример вывода:

```plaintext
Cloning into 'kanban-frontend'...
remote: Enumerating objects: 74, done.
remote: Total 74 (delta 0), reused 0 (delta 0), pack-reused 74 (from 1)
Receiving objects: 100% (74/74), 576.84 KiB | 3.26 MiB/s, done.
Resolving deltas: 100% (4/4), done.
```

Проверим расположение репозиториев

```bash
ls -a | grep kanban
```

Пример вывода:

```plaintext
kanban-backend
kanban-frontend
```

Теперь, когда мы склонировали репозитории на локальную машину, можно запушить их в репозиторий GitLab

```bash
cd kanban-backend
git remote -v
```

Пример вывода:

```plaintext
origin  https://gitlab.com/astn-dvps/kanban-backend.git (fetch)
origin  https://gitlab.com/astn-dvps/kanban-backend.git (push)
```

```bash
git remote remove origin
git remote -v
```

```bash
git remote add origin https://gitlab.com/aston-learning/kanban-backend.git
git remote -v
```

Пример вывода:

```plaintext
origin  https://gitlab.com/aston-learning/kanban-backend# (fetch)
origin  https://gitlab.com/aston-learning/kanban-backend# (push)
```

```bash
cd ~
cd kanban-frontend
git remote -v
```

Пример вывода:

```plaintext
origin  https://gitlab.com/astn-dvps/kanban-frontend.git (fetch)
origin  https://gitlab.com/astn-dvps/kanban-frontend.git (push)
```

```bash
git remote remove origin
git remote -v
git remote add origin https://gitlab.com/aston-learning/kanban-frontend.git
git remote -v
```

Пример вывода:

```plaintext
origin  https://gitlab.com/aston-learning/kanban-frontend# (fetch)
origin  https://gitlab.com/aston-learning/kanban-frontend# (push)
```

```bash
git branch #Если в ветке develop, то переименуем на ветку, которая в созданном нами репозитории
git status #Убедимся, что рабочее дерево не содержит незакоммиченных изменений.
git branch -M main
git branch
git status #Убедимся, что рабочее дерево не содержит незакоммиченных изменений.
git push -uf origin main #Пушим в репозиторий
```

Пример вывода:

```plaintext
Username for 'https://gitlab.com': e.koveshnikov.01@gmail.com
Password for 'https://e.koveshnikov.01%40gmail.com@gitlab.com':
warning: redirecting to https://gitlab.com/aston-learning/kanban-frontend.git/
Enumerating objects: 74, done.
Counting objects: 100% (74/74), done.
Compressing objects: 100% (69/69), done.
Writing objects: 100% (74/74), 576.84 KiB | 72.10 MiB/s, done.
Total 74 (delta 4), reused 74 (delta 4), pack-reused 0
To https://gitlab.com/aston-learning/kanban-frontend
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

```bash
cd ~/kanban-backend
git branch #Если в ветке develop, то переименуем на ветку, которая в созданном нами репозитории
git status #Убедимся, что рабочее дерево не содержит незакоммиченных изменений.
git branch -M main
git branch
git status #Убедимся, что рабочее дерево не содержит незакоммиченных изменений.
git push -uf origin main
```

Пример вывода:

```plaintext
Username for 'https://gitlab.com': e.koveshnikov.01@gmail.com
Password for 'https://e.koveshnikov.01%40gmail.com@gitlab.com':
warning: redirecting to https://gitlab.com/aston-learning/kanban-backend.git/
Enumerating objects: 60, done.
Counting objects: 100% (60/60), done.
Compressing objects: 100% (39/39), done.
Writing objects: 100% (60/60), 13.90 KiB | 6.95 MiB/s, done.
Total 60 (delta 7), reused 60 (delta 7), pack-reused 0
To https://gitlab.com/aston-learning/kanban-backend
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Далее следует проверить репозиторий GitLab.
---

## Шаг 2. Установка docker и docker compose на виртуальную машину.

```bash
sudo apt update
sudo apt install docker.io -y
docker --version
```
Пример вывода:
```plaintext
Docker version 29.1.3, build 29.1.3-0ubuntu3~24.04.2
```
```bash
sudo apt install docker-compose-v2 -y
docker compose version
```
Пример вывода:
```plaintext
Docker Compose version 2.40.3+ds1-0ubuntu1~24.04.1
```
 Проверяем сервис

 ```bash
 systemctl status docker
 ```

 Пример вывода:
 ```plaintext
 ○ docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ docker.socket
       Docs: https://docs.docker.com
root@aston:/home/odmin1# systemctl start docker
root@aston:/home/odmin1# docker run hello-world
Unable to find image 'hello-world:latest' locally
 ```

 Запускаем так как сервис в статусе inactive (dead)

 ```bash
systemctl start docker
```

 Проверяем сервис

 ```bash
 systemctl status docker
 ```
Пример вывода:

```plaintext
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-07-29 10:22:50 MSK; 3min 51s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 6337 (dockerd)
      Tasks: 9
     Memory: 104.3M (peak: 106.9M)
        CPU: 546ms
     CGroup: /system.slice/docker.service
             └─6337 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jul 29 10:22:50 aston dockerd[6337]: time="2026-07-29T10:22:50.358785308+03:00" level=info msg="Loading containers: done."
Jul 29 10:22:50 aston dockerd[6337]: time="2026-07-29T10:22:50.508610268+03:00" level=info msg="Docker daemon" commit="29.1.3-0ubuntu3~24.04.2" containerd-snapshotter=true storage-driver=overlayfs version=29.1.3
Jul 29 10:22:50 aston dockerd[6337]: time="2026-07-29T10:22:50.508849138+03:00" level=info msg="Initializing buildkit"
Jul 29 10:22:50 aston dockerd[6337]: time="2026-07-29T10:22:50.699011005+03:00" level=info msg="Completed buildkit initialization"
Jul 29 10:22:50 aston dockerd[6337]: time="2026-07-29T10:22:50.747729081+03:00" level=info msg="Daemon has completed initialization"
Jul 29 10:22:50 aston dockerd[6337]: time="2026-07-29T10:22:50.747853883+03:00" level=info msg="API listen on /run/docker.sock"
Jul 29 10:22:50 aston systemd[1]: Started docker.service - Docker Application Container Engine.
Jul 29 10:22:59 aston dockerd[6337]: time="2026-07-29T10:22:59.638966732+03:00" level=info msg="image pulled" digest="sha256:c3cbe1cc1aa588a64951ac6286e0df7b27fe2e6324b1001c619bb358770c0178" remote="docker.io/library/hello-world:latest"
Jul 29 10:23:00 aston dockerd[6337]: time="2026-07-29T10:23:00.805864315+03:00" level=info msg="sbJoin: gwep4 ''->'52308b859ed9', gwep6 ''->''" eid=52308b859ed9 ep=youthful_williamson net=bridge nid=5bb748a73
```

Сервис успешно запущен, проверим работу

```bash
docker run hello-world
```

Пример вывода:

```plaintext
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
Digest: sha256:c3cbe1cc1aa588a64951ac6286e0df7b27fe2e6324b1001c619bb358770c0178
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
---

## Шаг 3. Создание Dockerfile, docker-compose.yml.

```bash
cd ~/kanban-backend
nano Dockerfile
```

Содержимое Dockerfile:

```bash
FROM maven:3.9.11-eclipse-temurin-8 #Используем официальный Docker-образ, в котором уже установлены Java 8, Maven, информацию о необходимых компонентах взял в pom.xml

WORKDIR /app #Создаём рабочую директорию контейнера

COPY . . #Копируем весь проект внутрь контейнера

RUN mvn clean package -DskipTests #Во время сборки контейнера Maven скачает зависимости, скомпилирует проект, соберёт исполняемый JAR, -DskipTests отключает тесты во время сборки

EXPOSE 8081 #Приложение использует порт 8081

CMD ["java", "-jar", "target/kanban-app-0.0.1-SNAPSHOT.jar"] #После запуска контейнера будет выполнена команда запуска Spring Boot приложения
```

Далее нам необходимо собрать образ Dockerfile

```bash
docker build -t kanban-backend .
```

Пример вывода:

```plaintext
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:17 min
[INFO] Finished at: 2026-07-29T09:05:49Z
[INFO] ------------------------------------------------------------------------
 ---> Removed intermediate container d6ee6371e9d5
 ---> 6cd9df9edce0
Step 5/6 : EXPOSE 8081
 ---> Running in 5dd19500ea7f
 ---> Removed intermediate container 5dd19500ea7f
 ---> c8c4fdae3a58
Step 6/6 : CMD ["java", "-jar", "target/kanban-app-0.0.1-SNAPSHOT.jar"]
 ---> Running in 1347453771fb
 ---> Removed intermediate container 1347453771fb
 ---> b8972a0ee1a8
Successfully built b8972a0ee1a8
Successfully tagged kanban-backend:latest
```

Проверим список образов Docker

```bash
docker images
```

Пример вывода:

```bash
                                                                                                                                                                                                                         i Info →   U  In Use
IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest               c3cbe1cc1aa5       25.9kB         9.49kB    U
kanban-backend:latest            b8972a0ee1a8        769MB          247MB
maven:3.9.11-eclipse-temurin-8   e3c149f44c95        518MB          138MB
```

Запустим контейнер

```bash
docker run -d \
--name kanban-backend \
-p 8081:8081 \
kanban-backend
docker ps 
```

Пример вывода:

```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Мы видим, что контейнер не запустился, стоит проверить создался ли контейнер

```bash
docker ps -a
```

Пример вывода:

```plaintext
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                      PORTS     NAMES
f33d0470ec6c   kanban-backend   "/usr/local/bin/mvn-…"   36 seconds ago   Exited (1) 22 seconds ago             kanban-backend
25c238b986b3   hello-world      "/hello"                 2 hours ago      Exited (0) 2 hours ago                youthful_williamson
```

Мы видим, что контейнер был создан, но не запущен, проверим логи

```bash
docker logs kanban-backend
```

Пример вывода:

```plaintext

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.6.RELEASE)

2026-07-29 09:09:26.060  INFO 1 --- [           main] c.w.medium.kanban.KanbanApplication      : Starting KanbanApplication v0.0.1-SNAPSHOT on f33d0470ec6c with PID 1 (/app/target/kanban-app-0.0.1-SNAPSHOT.jar started by root in /app)
2026-07-29 09:09:26.073  INFO 1 --- [           main] c.w.medium.kanban.KanbanApplication      : No active profile set, falling back to default profiles: default
2026-07-29 09:09:28.946  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data repositories in DEFAULT mode.
2026-07-29 09:09:29.096  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 131ms. Found 2 repository interfaces.
2026-07-29 09:09:31.046  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$52d3f20a] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2026-07-29 09:09:32.035  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2026-07-29 09:09:32.196  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2026-07-29 09:09:32.196  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.21]
2026-07-29 09:09:32.602  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/api]    : Initializing Spring embedded WebApplicationContext
2026-07-29 09:09:32.602  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 6339 ms
2026-07-29 09:09:34.023  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2026-07-29 09:09:35.239 ERROR 1 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Exception during pool initialization.

org.postgresql.util.PSQLException: The connection attempt failed.
```

В выводе логов мы видим, что backend обращается к PostgreSQL, он не смог установить соединение, а следовательно и не запустился, закончим со сборкой контейнеров.

```bash
cd ~/kanban-frontend
nano Dockerfile
```

Содержимое файла Dockerfile

```bash
FROM node:12 as builder  #Так как у нас в файлах есть package.json, берём официальный образ Node.js, 12 версия, так как на 18 и 14 не собирается

WORKDIR /app #Так же создаём рабочую директорию для контейнера

COPY package*.json ./ #Копируем для установки зависимостей

RUN npm install --legacy-peer-deps #Установка зависимостей из package.json, ключ --legacy-peer-deps ключ, который разрешает npm использовать старую стратегию зависимостей

COPY . . #Копируем весь проект

RUN npm run build #Собирраем Angular для package.json

FROM nginx:latest #Подключаем nginx

COPY --from=0 /app/dist/ /usr/share/nginx/html/ #Копируем файлы в каталог nginx

EXPOSE 80 #Указываем порт, который будет слушать nginx

CMD ["nginx", "-g", "daemon off;"] #Запускаем nginx
```

Собираем образ

```bash
docker build -t kanban-frontend .
```

Пример вывода:

```plaintext
Step 9/10 : EXPOSE 80
 ---> Running in 6df12b14fd0d
 ---> Removed intermediate container 6df12b14fd0d
 ---> f943d7c0d6ad
Step 10/10 : CMD ["nginx", "-g", "daemon off;"]
 ---> Running in a0741645a1c5
 ---> Removed intermediate container a0741645a1c5
 ---> 8a79bd9171d0
Successfully built 8a79bd9171d0
Successfully tagged kanban-frontend:latest
```

Проверяем имеющиеся образы

```bash
docker image
```

Пример вывода:

```bash
IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest               c3cbe1cc1aa5       25.9kB         9.49kB    U
kanban-backend:latest            b8972a0ee1a8        769MB          247MB    U
kanban-frontend:latest           8a79bd9171d0        241MB         64.3MB
maven:3.9.11-eclipse-temurin-8   e3c149f44c95        518MB          138MB
nginx:latest                     5a88c9c45479        241MB           66MB
node:12                          01627afeb110       1.36GB          351MB
node:14                          a158d3b9b4e3       1.35GB          350MB
node:18                          c6ae79e38498       1.58GB          411MB
```

Попробуем запустить контейнер

```bash
docker run -d \
  --name kanban-frontend \
  -p 8080:80 \
  kanban-frontend
```

Пример вывода:

```plaintext
f4454d68ddcc4861abe7c2bdc029b5551e0cb7b135cf7a41052a0dd9becb9bb4
```

Проверяем контейнер

```bash
docker ps
```

Пример вывода:

```plaintext
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                     NAMES
```

Контейнер не запустился, стоит проверить создался ли контейнер и стоит проверить логи

```bash
docker ps -a
```

Пример вывода:

```plaintext
b8645c2b1ff7   kanban-frontend   "/docker-entrypoint.…"   2 minutes ago   Exited (1) 2 minutes ago             kanban-frontend
```

Контейнер создался, проверил логи

```bash
docker logs kanban-frontend
```

Привер вывода:

```plaintext
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/07/29 11:57:20 [emerg] 1#1: host not found in upstream "kanban-app" in /etc/nginx/conf.d/default.conf:8
nginx: [emerg] host not found in upstream "kanban-app" in /etc/nginx/conf.d/default.conf:8
```

Из логов видно, что контейнер не может найти по dns имени kanban-app, а следовательно падает в ошибку, аналогичная ситуация была с kanban-backend, значит необходимо создать структуру приложения, где была бы и БД и backend.

Создадим docker-compose файл

```bash
services:

  postgres: #Создаём контейнер с PostgreSQL для работы backend
    image: postgres:13
    container_name: kanban-postgres

    environment:
      POSTGRES_DB: kanban
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres

    ports:
      - "5432:5432"

    volumes:
      - postgres-data:/var/lib/postgresql/data

  backend: #Собираем из текущего Dockerfile backend и называем контейнер kanban-app, так как frontend ожидает именно это имя
    build: .

    container_name: kanban-app

    depends_on:
      - postgres

    ports:
      - "8081:8080"

  frontend: #Собираем контейнер из уже имеющегося Dockerfile Frontend, после backend, так как при старте он ожидает доступность DNS имени kanban-app
    build: ../kanban-frontend

    container_name: kanban-frontend

    depends_on:
      - backend

    ports:
      - "8080:80"

volumes:
  postgres-data:
```

Так же перед запуском стоит удалить старые контейнеры, которые мы собирали из Dockerfile вручную.

```bash
docker rm -f kanban-frontend kanban-backend
```

Запускаем сборку из docker-compose.yml

```bash
docker compose up --build -d
```

На по завершению сборки должны получить:

```plaintext
 ✔ backend                              Built                                                                                                                                                                                           0.0s
 ✔ frontend                             Built                                                                                                                                                                                           0.0s
 ✔ Network kanban-backend_default       Created                                                                                                                                                                                         1.3s
 ✔ Volume kanban-backend_postgres-data  Created                                                                                                                                                                                         0.4s
 ✔ Container kanban-postgres            Created                                                                                                                                                                                         6.1s
 ✔ Container kanban-app                 Created                                                                                                                                                                                         2.4s
 ✘ Container kanban-frontend            Error response from daemon: Conflict. The container name "/kanban-frontend" is already in use by container "b8645c2b1ff700052b2363d0cce4ab87961970afa913a0ee6f74...                             0.1s # Ошибка тут из-за того, что я не удалил старый контейнер.
```

Из-за того, что я не удалил старый контейнер пришлось перезапускать

```bash
docker compose up -d #Команда не включает ключ --build так как 
```

Пример вывода:

```plaintext
 ✔ Container kanban-postgres  Started                                                                                                                                                                                                   4.5s
 ✔ Container kanban-app       Started                                                                                                                                                                                                   5.4s
 ✔ Container kanban-frontend  Started                                                                                                                                                              
```

Проверим контейнеры

```bash
docker ps
```

Пример вывода:

```plaintext
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                                   NAMES
2ad3eef8d5ee   kanban-backend-frontend   "/docker-entrypoint.…"   13 seconds ago   Up 7 seconds    0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                 kanban-frontend
fea034467bd4   kanban-backend-backend    "/usr/local/bin/mvn-…"   13 seconds ago   Up 9 seconds    8081/tcp, 0.0.0.0:8081->8080/tcp, [::]:8081->8080/tcp   kanban-app
fc8e775b99f7   postgres:13               "docker-entrypoint.s…"   14 seconds ago   Up 11 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp             kanban-postgres
```

Далее стоит проверить работу, перейдя на IP адрес машины и на порт 8080 должна быть страница с "KANBAN BOARDS".

## Шаг 4. Дополнительные задания.

Для начала создадим балансировочную конфигурацию nginx

```bash
mkdir nginx
cd nginx
nano nginx.conf
```

Содержимое файла nginx.conf

```plaintext
events {}

http {

    upstream frontend {
        server frontend1:80;
        server frontend2:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://frontend;
        }
    }

}
```

Данный конфигурационный файл позволит отправлять запросы поочерёдно сначала на frontend1, затем на frontend2.

Далее отредактируем файл docker-compose.yml

```bash
nano docker-compose.yml
```

Содержимое docker-compose.yml

```bash
services:

  postgres:
    image: postgres:13
    container_name: kanban-postgres

    environment:
      POSTGRES_DB: kanban
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 10
    ports:
      - "5432:5432"

    volumes:
      - postgres-data:/var/lib/postgresql/data

  backend:
    build: .

    container_name: kanban-app


    depends_on:
      postgres:
        condition: service_healthy

    ports:
      - "8081:8080"

  frontend1:
    build: ../kanban-frontend
    container_name: frontend1

    depends_on:
      backend:
        condition: service_started

  frontend2:
    build: ../kanban-frontend
    container_name: frontend2

    depends_on:
      backend:
        condition: service_started

  nginx:
    image: nginx:latest
    container_name: kanban-nginx


    volumes:
      - /home/odmin1/kanban-backend/nginx/nginx.conf:/etc/nginx/nginx.conf:ro

    depends_on:
      - frontend1
      - frontend2

    ports:
      - "8080:80"

volumes:
  postgres-data:
```

Что изменилось?

1: Добавлены 2 контейнера, а так же конфигурация nginx которая будут брать логику работы из файла, который мы создали до этого

Контейнеры

```bash
  frontend1:
    build: ../kanban-frontend
    container_name: frontend1

    depends_on:
      backend:
        condition: service_started

  frontend2:
    build: ../kanban-frontend
    container_name: frontend2

    depends_on:
      backend:
        condition: service_started
```

Конфиг

```bash
  nginx:
    image: nginx:latest
    container_name: kanban-nginx


    volumes:
      - /home/odmin1/kanban-backend/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
```

Добавлена очерёдность загрузки, которая ожидает запуска контейнера и приложений в нём

```bash
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 10
```

```bash
    depends_on:
      postgres:
        condition: service_healthy
```

Перезапустим docker-compose.yml

```bash
docker compose down -v
docker compose up --build -d
```

Пример вывода:

```bash
 ✔ backend                              Built                                                                                                                                                                                           0.0s
 ✔ frontend1                            Built                                                                                                                                                                                           0.0s
 ✔ frontend2                            Built                                                                                                                                                                                           0.0s
 ✔ Network kanban-backend_default       Created                                                                                                                                                                                         0.8s
 ✔ Volume kanban-backend_postgres-data  Created                                                                                                                                                                                         0.3s
 ✔ Container kanban-postgres            Healthy                                                                                                                                                                                        22.5s
 ✔ Container kanban-app                 Started                                                                                                                                                                                        17.9s
 ✔ Container frontend1                  Started                                                                                                                                                                                        18.2s
 ✔ Container frontend2                  Started                                                                                                                                                                                        18.2s
 ✔ Container kanban-nginx               Started 
```