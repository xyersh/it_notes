- Docker registry - любое ПО предназначенное для хранения и пуллинга  docker-образов. 
- Registry - это не часть dicker engine, а отдельная софтина 
- Запустить  приватный registry на хосте можно командой:
```bash
docker run -d -p 5000:5000 --name=registry registry:2
```
- Закачать образ в свой registry можно командой:
```bash
docker image tag my-image localhost:5000/my-image
```

- Отправить образ в локальный registry
```bash
docker push localhost:5000/my-image
```

- Получить образ из локального registry на том  же хосте
```go
docker pull localhost:5000/my-image
```

- Получить образ из удаленного registry можно с указанием IP удаленного registry.
```bash
docker pull 192.168.20.51:5000/my-image
```
