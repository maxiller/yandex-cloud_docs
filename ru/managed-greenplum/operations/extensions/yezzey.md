---
title: Как использовать расширение {{ YZ }} в {{ mgp-full-name }}
description: Следуя данной инструкции, вы сможете использовать гибридное хранилище в {{ mgp-name }}.
---

# Использование {{ YZ }} в {{ mgp-name }}

Расширение {{ YZ }} позволяет использовать гибридное хранилище в кластере {{ mgp-name }} для хранения таблиц, [оптимизированных для добавления данных](../../concepts/tables.md) (append-optimized tables).

Вы можете [выгрузить](#offload-relation) редко используемые таблицы такого типа в холодное хранилище и продолжить работать с ними, как с обычными таблицами. При необходимости таблицы можно [загрузить обратно](#load-relation) в хранилище кластера.

Подробнее о типах хранилищ и расширении {{ YZ }} см. в разделе [Гибридное хранилище](../../concepts/hybrid-storage.md).

## Установить расширение {{ YZ }} в кластер {{ GP }} {#install-extension}

1. [Включите опцию для использования гибридного хранилища](../update.md#change-additional-settings), если этого не было сделано при [создании кластера](../cluster-create.md).

    {% note warning %}

    Эту опцию нельзя отключить после сохранения настроек кластера.

    {% endnote %}

1. [Подключитесь](../connect.md) к базе данных от имени владельца или пользователя, имеющего в базе данных разрешение `CREATE`, и выполните запрос:

    ```sql
    CREATE EXTENSION yezzey;
    ```

1. Проверьте, что расширение было установлено:

    ```sql
    SELECT extname FROM pg_extension;
    ```

## Получить информацию, где размещена таблица {#describe-relation}

Чтобы узнать, где размещена таблица и ее сегментные файлы — в хранилище кластера или в холодном хранилище:

1. [Подключитесь](../connect.md) к базе данных от имени владельца или пользователя, имеющего в базе данных разрешение `SELECT`.

1. Выполните один из запросов, чтобы узнать, где размещена таблица:

    * С указанием имени таблицы:

        ```sql
        SELECT * FROM yezzey_offload_relation_status('<имя_таблицы>');
        ```

    * С указанием схемы, где расположена таблица, и имени таблицы:

        ```sql
        SELECT * FROM yezzey_offload_relation_status('<имя_схемы>', '<имя_таблицы>');
        ```

    Будет выведена информация о каждом сегменте кластера {{ GP }}, в котором размещены данные таблицы. Изучите столбец `external_bytes`:
    * Если в столбце все значения нулевые, то таблица размещена в хранилище кластера.
    * Если в столбце есть ненулевые значения, то таблица размещена в холодном хранилище.

1. Выполните один из запросов, чтобы узнать, где размещены сегментные файлы таблицы:

    * С указанием имени таблицы:

        ```sql
        SELECT * FROM yezzey_offload_relation_status_per_filesegment('<имя_таблицы>');
        ```

    * С указанием схемы, где расположена таблица, и имени таблицы:

        ```sql
        SELECT * FROM yezzey_offload_relation_status_per_filesegment('<имя_схемы>', '<имя_таблицы>');
        ```

    Будет выведена информация о каждом сегментном файле таблицы. Изучите столбец `external_bytes`:
    * Если в столбце все значения нулевые, то файлы размещены в хранилище кластера.
    * Если в столбце есть ненулевые значения, то файлы размещены в холодном хранилище.

## Определить, как размещена таблица в холодном хранилище {#describe-relation-objstorage}

Если таблица [находится в холодном хранилище](#describe-relation), то ее сегментные файлы хранятся в служебном бакете.

Чтобы получить информацию о том, как сегментные файлы таблицы размещены в бакете:

1. [Подключитесь](../connect.md) к базе данных от имени владельца или пользователя, имеющего в базе данных разрешение `SELECT`.

1. Выполните один из запросов:

    * С указанием имени таблицы:

        ```sql
        SELECT * FROM yezzey_relation_describe_external_storage_structure('<имя_таблицы>');
        ```

    * С указанием схемы, где расположена таблица, и имени таблицы:

        ```sql
        SELECT * FROM yezzey_relation_describe_external_storage_structure('<имя_схемы>', '<имя_таблицы>');
        ```

    Будет выведена следующая информация о каждом сегментном файле таблицы:

    * `offload_reloid` — идентификатор объекта (object identifier, OID) для таблицы.
    * `segindex` — номер сегмента.
    * `segfileindex` — номер сегментного файла.
    * `external_storage_filepath` — путь к сегментному файлу в бакете {{ objstorage-name }}.

        Путь к сегментному файлу формируется автоматически. Он зависит от структуры таблицы и количества сегментов в кластере {{ GP }}.

    * `local_bytes` и `local_commited_bytes` — сколько байтов сегментного файла находится в хранилище кластера. Значения должны быть нулевыми.
    * `external_bytes` — размер сегментного файла.

## Выгрузить таблицу из хранилища кластера в холодное хранилище {#offload-relation}

Если таблица находится в хранилище кластера, то ее можно выгрузить в холодное хранилище. Чтобы понять, где она находится, [получите информацию о размещении таблицы](#describe-relation).

{% note info %}

Во время выгрузки изменение таблицы будет недоступно.

После завершения выгрузки и снятия эксклюзивной блокировки (exclusive lock) таблицу снова можно будет изменять.

{% endnote %}

Чтобы выгрузить таблицу:

1. [Подключитесь](../connect.md) к базе данных от имени владельца или пользователя, имеющего в базе данных разрешение `SELECT`.

1. Вызовите одну из функций:

    * С указанием имени таблицы:

        ```sql
        SELECT yezzey_define_offload_policy('<имя_таблицы>');
        ```

    * С указанием схемы, где расположена таблица, и имени таблицы:

        ```sql
        SELECT yezzey_define_offload_policy('<имя_схемы>', '<имя_таблицы>');
        ```

Длительность выгрузки зависит от размера таблицы и количества сегментных файлов. Во время выгрузки будут выводиться сообщения, по которым можно отслеживать состояние процесса:

* Сообщения следующего вида показывают размер выгруженных ранее сегментных файлов:

    ```bash
    NOTICE:  yezzey: relation virtual size calculated: 0  (segX sliceY ... pid=...)
    ```

    Значение `0` означает, что ранее выгруженных сегментных файлов нет (они выгружаются впервые).

* Сообщения следующего вида свидетельствуют об успешной выгрузке сегментных файлов:

    ```bash
    INFO:  yezzey: relation segment reached external storage (blkno=...), up to logical eof ...  (segX sliceY ... pid=...)
    ```

## Загрузить таблицу из холодного хранилища в хранилище кластера {#load-relation}

Если таблица находится в холодном хранилище, то ее можно загрузить в хранилище кластера. Чтобы понять, где она находится, [получите информацию о размещении таблицы](#describe-relation).

{% note info %}

Во время загрузки изменение таблицы будет недоступно.

После завершения загрузки и снятия эксклюзивной блокировки (exclusive lock) таблицу снова можно будет изменять.

{% endnote %}

Чтобы загрузить таблицу:

1. [Подключитесь](../connect.md) к базе данных от имени владельца или пользователя, имеющего в базе данных разрешение `SELECT`.

1. Вызовите одну из функций:

    * С указанием имени таблицы:

        ```sql
        SELECT yezzey_load_relation('<имя_таблицы>');
        ```

    * С указанием схемы, где расположена таблица, и имени таблицы:

        ```sql
        SELECT yezzey_load_relation('<имя_схемы>', '<имя_таблицы>');
        ```

Длительность загрузки зависит от размера таблицы и количества сегментных файлов. После завершения загрузки будет выведено сообщение следующего вида:

```sql
INFO:  loaded relation ... to local storage
```

## Пример использования {#usage-example}

Пример использования расширения для работы с гибридным хранилищем см. в разделе [{#T}](../../tutorials/yezzey.md).

{% include [greenplum-trademark](../../../_includes/mdb/mgp/trademark.md) %}
