Домашнее задание к занятию 5. «Практическое применение Docker» - Виктор Юрочкин
---
Репозиторий GitHub: https://github.com/victoryurochkin/shvirtd-example-python
---

Виртуалки подняты на своём сервере в proxmox, а также у хостинг провайдера smartape - для альтернативного решения не через yandex cloud

<img width="3071" height="1421" alt="image" src="https://github.com/user-attachments/assets/840f897d-0471-4ccd-a28c-1b0cf9a0ef2f" />

<img width="3038" height="750" alt="image" src="https://github.com/user-attachments/assets/e37cc4c0-3dd7-4603-bd15-109ffd907b9b" />

для виртуалок работы с гитом - делал ssh ключи

<img width="2627" height="1018" alt="image" src="https://github.com/user-attachments/assets/d79e0d69-d47b-4fed-8b44-02832c5fa7ae" />

---

Задача 0
Условие
Необходимо убедиться, что:
legacy-утилита `docker-compose` (с дефисом) не установлена;
установлен Docker Compose plugin и команда `docker compose version` возвращает версию не ниже v2.24.x.
Выполненные действия
Проверка состояния системы:
```bash
docker-compose --version
docker compose version
docker --version
```
Результат до установки Docker:
`docker-compose --version` -> `Command 'docker-compose' not found`
`docker compose version` -> не выполнялась, так как сам Docker отсутствовал
`docker --version` -> Docker отсутствовал
Далее был установлен Docker Engine и Compose plugin.
Проверка после установки:
```bash
docker --version
docker compose version
docker-compose --version
```
Результат
`docker-compose --version` возвращает:
```text
  Command 'docker-compose' not found, but can be installed with:
  apt install docker-compose
  ```
`docker --version`:
```text
  Docker version 29.3.1, build c2be9cc
  ```
`docker compose version`:
```text
  Docker Compose version v5.1.1
  ```
Вывод
Условие выполнено:
устаревший `docker-compose` не установлен;
используется современный `docker compose`;
версия Compose выше требуемой `v2.24.x`.
---
Задача 1
Условие
Сделать fork репозитория, добавить:
`Dockerfile.python`
`.dockerignore`
Требования:
использовать `python:3.12-slim`
обязательно использовать `COPY . .`
запускать через:
```dockerfile
  CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000"]
  ```
использовать multistage сборку
Используемый репозиторий
Исходный репозиторий:
```text
https://github.com/netology-code/shvirtd-example-python
```
Мой fork:
```text
https://github.com/victoryurochkin/shvirtd-example-python
```
Созданный файл `Dockerfile.python`
```dockerfile
FROM python:3.12-slim AS builder

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PIP_NO_CACHE_DIR=1

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --upgrade pip && pip install -r requirements.txt

FROM python:3.12-slim AS runtime

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/opt/venv/bin:$PATH"

COPY --from=builder /opt/venv /opt/venv
COPY . .

EXPOSE 5000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000"]
```
Созданный файл `.dockerignore`
```gitignore
.git
.github
__pycache__
*.pyc
*.pyo
*.pyd
.Python
.python-version
.venv
venv
env
ENV
.idea
.vscode
.pytest_cache
.mypy_cache
.coverage
htmlcov
dist
build
*.egg-info
```
Сборка образа
```bash
docker build -f Dockerfile.python -t shvirtd-example-python:python .
```
Результат
Образ был успешно собран:
```text
shvirtd-example-python:python
```
Также приложение было успешно запущено и подключилось к MySQL.
Вывод
Задача 1 выполнена:
fork создан;
добавлены `Dockerfile.python` и `.dockerignore`;
использована multistage-сборка;
применён `python:3.12-slim`;
использован `COPY . .`;
приложение успешно собирается и запускается.
---
Задача 2 (*)
Условие
Требовалось:
создать Yandex Cloud Container Registry `test` с помощью `yc`;
настроить авторизацию Docker;
собрать и загрузить образ;
просканировать образ на уязвимости.
Фактическая реализация
Вместо Yandex Cloud Container Registry был развёрнут локальный приватный registry Harbor со встроенным сканером Trivy.
> Это является **альтернативной реализацией** механизма private registry + vulnerability scanning.
Выполненные действия
Была подготовлена отдельная VM с Ubuntu 22.04 и IP:
```text
192.168.1.83
```
На ней были выполнены:
установка Docker;
установка Harbor;
включение встроенного Trivy scanner.
Далее:
выполнен `docker login` в Harbor;
создан проект `test`;
образ `shvirtd-example-python:python` был перетегирован:
```text
  192.168.1.83/test/shvirtd-example-python:python
  ```
