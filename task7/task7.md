# Task 7 Docker

> Этот файл демонстрирует шаги установки и выполнения задания.

---

## Шаг 1. Подготовка всех необходимых компонентов.

Так как с прошлых домашних заданий у нас остались контейнеры и конфиг nginx, будем использовать уже имеющиеся данные и файлы.

Для начала изменим конфиг файл nginx

```bash
cd ~/kanban-backend
cd nginx
nano nginx.conf
```

Содержимое nginx.conf

```plaintext
events {}

http {

    upstream frontend {
        server frontend1:80;
        server frontend2:80;
    }

    upstream backend {
        server kanban-app:8080;
    }

    server {
        listen 80;
        server_name app.local;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name app.local;

        ssl_certificate     /etc/nginx/certs/app.local.crt;
        ssl_certificate_key /etc/nginx/certs/app.local.key;

        location /api/ {
            proxy_pass http://backend/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

В этоом файле мы добавили работу по HHTPS у nginx, прописали, сертификаты, используемые nginx


Создаём папку для самоподписанного сертификата, т.к. в config файле мы прописали уже HTTPS протокол и редирект с 80 на 443, создаём самоподписанный сертификат и кладём его файлы в созданную папку, как мы это делали в предыдущем задании

```bash
mkdir -p nginx/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048   -keyout nginx/certs/app.local.key   -out nginx/certs/app.local.crt   -subj "/CN=app.local"
```


Добавляем в файл /etc/hosts резовл app.local на 127.0.0.1

```bash
nano /etc/hosts
```

Содержимое /etc/hosts

```plaintext
127.0.0.1 localhost
127.0.1.1 aston
127.0.0.1 app.local
```

Переходим к docker-compose.yml файлу, необходимо отредактировать очерёдность запуска, а так же прописать работу с сертификатами и 443 портом

```bash
nano docker-compose.yml
```

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
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      frontend1:
        condition: service_healthy
      frontend2:
        condition: service_healthy
    ports:
      - "80:80"
      - "443:443"

volumes:
  postgres-data:
```

Что изменилось с прошлого задания:

1: Прописана конфигурация работы по HTTPS протоколу
2: Изменена методология проверки очерёдности запуска контейнеров


Далее для проверки конфигра прогоним через встроеный валидатор docker-compose.yml

```bash
docker compose config
```

Пример вывода:

```plaintext
name: kanban-backend
services:
  backend:
    build:
      context: /home/odmin1/kanban-backend
      dockerfile: Dockerfile
    container_name: kanban-app
    depends_on:
      postgres:
        condition: service_healthy
        required: true
    networks:
      default: null
    ports:
      - mode: ingress
        target: 8080
        published: "8081"
        protocol: tcp
  frontend1:
    build:
      context: /home/odmin1/kanban-frontend
      dockerfile: Dockerfile
    container_name: frontend1
    depends_on:
      backend:
        condition: service_started
        required: true
    networks:
      default: null
  frontend2:
    build:
      context: /home/odmin1/kanban-frontend
      dockerfile: Dockerfile
    container_name: frontend2
    depends_on:
      backend:
        condition: service_started
        required: true
    networks:
      default: null
  nginx:
    container_name: kanban-nginx
    depends_on:
      frontend1:
        condition: service_healthy
        required: true
      frontend2:
        condition: service_healthy
        required: true
    image: nginx:latest
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "80"
        protocol: tcp
      - mode: ingress
        target: 443
        published: "443"
        protocol: tcp
    volumes:
      - type: bind
        source: /home/odmin1/kanban-backend/nginx/nginx.conf
        target: /etc/nginx/nginx.conf
        read_only: true
        bind:
          create_host_path: true
      - type: bind
        source: /home/odmin1/kanban-backend/nginx/certs
        target: /etc/nginx/certs
        read_only: true
        bind:
          create_host_path: true
  postgres:
    container_name: kanban-postgres
    environment:
      POSTGRES_DB: kanban
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U postgres
      timeout: 5s
      interval: 5s
      retries: 10
    image: postgres:13
    networks:
      default: null
    ports:
      - mode: ingress
        target: 5432
        published: "5432"
        protocol: tcp
    volumes:
      - type: volume
        source: postgres-data
        target: /var/lib/postgresql/data
        volume: {}
networks:
  default:
    name: kanban-backend_default
volumes:
  postgres-data:
    name: kanban-backend_postgres-data
```

Так же стоит изменить default.conf, чтобы он позволил контейнерам увидеть друг друга

```bash
nano /home/odmin1/kanban-frontend/default.conf
```

Содержимое файла default.conf:

```plaintext
server {
    listen 80;

    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Из файла были удалены правила proxy_pass, так как маршрутизацией между frontend и backend теперь занимается отдельный контейнер Ngin


Далее пробуем собрать, применив все параметры и изменения, которые мы внесли

```bash
docker compose up -d --build
```

Пример вывода:

```plaintext
 ✔ backend                              Built                                                                                                                                                                                           0.0s
 ✔ frontend1                            Built                                                                                                                                                                                           0.0s
 ✔ frontend2                            Built                                                                                                                                                                                           0.0s
 ✔ Network kanban-backend_default       Created                                                                                                                                                                                         0.6s
 ✔ Volume kanban-backend_postgres-data  Created                                                                                                                                                                                         0.2s
 ✔ Container frontend1                  Healthy                                                                                                                                                                                        26.2s
 ✔ Container frontend2                  Healthy                                                                                                                                                                                        26.1s
 ✔ Container kanban-postgres            Healthy                                                                                                                                                                                        27.5s
 ✔ Container kanban-nginx               Started                                                                                                                                                                                        20.6s
 ✔ Container kanban-app                 Started 
```

После завершения сборки приложение становится доступно по адресу https://app.local. При обращении к / открывается frontend, а запросы к /api/ автоматически перенаправляются в backend через Nginx