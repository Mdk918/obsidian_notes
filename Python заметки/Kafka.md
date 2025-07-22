```YAML
services:
  kafka-0:
    image: bitnami/kafka:3.4
    ports:
      - "9094:9094"
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-0:9093,1@kafka-1:9093,2@kafka-2:9093
      - KAFKA_KRAFT_CLUSTER_ID=abcdefghijklmnopqrstuv
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka-0:9092,EXTERNAL://127.0.0.1:9094
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
   
    volumes:
      - kafka_0_data:/bitnami/kafka
   
   
  kafka-1:
    image: bitnami/kafka:3.4
    ports:
      - "9095:9095"
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-0:9093,1@kafka-1:9093,2@kafka-2:9093
      - KAFKA_KRAFT_CLUSTER_ID=abcdefghijklmnopqrstuv
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9095
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka-1:9092,EXTERNAL://127.0.0.1:9095
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
   
    volumes:
      - kafka_1_data:/bitnami/kafka
   
  kafka-2:
    image: bitnami/kafka:3.4
    ports:
      - "9096:9096"
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_NODE_ID=2
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-0:9093,1@kafka-1:9093,2@kafka-2:9093
      - KAFKA_KRAFT_CLUSTER_ID=abcdefghijklmnopqrstuv
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9096
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka-2:9092,EXTERNAL://127.0.0.1:9096
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
    volumes:
      - kafka_2_data:/bitnami/kafka
  ui:
  image: provectuslabs/kafka-ui:v0.7.0
  ports:
    - "8080:8080"
  environment:
    - KAFKA_CLUSTERS_0_BOOTSTRAP_SERVERS=kafka-0:9092
    - KAFKA_CLUSTERS_0_NAME=kraft

volumes:
  kafka_0_data:
  kafka_1_data:
  kafka_2_data:
 
```

- `KAFKA_ENABLE_KRAFT=yes` — разрешить использование протокола KRaft;
- `KAFKA_CFG_PROCESS_ROLE=broker,controller` — узел может входить в кворум как контроллер, но также как брокер обеспечивает хранение разделов и добавление новых сообщений в разделы;
- `KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER` — определение типа слушателя для публикации контроллера (используется далее в KAFKA_CFG_LISTENERS и KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP);
- `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-0:9093` — обозначаем идентификатор контроллера (как части кворума) и его адрес и порт (здесь нужно перечислить адреса всех известных контроллеров);
- `KAFKA_KRAFT_CLUSTER_ID=somevalue` — идентификатор кластера (должен быть одинаковым у всех контроллеров и брокеров), его мы получим после первого запуска;
- `KAFKA_CFG_LISTENERS` — этот параметр определяет, на каких адресах и портах брокер Kafka будет прослушивать входящие соединения. Это включает в себя все типы соединений, внутри докер-сети или локальные. По сути, здесь мы говорим Kafka, где искать входящие запросы. В вашем конфигурационном файле вы указываете три слушателя: `PLAINTEXT://:9092`, `CONTROLLER://:9093` и `EXTERNAL://:9094`. Первые два предназначены для внутренних коммуникаций внутри Docker, а последний — для внешних соединений с хост-машиной;
- `KAFKA_CFG_ADVERTISED_LISTENERS` — этот параметр указывает на те адреса и порты, которые брокер Kafka будет «объявлять» или «рекламировать» своим клиентам. Это важно, потому что клиенты Kafka должны знать, как подключиться к брокеру. Если ваш брокер Kafka работает внутри докер-сети, этот параметр может указывать на адрес в этой сети. Если же вам нужно подключаться к брокеру локально, здесь вы можете указать соответствующий порт и протокол. В вашем конфигурационном файле вы объявляете два слушателя `PLAINTEXT://kafka-0:9092` и `EXTERNAL://127.0.0.1:9094`. Первый слушатель используется для обслуживания внутренних коммуникаций в Docker, второй — для обслуживания внешних коммуникаций с хост-машиной;
- `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` — этот параметр позволяет вам определить, какой протокол безопасности будет использоваться для каждого прослушивателя, которые вы определили в параметрах выше. Это важно для обеспечения безопасности ваших данных при передаче между брокером и клиентами. В вашем конфигурационном файле вы указываете, что все ваши слушатели (`CONTROLLER`, `EXTERNAL`, `PLAINTEXT`) будут использовать протокол `PLAINTEXT`, который не предполагает шифрования данных.

Теперь по адресу `http://localhost:8080` у нас доступен интерфейс для управления Kafka.