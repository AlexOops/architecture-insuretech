# Task5 — Проектирование GraphQL API

## Цель

Перевести REST API сервиса `client-info` на GraphQL, чтобы потребители могли получать только нужные поля клиентской карточки и не выполнять несколько отдельных REST-запросов для одного бизнес-сценария.

## Проблема REST API

Текущий REST API предоставляет отдельные ресурсы:

- `GET /clients/{id}` — базовая информация о клиенте;
- `GET /clients/{id}/documents` — документы клиента;
- `GET /clients/{id}/relatives` — родственники клиента.

Такой подход приводит к двум проблемам:

1. Если сценарию нужны разные части карточки клиента, приходится выполнять несколько HTTP-запросов.
2. Если сделать один большой REST endpoint для всей карточки клиента, он будет возвращать слишком много данных, так как полный атрибутивный состав карточки достигает 500 полей.

## Решение

Для `client-info` спроектирована GraphQL-схема:

- `Task5/schema.graphql`

Дополнительно добавлены примеры запросов:

- `Task5/examples.graphql`

## Основные типы

- `Client`
- `Document`
- `Relative`

## Основные queries

```graphql
client(id: ID!): Client
clientDocuments(clientId: ID!): [Document!]!
clientRelatives(clientId: ID!): [Relative!]!
```

## Как GraphQL решает проблему

GraphQL позволяет потребителю явно указать, какие поля нужны в конкретном сценарии.

Например, если веб-приложению нужны только имя клиента и номера документов, оно может запросить только эти поля:

```graphql
query GetClientWithDocuments {
  client(id: "client-123") {
    id
    name
    documents {
      type
      number
    }
  }
}
```

Это уменьшает:

- количество HTTP-запросов;
- объём передаваемых данных;
- нагрузку на `client-info`;
- связность между потребителями и внутренней структурой REST API.

## Покрытие REST API

| REST endpoint | GraphQL query |
|---|---|
| `GET /clients/{id}` | `client(id: ID!)` |
| `GET /clients/{id}/documents` | `client(id) { documents { ... } }` или `clientDocuments(clientId)` |
| `GET /clients/{id}/relatives` | `client(id) { relatives { ... } }` или `clientRelatives(clientId)` |

## Рекомендации по реализации

При реализации GraphQL resolver'ов нужно учитывать возможную проблему N+1 запросов.

Рекомендуется использовать:

- DataLoader;
- batch-загрузку документов и родственников;
- лимиты глубины запроса;
- лимиты сложности запроса;
- авторизацию на уровне resolver'ов;
- логирование slow queries.
