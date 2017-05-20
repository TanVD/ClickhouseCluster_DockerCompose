Установите Docker Engine и Docker Compose.

Выполните docker compose up. В результате будут подняты три Zookeeper и три Clickhouse к ним подключенные. Для подключения к Zookeeper Clickhouse использует параметры  из config.xml (блок zookeeper). Zookeeper не нужно знать о доменном имени и портах Clickhouse, Clickhouse всю информацию анонсирует Zookeeper cfv.

Наружу будут вывешены порты 9001 (у первой реплики), 9002 (у второй реплики), 9003 (у третьей реплики) для clickhouse-client. Используйте для подключения команду:

`clickhouse-client --port={port-number}`

Поочерёдно на каждой реплике выполните команду создания реплицируемой таблицы (замените {replica-number} на порядковый номер реплики):

`CREATE TABLE ExampleTable (ExDate default today(), SomeText String) ENGINE = ReplicatedMergeTree('/clickhouse/tables/example', '{replica-number}', ExDate, (ExDate, SomeText), 8192)`

К сожалению, DDL на данный момент не реплицируются.

Теперь можно создать Distributed таблицу. Чтение с неё будет автоматически распараллеливаться в случае шардирования. В случае одного шарда можно применить load_balancing (что тоже неплохо, хотя читать параллельно с нескольких реплик Clickhouse всё-таки не умеет):

`CREATE TABLE DistributedExampleTable (ExDate default today(), SomeText String) ENGINE = Distributed(example, currentDatabase(), ExampleTable)`

Теперь вы можете отключить некоторое количество узлов кластера. Кластер толерантен к потере половины ZooKeeper с округлением вниз и к потере любого числа Clickhouse, кроме всех (при условии запроса нужного Clickhouse).