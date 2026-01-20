# Netology_bogatyrevam
```bash
$ docker build -t my-nginx .
$ docker run -d --name “bogatyrevam-custom-nginx-t2” -p 127.0.0.1:8080:80 my-nginx:latest
$ docker rename “bogatyrevam-custom-nginx-t2” my-nginx-t2
$ date +"%d-%m-%Y %T.%N %Z" ; sleep 0.150 ; docker ps ; ss -tlpn | grep 127.0.0.1:8080 ; docker logs custom-nginx-t2 -n1 ; docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html
$ docker run -d --name “bogatyrevam-custom-nginx-t2” -p 127.0.0.1:8080:80 my-nginx:latest
$ docker rename “bogatyrevam-custom-nginx-t2” my-nginx-t2
$ docker attach my-nginx-t2
$ docker start my-nginx-t2
$ docker exec -it my-nginx-t2 bash
$ docker port my-nginx-t2
```
Объяснение возникшей проблемы
В контейнере nginx теперь слушает порт 81, а не 80.
На хосте проброшен порт 8080 на порт 80 контейнера.
Поэтому запросы на 127.0.0.1:8080 не доходят до nginx, так как внутри контейнера nginx не слушает 80 порт.
В итоге curl на 127.0.0.1:8080 не получает ответ.

```bash
$ docker stop my-nginx-t2
$ docker commit my-nginx-t2 my-nginx-modified
$ docker rm my-nginx-t2
$ docker run -d --name my-nginx-t2 -p 127.0.0.1:8080:81 my-nginx-modified
$ docker rm -f my-nginx-t2
```
Запуск первого контейнера из образа centos с монтированием текущего каталога в /data
`$ docker run -d -v $(pwd):/data --name centos-container centos`

Запуск второго контейнера из образа debian с монтированием текущего каталога в /data
`$ docker run -d -v $(pwd):/data --name debian-container debian`

Подключение к первому контейнеру и создание файла в /data
`$ docker exec -it centos-container bash`
Внутри контейнера:
`$ echo “Hello from CentOS container” > /data/centos_file.txt`
`$ exit`

Добавление файла в текущий каталог на хосте
В терминале хоста:
`$ echo “Hello from host” > host_file.txt`

Подключение ко второму контейнеру и просмотр файлов в /data
`$ docker exec -it debian-container bash`
Внутри контейнера:
```bash
$ ls -l /data
$ cat /data/centos_file.txt
$ cat /data/host_file.txt
$ exit
```

Полная установка docker compose
```bash
$ apt-get update
$ apt-get install docker-compose-plugin
$ apt-get install ca-certificates curl gnupg
$ mkdir -p /etc/apt/keyrings
$ curl -fsSL download.docker.com...ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ curl -fsSL download.docker.com...ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ echo
“deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] download.docker.com...nux/ubuntu
$(lsb_release -cs) stable” | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ apt-get update
$ apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
$ docker --version
$ docker compose version
```
Задача 5
Создайте отдельную директорию(например /tmp/netology/docker/task5) и 2 файла внутри него. “compose.yaml” с содержимым:
version: "3"
services:
portainer:
network_mode: host
image: portainer/portainer-ce:latest
volumes:
- /var/run/docker.sock:/var/run/docker.sock

“docker-compose.yaml” с содержимым:
```yml
version: "3"
services:
registry:
image: registry:2
ports:
- "5000:5000"
```

Решение
```bash
$ mkdir -p /tmp/netology/docker/task5
$ cd /tmp/netology/docker/task5
$ nano compose.yaml
$ nano docker-compose.yaml
```
И выполните команду `docker compose up -d`. Какой из файлов был запущен и почему? (подсказка: docs.docker.com...mpose-file )

Решение
✔ Container task5-portainer-1 Created

Отредактируйте файл compose.yaml так, чтобы были запущенны оба файла. (подсказка: docs.docker.com...4-include/)

Решение
`$ nano compose.yaml`
```yml
version: “3.9”
include:
```
`docker-compose.yaml`
```yml
services:
portainer:
network_mode: host
image: portainer/portainer-ce:latest
volumes:
- /var/run/docker.sock:/var/run/docker.sock
```

Выполните в консоли вашей хостовой ОС необходимые команды чтобы залить образ custom-nginx как custom-nginx:latest в запущенное вами, локальное registry. Дополнительная документация: distribution.github.io...deploying/

Решение
`$ docker build -t my-nginx:latest -f /home/admin/dockerfile /home/admin`
Далее идём в настройки GitHub:
`Settings` → `Developer settings` → `Personal access tokens` → `Tokens (classic)` → `Generate new token`
Выбираем права:
`read:packages`
`write:packages (если нужно пушить)`
`delete:packages (опционально)`
Генерим токен и вставляем его после команды
`$ docker login ghcr.io -u dragondamage`
Получаем ответ: `Login Succeeded`

```bash
$ docker tag my-nginx:latest 127.0.0.1:5000/my-nginx:latest
$ docker push 127.0.0.1:5000/my-nginx:latest
```
Откройте страницу “https://127.0.0.1:9000” и произведите начальную настройку portainer.(логин и пароль адмнистратора)

`http://158.160.55.184:9000`
Откройте страницу “http://127.0.0.1:9000/#!/home”, выберите ваше local окружение. Перейдите на вкладку “stacks” и в “web editor” задеплойте следующий компоуз:

`http://158.160.55.184:9000/#!/home`
Вкладка [stacks] -> [Create stack] -> [web editor]

```yml
version: ‘3’

services:
nginx:
image: 127.0.0.1:5000/custom-nginx
ports:
- “9090:80”
```
Меняем custom-nginx на my-nginx
Перейдите на страницу “http://127.0.0.1:9000/#!/2/docker/containers”, выберите контейнер с nginx и нажмите на кнопку "inspect"
В представлении <> Tree разверните поле “Config” и сделайте скриншот от поля “AppArmorProfile” до “Driver”

`http://158.160.55.184:9000/#!/3/docker/containers`
Удалите любой из манифестов компоуза(например compose.yaml). Выполните команду “docker compose up -d”. Прочитайте warning, объясните суть предупреждения и выполните предложенное действие. Погасите compose-проект ОДНОЙ(обязательно!!) командой.
```bash
$ rm compose.yaml
$ docker compose down
```
