# Task 6 Docker

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

    # HTTP -> HTTPS редирект
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

Создаём папку для самоподписанного сертификата, т.к. в config файле мы прописали уже HTTPS протокол и редирект с 80 на 443

```bash
mkdir -p nginx/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048   -keyout nginx/certs/app.local.key   -out nginx/certs/app.local.crt   -subj "/CN=app.local"
```

Создаём самоподписанный сертификат и кладём его файлы в созданную папку.

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

Переходим к docker-compose.yml файлу, необходимо отредактировать очерёдность запуска.

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
    ports:
      - "80:80"
      - "443:443"

    ports:
      - "8080:80"

volumes:
  postgres-data:
```