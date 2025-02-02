---
title: Аутентификация пользователей {{ mgp-full-name }}
description: Из статьи вы узнаете про аутентификацию пользователей в {{ mgp-name }} и особенности настройки правил аутентификации.
---

# Аутентификация пользователей

Аутентификация пользователей в {{ mgp-name }} настраивается с помощью [правил](../operations/user-auth-rules.md) в разделе **{{ ui-key.yacloud.greenplum.cluster.user-auth.title_page-auth-user }}**. Этот раздел представляет собой интерфейс для управления конфигурационным файлом [pg_hba.conf]({{ pg-docs }}/auth-pg-hba-conf.html) с некоторыми ограничениями:

* доступны не все типы соединения и методы аутентификации;
* запрещены системные базы данных и пользователи;
* недоступны специальные значения и регулярные выражения для баз данных и пользователей.

Подробнее об этих ограничениях см. в разделе [Настройки правил аутентификации](#auth-settings).

Каждое правило аутентификации определяет тип соединения, имя базы данных, имя пользователя или группы пользователей, [FQDN](../../glossary/fqdn.md) хоста или диапазон IP-адресов, с которых будет выполняться соединение, и способ аутентификации. Правила читаются сверху вниз, первое подходящее правило применяется для аутентификации. Если по первому подходящему правилу аутентификация не прошла, последующие не рассматриваются.

Если правила аутентификации не заданы, используется правило по умолчанию, которое разрешает аутентификацию всем пользователям во всех базах и со всех хостов с помощью метода `md5` (по паролю). Если правила аутентификации заданы, правило по умолчанию читается последним.

## Настройки правил аутентификации {#auth-settings}

Вы можете задать следующие настройки аутентификации при [добавлении](../operations/user-auth-rules.md#add-rules) или [изменении](../operations/user-auth-rules.md#edit-rules) правил:

Доступные типы соединений:

* `{{ ui-key.yacloud.greenplum.cluster.user-auth.value_hba-type-host }}` — TCP/IP с применением шифрования SSL и без него.
* `{{ ui-key.yacloud.greenplum.cluster.user-auth.value_hba-type-hostssl }}` — TCP/IP с применением шифрования SSL.
* `{{ ui-key.yacloud.greenplum.cluster.user-auth.value_hba-type-hostnossl }}` — TCP/IP без шифрования SSL.

Для баз данных и пользователей недоступны:

* Системные базы данных, например `postgres`.
* Системные пользователи, например [mdb_admin](cluster-users.md#mdb_admin).
* Специальные значения, например `all` или `sameuser`.
* [Регулярные выражения]({{ pg-docs }}/functions-matching.html#POSIX-SYNTAX-DETAILS).

Имя группы пользователей базы данных должно начинаться с символа `+`, например `+dbwriters`.

В качестве адреса вы можете использовать FQDN хоста, диапазон IP-адресов или специальное значение `all`, которое разрешает подключение с любого хоста:

* `{{ host-name }}.{{ dns-zone }}`
* `172.20.143.89/32`
* `::0/0`
* `all`

Поддерживаются следующие методы аутентификации:

* `md5` — по паролю. Подробнее см. в [документации {{ PG }}]({{ pg-docs }}/auth-password.html).
* `reject` — запрещает подключение пользователя.

Подробнее о настройках см. в [документации {{ PG }}]({{ pg-docs }}/auth-pg-hba-conf.html).

{% include [greenplum-trademark](../../_includes/mdb/mgp/trademark.md) %}
