# Task 8 CI/CD

> Этот файл демонстрирует шаги установки и выполнения задания.

---

## Шаг 1. Установка GitLab Runner

Для начала нам необходимо установить GitLab Runner для реализации CD

```bash
apt-cache policy gitlab-runner gitlab-runner-helper-images #Проверяем доступные версии раннера
```

Пример вывода:

```plaintext
gitlab-runner:
  Installed: (none)
  Candidate: 19.2.0-1
  Version table:
     19.2.0-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     19.1.2-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     19.1.1-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     19.1.0-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     19.0.2-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     19.0.1-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     19.0.0-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     18.11.4-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     18.11.3-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
     18.11.2-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main amd64 Packages
gitlab-runner-helper-images:
  Installed: (none)
  Candidate: 19.2.1-1
  Version table:
     19.2.1-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.2.0-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.1.2-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.1.1-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.1.0-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.0.2-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.0.1-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     19.0.0-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     18.11.4-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
     18.11.3-1 500
        500 https://packages.gitlab.com/runner/gitlab-runner/ubuntu noble/main all Packages
```

Устанавливаем последнюю доступную версию

```bash
sudo apt install gitlab-runner=19.2.0-1 gitlab-runner-helper-images=19.2.1-1
```

Далее для клонированного репозитория создадим CI/CD GitLab Runner и зарегистрируем его

```bash
gitlab-runner register  --url https://gitlab.com  --token glrt-eRUv5OXQLbksXwmLK8py4GM6MQpvOjEKcDoxZWsyZWoKdDozCnU6bzVnNDMc.01.1o0mld5bm
```

После регистрации будет вывод:

```plaintext
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!

Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml"
```

Далее в корне локального клонированного репозитория создадим файл .gitlab-ci.yml

```bash
nano .gitlab-ci.yml
```

Содержимое .gitlab-ci.yml:

```plaintext
stages:
  - build
  - deploy

variables:
  COMPOSE_DIR: /home/odmin1/kanban-backend

build_backend:
  stage: build
  image: maven:3.9.11-eclipse-temurin-8
  script:
    - mvn clean package -DskipTests
  artifacts:
    paths:
      - target/*.jar
  only:
    - main

deploy:
  stage: deploy
  script:
    - cd $COMPOSE_DIR
    - docker compose down --remove-orphans
    - docker compose up --build -d
  only:
    - main
```

Далее необходимо повторить данные действие для frontend репозитория

Так же нам необходимо создать файл .gitlab-ci.yml

```plaintext
stages:
  - build
  - deploy

variables:
  COMPOSE_DIR: /home/odmin1/kanban-backend

build_frontend:
  stage: build
  image: node:12
  script:
    - npm install --legacy-peer-deps
    - npm run build
  artifacts:
    paths:
      - dist/
  only:
    - main

deploy:
  stage: deploy
  script:
    - cd $COMPOSE_DIR
    - docker compose down --remove-orphans
    - docker compose up --build -d
  only:
    - main
```

Пробуем закоммитить через CI

```bash
git add .gitlab-ci.yml
git commit -m "Add GitLab CI"
git push
```

GitLab вернул нам ошибку

```plaintext
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'root@aston.(none)')
```

По инструкции объявим себя

```bash
git config --global user.email "e.koveshnikov.01@gmail.com" #В данном случае есть ключ --global, который применяет эти параметры глобально, если необходимо только для этого репозитория, то следует убрать ключ
git config --global user.name "Egor Koveshnikov" #В данном случае есть ключ --global, который применяет эти параметры глобально, если необходимо только для этого репозитория, то следует убрать ключ
```

Теперь пробуем запустить пайплайн

```bash
git add .gitlab-ci.yml
git commit -m "Add GitLab CI pipeline"
git push origin main
```

Далее у нас запросит учётные данные от GitLab

```bash
Username for 'https://gitlab.com': e.koveshnikov.01@gmail.com
Password for 'https://e.koveshnikov.01%40gmail.com@gitlab.com':
```

Далее если Pipeline прошёл успешно, мы увидим вывод

```bash
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 885 bytes | 885.00 KiB/s, done.
Total 6 (delta 2), reused 0 (delta 0), pack-reused 0
To https://gitlab.com/aston-learning/kanban-frontend
   a233af4..eb5a033  main -> main
```

Аналогично для backend

```bash
warning: redirecting to https://gitlab.com/aston-learning/kanban-backend.git/
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 540 bytes | 540.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To https://gitlab.com/aston-learning/kanban-backend
   bbde17c..bdf22e8  main -> main
```

Далее переходим в GitLab и проверяем, если аккаунт не верифицирован, то pipeline будет в ошибке, но деплой должен случиться.

Один из простых методов проверки - просмотр файла default.conf, так как он был изменён в предыдущем задании, а так же по наличию файла .gitlab-ci.yml