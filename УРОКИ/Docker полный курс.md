
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


- docker stop <container> - остановка работающего контейнера

- docker rmi <image[:version]>

- docker build . <new_image_name:version>

- docker stats - статистика по работающим контейнерам

- docker diff <container> - показывает изменения в контейнере (установки программ, удаления) по отношению к оригинальному образу

- docker commit <container> <new_image[:version]> - создать новый образ на базе существующего контейнера

- docker logs <container> - просмотреть логи контейнера

- docker system prune [-a] - удалить все неработающие контейнеры. и кэш
	- -a - удалить также образы 

- docker builder prune [-a] - удалить кэш сборок

### Команды linux
- apt-get update
- apt-get install -y curl


### Dockerfile
- FROM
- WORKDIR
- COPY
- RUN
- ENV
- CMD

Каждая команда RUN = один дополнительный слой в образе
	