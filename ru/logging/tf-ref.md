---
title: Справочник {{ TF }} для {{ cloud-logging-full-name }}
description: На этой странице приведен справочник ресурсов и источников данных провайдера {{ TF }}, которые поддерживаются для сервиса {{ cloud-logging-name }}.
---

# Справочник {{ TF }} для {{ cloud-logging-full-name }}

{% include [terraform-ref-intro](../_includes/terraform-ref-intro.md) %}

## Ресурсы {#resources}

Для {{ cloud-logging-name }} поддерживаются следующие ресурсы провайдера {{ TF }}:

| **Ресурс {{ TF }}** | **Ресурс {{ yandex-cloud }}** |
| --- | --- |
| [yandex_logging_group]({{ tf-provider-resources-link }}/logging_group) | [Лог-группа](./concepts/log-group.md) |

## Источники данных {#data-sources}

Для {{ cloud-logging-name }} поддерживаются следующие источники данных провайдера {{ TF }}:

| **Источник данных {{ TF }}** | **Описание** |
| --- | --- |
| [yandex_logging_group]({{ tf-provider-datasources-link }}/logging_group) | Информация о [лог-группе](./concepts/log-group.md) |