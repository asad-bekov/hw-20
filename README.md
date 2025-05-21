# Домашнее задание к занятию 4 «Оркестрация группой Docker контейнеров на примере Docker Compose»

*Асадбеков Асадбек*

## Задача 1 — Docker: установка и создание образа

- Установлены Docker и Docker Compose  
- Добавлены зеркала DockerHub  
- Создан кастомный `Dockerfile` на основе `nginx:1.21.1`  
- Заменена страница `index.html`  
- Образ собран и залит в Docker Hub:

** Образ:**  
[https://hub.docker.com/r/asadnet/custom-nginx](https://hub.docker.com/repository/docker/asadnet/custom-nginx/general)

**Скриншоты:**  
- `docker build` и `docker push`

- Проверка содержимого контейнера
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/1.png)

- Отображение HTML-страницы
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/2.png)

---

## Задача 2 — Запуск контейнера и диагностика

- Запущен контейнер с именем `custom-nginx-t2`  
- Порт 8080 проброшен на хост  
- Переименование контейнера  
- Проверка сетевого подключения, логов, содержимого страницы

**Скриншот:**  

![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/3.png)

---

## Задача 3 — Работа с контейнером изнутри

- Подключение через `docker attach`  
- Объяснение, почему контейнер остановился после `Ctrl+C`  
- Перезапуск контейнера  
- Подключение к `bash`, установка `nano`  
- Редактирование `/etc/nginx/conf.d/default.conf`, замена порта 80 на 81  
- Перезапуск nginx внутри контейнера  
- Анализ проблемы с проброшенным портом  
- Удаление контейнера `--force`
  
*Проверьте вывод команд: `ss -tlpn | grep 127.0.0.1:8080`, `docker port custom-nginx-t2`, `curl http://127.0.0.1:8080`. Кратко объясните суть возникшей проблемы.*

### Объяснение

**Почему контейнер остановился после `docker attach` → `Ctrl+C`:**  
Когда я подключился к контейнеру через `docker attach` и нажал `Ctrl+C`, это отправило сигнал `SIGINT` основному процессу `nginx`, работающему в режиме `foreground`. Завершение этого процесса остановило весь контейнер, так как в Docker контейнер работает только пока жив его главный процесс (PID 1).

**Почему `curl http://127.0.0.1:8080` перестал работать после изменения порта:**  
Я изменил конфигурацию nginx с `listen 80;` на `listen 81;`. Но при запуске контейнера был настроен проброс: `-p 127.0.0.1:8080:80`, т.е. внешний порт 8080 подключается к внутреннему 80. После изменения nginx больше не слушает порт 80, поэтому проброс больше не работает — отсюда ошибка подключения через `curl`.

**Скриншоты:**  
- `nano`, `nginx -s reload`, `curl` внутри и снаружи  
- `ss -tlpn`, `docker port`, `docker ps -a`

![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/3.png)
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/4.png)
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/5.png)
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/6.png)
---

## Задача 4 — Том между контейнерами

- Запущен контейнер `centos:7` и `debian` с общим volume `-v $(pwd):/data`  
- Создан файл в одном контейнере  
- Добавлен файл на хосте  
- Проверка в другом контейнере

**Скриншоты:**  
- Создание файлов
- `ls -l /data` и `cat` внутри контейнера `debian`
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/6.png)
---

## Задача 5 — Compose, Portainer и Registry

- Создан `compose.yaml` и `docker-compose.yaml`  
- Запуск `Portainer` и `Registry`  
- Образ `custom-nginx` загружен в `localhost:5000`  
- Portainer доступен на `http://127.0.0.1:9000`  
- Через Web UI развернут стек с `nginx` из локального registry  
- Скриншот `Inspect` контейнера (от AppArmorProfile до Driver)  
- Удаление `docker-compose.yaml`, повторный запуск с `compose.yaml`  
- Обработка предупреждения `remove-orphans`

###Пояснение по `include` и `docker-compose.yaml`

**Что означает предупреждение после удаления `docker-compose.yaml`:**  

В `compose.yaml` была директива `include`, ссылающаяся на `docker-compose.yaml`. После удаления включённого файла Docker не нашёл сервисы, которые ранее запускались через него, и выдал предупреждение, что они больше не управляются текущим манифестом. Это может привести к "висячим" (orphaned) контейнерам, которые не контролируются Compose.

**Что сделала команда `docker compose down --remove-orphans`:**  
Она остановила все сервисы, описанные в `compose.yaml`, и удалила "осиротевшие" контейнеры, ранее запущенные через удалённый манифест. Это гарантирует, что все ресурсы удалены корректно, и Docker Compose работает в чистом состоянии.

**Скриншоты:**  

- Web-интерфейс Portainer, Stack, Inspect  
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/7.png)
![alt text](https://github.com/asad-bekov/hw-20/raw/main/img/8.png)
---

## ИТОГ:

- Все задачи выполнены  
- Все образы и контейнеры протестированы  
- Установлены связи между томами, сетями, образами и Portainer  
- Образ доступен в Docker Hub и локально

---

## Примечания

- Вместо `centos:latest` использован `centos:7` (официальная поддержка закончилась)
- Использован `tail -f /dev/null` для удержания контейнеров активными
- Portainer настроен на `host`-сети для удобства доступа без прокси

---
