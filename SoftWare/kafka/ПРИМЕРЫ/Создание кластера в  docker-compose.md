
### Создание файла **docker-compose.yml** 

```yml
  
services:
  # --- Брокер 1 ---
  kafka-1:
    image: cleanstart/kafka:latest
    container_name: kafka-1
    ports:
      - "9092:9094" # Внешний порт для подключения с хост-машины
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka-1:9092,EXTERNAL://localhost:9092
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_KRAFT_CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk
      - ALLOW_PLAINTEXT_LISTENER=yes
    volumes:
      - kafka_data_1:/cleanstart/kafka

  # --- Брокер 2 ---
  kafka-2:
    image: cleanstart/kafka:latest
    container_name: kafka-2
    ports:
      - "9093:9094"
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka-2:9092,EXTERNAL://localhost:9093
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
      - KAFKA_CFG_NODE_ID=2
      - KAFKA_KRAFT_CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk
      - ALLOW_PLAINTEXT_LISTENER=yes

    volumes:
      - kafka_data_2:/cleanstart/kafka

  # --- Брокер 3 ---
  kafka-3:
    image: cleanstart/kafka:latest
    container_name: kafka-3
    ports:
      - "9094:9094"

    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka-3:9092,EXTERNAL://localhost:9094
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
      - KAFKA_CFG_NODE_ID=3
      - KAFKA_KRAFT_CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk
      - ALLOW_PLAINTEXT_LISTENER=yes

    volumes:
      - kafka_data_3:/cleanstart/kafka

  

  # --- Kafka UI ---

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    ports:
      - "8888:8080"
    environment:
      - DYNAMIC_CONFIG_ENABLED=true
      - KAFKA_CLUSTERS_0_NAME=local-kraft-cluster
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-1:9092,kafka-2:9092,kafka-3:9092
    depends_on:
      - kafka-1
      - kafka-2
      - kafka-3

  

volumes:
  kafka_data_1:
  kafka_data_2:
  kafka_data_3:
```

### Разбор и разъяснение конфигурации

#### Отказ от Zookeeper (Режим KRaft)

- `KAFKA_ENABLE_KRAFT=yes`: Включает режим KRaft.
- `KAFKA_CFG_PROCESS_ROLES=broker,controller`: Указывает, что каждый узел выполняет обе роли: хранит данные (broker) и участвует в выборе лидера/управлении мета-данными (controller). Для локального/тестового кластера из 3 узлов это оптимальное решение.


#### Идентификация и Кворум (Quorum)

- `KAFKA_CFG_NODE_ID`: Уникальный числовой идентификатор узла (1, 2 и 3).
- `KAFKA_KRAFT_CLUSTER_ID`: Уникальный идентификатор кластера (в формате Base64 UUID). **Он должен быть абсолютно одинаковым для всех трех брокеров**, иначе они не смогут объединиться в кластер. _В файле указан заранее сгенерированный ID. Если вы хотите сгенерировать свой, выполните команду: `docker run --rm bitnami/kafka:3.7 /opt/bitnami/kafka/bin/kafka-storage.sh random-uuid`._
- `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS`: Список всех контроллеров в формате `NODE_ID@hostname:port`. Это "адресная книга" для выбора лидера кворума.


#### Настройка Слушателей (Listeners)

Настройка сетевых портов — самая сложная часть в Docker для Kafka. Мы используем два внутренних и один внешний слушатель:
- `PLAINTEXT://:9092` — для внутреннего общения брокеров между собой и для подключения **Kafka UI** (по именам контейнеров `kafka-1`, `kafka-2`).
- `CONTROLLER://:9093` — исключительно для внутреннего обмена мета-данными между контроллерами.
- `EXTERNAL://:9094` — для подключения с вашей **хост-машины** (например, из ваших приложений или консольных утилит).
- В `ports` мы маппим внутренний порт `9094` на разные внешние порты хоста: `9092`, `9093`, `9094`, чтобы избежать конфликтов.

#### Kafka UI

- Подключается к кластеру, используя внутренние DNS-имена Docker (`kafka-1:9092,kafka-2:9092,kafka-3:9092`).
- Доступен в браузере по адресу `http://localhost:8888`.

### Запуск кластера

1. Откройте терминал в директории с файлом `docker-compose.yml`.
2. Запустите контейнеры в фоновом режиме:
```bash
docker-compose up -d
```

3. Проверьте, что все контейнеры успешно стартовали и не находятся в состоянии `Restarting`:
```bash
docker-compose ps
```

> Примечание: Kafka UI может показывать статус брокеров как "Offline" первые 10-20 секунд, пока брокеры инициализируют KRaft кворум и обмениваются мета-данными. Просто обновите страницу.


### Проверка работоспособности
**Через Web-интерфейс:** Откройте браузер и перейдите по адресу [http://localhost:8080](http://localhost:8080).
- Перейдите во вкладку **Brokers**. Вы должны увидеть все 3 узла (kafka-1, kafka-2, kafka-3) со статусом `Online`.
- Перейдите во вкладку **Topics** и создайте тестовый топик, чтобы убедиться, что кластер принимает данные.

**Через логи (опционально):** Посмотрите логи первого брокера, чтобы убедиться, что он успешно выбрал лидера контроллера:
```bash
docker-compose logs -f kafka-1
```
В логах вы должны увидеть сообщения вроде `Kafka Server started` и `Registered broker 1 in KRaft mode`.


### Остановка и очистка
Чтобы остановить кластер:
```bash
docker-compose down
```

Если вы хотите полностью удалить кластер **вместе с сохраненными данными** (топиками и мета-данными), используйте флаг `-v`:
```bash
docker-compose down -v
```