выполнен `docker push` в Harbor;
запущено сканирование образа.
Результат сканирования
Всего уязвимостей: 105
Critical: 0
High: 12
Medium: 9
Low: 74
Исправимых: 7
Сканер: Trivy v0.69.3
Итоговая оценка: Высокая
Примеры найденных уязвимостей

`CVE-2026-4046` — `libc-bin`, `libc6`
`CVE-2025-69720` — `libncursesw6`, `libtinfo6`, `ncurses-base`, `ncurses-bin`
`CVE-2026-29111` — `libsystemd0`, `libudev1`
`CVE-2024-21272` — `mysql-connector-python 8.2.0`
`CVE-2025-4565` — `protobuf 4.21.12`
`CVE-2026-0994` — `protobuf 4.21.12`
`CVE-2024-47874` — `starlette 0.27.0`

<img width="3071" height="1747" alt="image" src="https://github.com/user-attachments/assets/2e9a82ee-bb72-4d0d-a6d1-3bf485695eb4" />

Вывод
Технически задача на развёртывание registry, загрузку образа и его сканирование выполнена, но использован локальный Harbor, а не Yandex Cloud Registry.
---
Задача 3
Условие
Создать `compose.yaml`, подключить `proxy.yaml` через `include` и описать сервисы:
`web`
`db`
Требования:
`web` должен работать в сети `backend` с IP `172.20.0.5`
`db` должен работать в сети `backend` с IP `172.20.0.10`
использовать `.env`
проект должен корректно отвечать на:
```bash
  curl -L http://127.0.0.1:8090
  ```
Созданный файл `compose.yaml`
```yaml
include:
  - proxy.yaml

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.python
    container_name: web
    restart: always
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_HOST: db
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
      DB_NAME: ${MYSQL_DATABASE}
    networks:
      backend:
        ipv4_address: 172.20.0.5

  db:
    image: mysql:8
    container_name: db
    restart: always
    env_file:
      - .env
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -uroot -p$${MYSQL_ROOT_PASSWORD} --silent"]
      interval: 10s
      timeout: 5s
      retries: 20
      start_period: 20s
    networks:
      backend:
        ipv4_address: 172.20.0.10
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```
Запуск
```bash
docker compose up -d --build
docker compose ps
curl -L http://127.0.0.1:8090
```
Результат
Ответ сервиса:
```text
"TIME: 2026-04-03 09:07:15, IP: 127.0.0.1"
```
Проверка MySQL
Подключение:
```bash
docker exec -ti db mysql -uroot -pYtReWq4321
```
SQL-команды:
```sql
show databases;
use virtd;
show tables;
SELECT * from requests LIMIT 10;
```
Результат:
```text
+----+---------------------+------------+
| id | request_date        | request_ip |
+----+---------------------+------------+
|  1 | 2026-04-03 09:07:15 | 127.0.0.1  |
+----+---------------------+------------+
```
Вывод
Задача 3 выполнена полностью:
`compose.yaml` создан;
`proxy.yaml` подключён через `include`;
стек поднят;
`curl` через `8090` работает;
данные записываются в MySQL.
---
Задача 4
Условие
Развернуть проект на облачной VM, установить Docker, написать bash-скрипт, который скачает fork в `/opt` и запустит проект целиком. Проверить внешний HTTP-доступ и повторить SQL-запрос.
Фактическая реализация
Вместо VM в Yandex Cloud был использован VPS с Ubuntu 22.04 и публичным IP:
```text
188.127.231.84
```

<img width="1643" height="1465" alt="image" src="https://github.com/user-attachments/assets/de4550b7-6c50-4748-bd0b-afbdb32be200" />

