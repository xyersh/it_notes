
[Урок](https://www.youtube.com/watch?v=wgA3yntF_ek)
[Пример проекта](https://github.com/PythonHubStudio/docker-course-demo)
## Docker

### Тезисы
- реестр образов - hub.docker.com, образы на компе - тоже реестр
- репозиторий - это коллекция версий отдельного образа в реестре 


### Команды docker
- docker version
- docker run hello-world
- docker images
- docker pull <image[:version]>
- docker ps
- docker ps -a
- docker ps -a -q - список ID ВСЕХ контейнеров
- docker rm <container_hash>
- docker container prune - удалить все неработащие контейнеры
- docker run --rm <image> -запуск образа с последующим удалением
- docker run --name <name> <image> - запуск образа с заданием имени новому контейнеру
- docker run --it <image> - запуск образа в интерактивном режиме
- docker run -d <image> - запуск образа в detouched режиме (противоположность -it)
- docker start <container>
- docker start -i <contaner>
- docker run -p <[IP:]host-port>:<container-port> <image> - запуск контейнера с пробросом портов. Можно указывать IP, по которому приложение будет доступно.
- docker run -v <host_path>:<container_path> - запуск контейнера с в bind mount (с привязкой к диреториям хоста)
```
 docker run -d -p 127.0.0.1:8888:80 -v ${pwd}\dev\site:/usr/share/nginx/html --name server nginx
```
- docker stop <container> - остановка работающего контейнера

- docker inspect <container> - получить инфо о работающем контейнере
- docker inspect <container> | grep <TAG> - просмотреть инфо конкретного раздела работающего контейнера (linux)
- docker inspect <container> | findstr <TAG> - просмотреть инфо конкретного раздела работающего контейнера (Windows)

- docker exec -it <container> <app> - открыть терминал работающего контейнера с запуском оболочки

- docker rmi <image[:version]>

- docker build . <new_image_name:version>

- docker stats - статистика по работающим контейнерам

- docker diff <container> - показывает изменения в контейнере (установки программ, удаления) по отношению к оригинальному образу

- docker commit <container> <new_image[:version]> - создать новый образ на базе существующего контейнера

- docker logs <container> - просмотреть логи контейнера

- docker system prune [-a] - удалить все неработающие контейнеры. и кэш
	- -a - удалить также образы 

- docker builder prune [-a] - удалить кэш сборок

- docker compose up - запуск файла docker-compose.yml находящегося в текущей директории 
- docker compose  up -d -  ... в detauched режиме 
- docker compose -f <compose_file> up - запуск сборки из файла с указанным именем.
- docker compose down - остановка приложения с последующим удалением запущенных контейнеров и созданных сетей
-

### Команды linux
- apt-get update
- apt-get install -y curl
- cat <file_name> - показать все  содержимое файла
- head <file_name> - показать содержимое первых строк файла
- tail <file_name> - показать содержимое последних строк файла
- more <file_name> - показать содержимое файла с возможностью прокрутки
- pwd - текущая рабочая директория

### Dockerfile
- FROM
- WORKDIR
- COPY
- RUN
- ENV
- CMD

Каждая команда RUN = один дополнительный слой в образе

### Перенос строки в командной оболочке
- `\` - linux
- \` - windows powershell
- `^` - windows cmd



### nginx
- `/etc/nginx/conf.d/default.conf` - здесь настройки сервера
	- location
		- root - директория для статических файлов
		- index - указываются 
