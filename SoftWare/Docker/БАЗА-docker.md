[hub.docker.com](https://hub.docker.com/) - дефолтный репозиторий для образов docker

### Глоссарий

- **Docker Client** - любой клиент для управления образами/контейнерами.
- **Docker Deamon** - собственно, выполняет команды от клиента
- **Docker Registry** - хранилище образов Docker. Например: hub.docker.com
- **Образ** -  
- **Контейнер** -  


### Работа с образами
- `docker images` - показать список установленных образов 
- `docker run <image_name>` - запустить контейнер из образа
- `docker run --name <container_name>  <image_name>` - запустить контейнер из образа с указанием имени для создаваемого контейнера.
### Работа с контейнерами
- `docker ps` - просмотр запущенных контейнеров
- `docker exec [опции] <контейнер> <команда>` - войти в оболочку работающего контейнера
-  `docker start <контейнер>[контейнер...]`- запуск существующего контейнера(ов) по имени контейнера или ID 
-  `docker stop <контейнер>` - остановить работающий контейнер.
### Примеры команд



- Запуск веб-сервера nginx из образа
docker run -d -p 80:80 --name mynginx nginx

docker run -d -p 5432:5432 --name mypostgres -e POSTGRES_USER=demo -e POSTGRES_PASSWORD=demo postgres:15