> Это не Yandex Cloud, а альтернативная облачная VM/VPS.
Bash-скрипт `/opt/deploy-shvirtd.sh`
```bash
#!/usr/bin/env bash
set -Eeuo pipefail

REPO_URL="https://github.com/victoryurochkin/shvirtd-example-python.git"
APP_DIR="/opt/shvirtd-example-python"
COMPOSE_FILE="${APP_DIR}/compose.yaml"

echo "[1/9] Проверка зависимостей"
command -v git >/dev/null 2>&1 || { echo "git not found"; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "docker not found"; exit 1; }
docker compose version >/dev/null 2>&1 || { echo "docker compose not found"; exit 1; }

echo "[2/9] Подготовка каталога /opt"
mkdir -p /opt

echo "[3/9] Клонирование или обновление репозитория"
if [ -d "${APP_DIR}/.git" ]; then
  git -C "${APP_DIR}" fetch --all
  git -C "${APP_DIR}" reset --hard origin/main
else
  git clone "${REPO_URL}" "${APP_DIR}"
fi

echo "[4/9] Проверка compose-файла"
if [ ! -f "${COMPOSE_FILE}" ]; then
  echo "ERROR: ${COMPOSE_FILE} not found"
  exit 1
fi

echo "[5/9] Переход в каталог проекта"
cd "${APP_DIR}"

echo "[6/9] Остановка старого стека, если был"
docker compose -f "${COMPOSE_FILE}" down || true

echo "[7/9] Удаление старых одиночных контейнеров, если были ручные тесты"
docker rm -f shvirtd-mysql >/dev/null 2>&1 || true
docker rm -f shvirtd-app >/dev/null 2>&1 || true

echo "[8/9] Сборка и запуск проекта"
docker compose -f "${COMPOSE_FILE}" up -d --build

echo "[9/9] Текущий статус"
docker compose -f "${COMPOSE_FILE}" ps
echo
echo "Локальная проверка:"
echo "curl -L http://127.0.0.1:8090"
```

<img width="2160" height="1371" alt="image" src="https://github.com/user-attachments/assets/45fd045c-aa70-4641-8a0e-fa6bd6fce060" />

Проверка локально на сервере
```bash
curl -L http://127.0.0.1:8090
```
Результат:
```text
TIME: 2026-04-03 09:44:32, IP: 127.0.0.1
```
Внешняя проверка
Внешний запрос к сервису:
```text
http://188.127.231.84:8090
```
Результат:
```text
TIME: 2026-04-03 09:45:52, IP: 185.239.49.167
```
<img width="982" height="285" alt="image" src="https://github.com/user-attachments/assets/be5d545c-8144-4dc0-955f-7447e56f1232" />


SQL-проверка
```sql
SELECT * from requests LIMIT 10;
```
Результат:
```text
+----+---------------------+----------------+
| id | request_date        | request_ip     |
+----+---------------------+----------------+
|  1 | 2026-04-03 09:44:16 | 127.0.0.1      |
|  2 | 2026-04-03 09:44:32 | 127.0.0.1      |
|  3 | 2026-04-03 09:45:52 | 185.239.49.167 |
+----+---------------------+----------------+
```
<img width="1525" height="1500" alt="image" src="https://github.com/user-attachments/assets/b197e033-77ef-4a23-8698-2e2c6af684f7" />

Вывод
Проект успешно задеплоен на облачную VM/VPS, внешний доступ по `8090` работает, запись в БД при внешнем запросе подтверждена.
---
Задача 5 (*)
Условие
Написать и задеплоить bash-скрипт резервного копирования MySQL в `/opt/backup` с помощью контейнера `schnitzler/mysqldump`, протестировать ручной запуск, настроить запуск раз в минуту, не светить логин/пароль в git.
Статус
В ходе этой сессии задача 5 не была доведена до полной реализации:
рабочий скрипт резервного копирования не был окончательно создан и протестирован;
cron/systemd timer не был настроен;
скриншот с несколькими резервными копиями не был получен.
Вывод
Задача 5 в рамках данной работы не завершена.
---
Задача 6
Условие
Скачать образ `hashicorp/terraform:latest`, найти и извлечь бинарный файл `/bin/terraform` с помощью `dive` и `docker save`.
Выполненные действия
Скачивание образа:
```bash
docker pull hashicorp/terraform:latest
```
Просмотр через `dive`:
был найден файл:
```text
  /bin/terraform
  ```
Сохранение образа:
```bash
docker save -o terraform_latest.tar hashicorp/terraform:latest
```
Распаковка и анализ `manifest.json` показали список слоёв.
Нужный слой:
```text
blobs/sha256/f491828cf45bec9cdfb7d00792af9367df8ffe018d010abc1b79cea9de7931aa
```
Именно в нём находился файл:
```text
bin/terraform
```
Извлечение бинарника:
```bash
tar -xf /tmp/terraform-image/blobs/sha256/f491828cf45bec9cdfb7d00792af9367df8ffe018d010abc1b79cea9de7931aa -C /tmp/terraform-extract
```
Проверка:
```bash
/tmp/terraform-extract/bin/terraform version
```
Результат:
```text
Terraform v1.14.8
on linux_amd64
```

<img width="2513" height="1529" alt="image" src="https://github.com/user-attachments/assets/24b43ffa-4a0d-4a77-b8b2-4938f1dab292" />

Вывод
Задача 6 выполнена полностью.
---
Задача 6.1
Условие
Добиться аналогичного результата, используя `docker cp`.
Выполненные действия
Создание контейнера:
```bash
docker create --name terraform-cp hashicorp/terraform:latest
```
Копирование бинарника:
```bash
docker cp terraform-cp:/bin/terraform /tmp/terraform-cp
```
Проверка:
```bash
file /tmp/terraform-cp
/tmp/terraform-cp version
```
Результат:
```text
/tmp/terraform-cp: ELF 64-bit LSB executable, x86-64
Terraform v1.14.8
on linux_amd64
```
Удаление контейнера:
```bash
docker rm terraform-cp
```

<img width="2523" height="596" alt="image" src="https://github.com/user-attachments/assets/0d3785fd-b72c-426a-bdcc-c86221b53c85" />

Вывод
Задача 6.1 выполнена полностью.
---
Задача 6.2 (**)
Условие
Предложить способ извлечь файл из контейнера, используя только `docker build` и любой `Dockerfile`.
Использованный Dockerfile
```dockerfile
FROM hashicorp/terraform:latest AS source

FROM scratch
COPY --from=source /bin/terraform /terraform
```
Выполненные действия
```bash
docker build --output type=local,dest=/tmp/terraform-export .
```
В результате в каталоге `/tmp/terraform-export` появился файл:
```text
terraform
```
Проверка:
```bash
file /tmp/terraform-export/terraform
/tmp/terraform-export/terraform version
```
Результат:
```text
Terraform v1.14.8
on linux_amd64
```

<img width="2529" height="367" alt="image" src="https://github.com/user-attachments/assets/abe9437f-f7d0-4419-8ab0-a04edeeb698e" />


Вывод
Задача 6.2 выполнена полностью.
---
Задача 7 (***)
Условие
Запустить python-приложение с помощью `runC`, не используя Docker или containerd для самого запуска приложения.
Принятый способ
Для реализации был выбран компромиссный, но рабочий вариант:
MySQL оставлена как внешний сервис в Docker;
web-приложение запущено напрямую через runC.
Выполненные действия
Установлен standalone-бинарник `runc`:
```bash
   /usr/local/sbin/runc
   ```
Подготовлен OCI bundle:
```text
   /opt/runc-web
   ```
Создан `rootfs` через `debootstrap`.
Внутрь `rootfs` установлены:
`python3`
`python3-pip`
`fastapi==0.104.1`
`uvicorn[standard]==0.24.0`
`mysql-connector-python==8.2.0`
В `rootfs` скопирован файл:
```text
   /app/main.py
   ```
В `config.json` были изменены:
`cwd = /app`
команда запуска:
```text
     python3 -m uvicorn main:app --host 0.0.0.0 --port 5000
     ```
переменные окружения:
`DB_HOST=172.20.0.10`
`DB_USER=app`
`DB_PASSWORD=QwErTy1234`
`DB_NAME=virtd`
удалён `network` namespace
`root.readonly = false`
Запуск приложения:
```bash
   sudo /usr/local/sbin/runc run web-runc
   ```
Результат
Приложение стартовало успешно:
```text
INFO:     Started server process [1]
INFO:     Waiting for application startup.
Приложение запускается...
Соединение с БД установлено и таблица 'requests' готова к работе.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:5000
```
Проверка:
```bash
curl http://127.0.0.1:5000
```
Ответ:
```text
TIME: 2026-04-03 10:30:36, IP: похоже, что вы направляете запрос в неверный порт...
```
Это является штатным поведением приложения при прямом запросе на `5000`.
SQL-проверка:
```sql
select * from requests order by id desc limit 5;
```
Результат:
```text
+----+---------------------+------------+
| id | request_date        | request_ip |
+----+---------------------+------------+
|  2 | 2026-04-03 10:30:36 | NULL       |
|  1 | 2026-04-03 09:07:15 | 127.0.0.1  |
+----+---------------------+------------+
```

<img width="1777" height="1049" alt="image" src="https://github.com/user-attachments/assets/192c91ae-acca-49e7-aed1-32ee70db3c34" />

<img width="2518" height="499" alt="image" src="https://github.com/user-attachments/assets/688edc6b-1fcc-48fe-ba22-2f0c46dac7b0" />

Вывод
Задача 7 выполнена:
приложение `web` было запущено через `runC`;
приложение стартовало и подключилось к MySQL;
HTTP-запрос обработан;
новая запись появилась в базе данных.
---
Итог
В рамках работы были выполнены задачи 0, 1, 3, 4, 6, 6.1, 6.2, 7.  
Задача 2 выполнена в виде альтернативной реализации через Harbor + Trivy вместо Yandex Cloud Registry.  
Задача 5 в рамках текущей сессии не была завершена.
---
Ссылка на fork
```text
https://github.com/victoryurochkin/shvirtd-example-python
```
