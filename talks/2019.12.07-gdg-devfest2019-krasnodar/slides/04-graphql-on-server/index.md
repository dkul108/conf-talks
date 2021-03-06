# Что такое GraphQL <br/> на сервере?

-----

## Это минимум 2 пакета

![Диаграмма экосистемы](./diagram-ecosystem.svg) <!-- .element: style="width: 90vw;" class="plain"  -->

-----

## Фаза запуска приложения

![Диаграмма экосистемы](./diagram-ecosystem-schema.svg) <!-- .element: style="width: 90vw;" class="plain"  -->

-----

# GraphQL Schema — это

### описание ваших типов данных на сервере, <!-- .element: class="fragment" -->

### связей между ними <!-- .element: class="fragment" -->

### и логики получения этих самых данных. <!-- .element: class="fragment" -->

-----

##### Hello world schema (startup phase)

```js
import { GraphQLSchema, GraphQLObjectType, GraphQLString } from 'graphql';

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
      hello: {
        type: GraphQLString,
        resolve: () => 'world',
      }
    }
  }),
  // mutation: { ... },
  // subscription: { ... },
});

```

`build phase` – это когда вы описываете структуру своих данных и методы получения.

-----

## Runtime фаза

![Диаграмма экосистемы](./diagram-ecosystem-server.svg) <!-- .element: style="width: 90vw;" class="plain"  -->

-----

##### Hello world schema (runtime phase)

```js
import { graphql } from 'graphql';
import { schema } from './your-schema';

const query = '{ hello }';
const result = await graphql(schema, query);

// returns: { data: { hello: "world" } }

```

<span>`runtime phase` – это когда выполняется клиентский запрос через метод `graphql()`, который:</span> <!-- .element: class="fragment" -->

- производит парсинг GraphQL-запроса <!-- .element: class="fragment" -->
- производит валидацию запроса на соответствие GraphQL-схеме <!-- .element: class="fragment" -->
- выполняет запрос, пробегаясь по дереву схемы <!-- .element: class="fragment" -->
- валидирует возвращаемый ответ <!-- .element: class="fragment" -->

-----

![Диаграмма работы сервера](./diagram-server.svg) <!-- .element: style="width: 90vw;" class="plain"  -->
<!-- https://drive.google.com/file/d/1G-Iu_fZdrois9NZY1-5YGWNwELJEzy6Y/view?usp=sharing -->

-----

### Каковы обычно требования к серверу?

<br />
<br />

### По некому протоколу обслуживать множество запросов от разных клиентов. <!-- .element: class="fragment" -->

<br />
<br />

<span>Это может быть `http(s)` или `websockets`, либо вообще что-то экзотическое типа `ssh`, `telnet`.</span> <!-- .element: class="fragment" -->

-----

### Что должен делать HTTP-сервер

- открыть порт и слушать http-запросы от клиентов
- вытаскивать из них GraphQL-запрос с переменными
- формировать `context` с данными текущего пользователя и глобальными ресурсами (в рамках запроса)
- откуда-то получить "сваренную" GraphQL-схему
- отправить на выполнение GraphQL-схему, запрос и `context` в пакет `graphql`
- из полученных данных от `graphql` сформировать http-ответ и отдать клиенту

-----

### По-желанию попутно сделать всякие операции, типа:

- парсинга кук
- проверки токенов
- базовая авторизация
- превращение persistent query по id в GraphQL-запрос
- логирования запросов
- кеша запросов
- отлова ошибок и отправки их в tracker.

-----

### Популярные сервера:

- [express-graphql](https://github.com/graphql/express-graphql)
- [koa-graphql](https://github.com/chentsulin/koa-graphql)
- [apollo-server@1.x.x](https://github.com/apollographql/apollo-server/tree/version-1)
- [apollo-server@2.x.x](https://github.com/apollographql/apollo-server/tree/version-2)
- [graphql-yoga](https://github.com/prisma/graphql-yoga)

<br />
<br />

TL;DR: берите `apollo-server@2.x.x`

Подробнее о серверах, [читать тут](https://github.com/nodkz/conf-talks/tree/master/articles/graphql/server)

-----

![Диаграмма экосистемы](./diagram-ecosystem.svg) <!-- .element: style="width: 90vw;" class="plain"  -->

-----

### Запускаем сервер на NodeJS

```js
import { GraphQLSchema, GraphQLObjectType, GraphQLString } from 'graphql';
import { ApolloServer } from 'apollo-server';

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      hello: {
        type: GraphQLString,
        args: {
          name: { type: GraphQLString, defaultValue: 'world' },
        },
        resolve: (source, args, context) => {
          return `Hello, ${args.name} from ip ${context.req.ip}`;
        },
      },
    },
  }),
});

const server = new ApolloServer({
  schema,
  context: ({ req }) => ({ req }),
  playground: true,
});

server
  .listen({
    port: 5000,
    endpoint: '/',
    playground: '/playground',
  })
  .then(({ url }) => {
    console.log(`🚀  Server ready at ${url}`);
  });

```

<span class="fragment" data-code-focus="1-2" />
<span class="fragment" data-code-focus="4-18" />
<span class="fragment" data-code-focus="21-25" />
<span class="fragment" data-code-focus="27-35" />

-----

### Результат в браузере

![Launch server result](./launch-server.png)

-----

### Итак, грубо говоря <!-- .element: class="red" -->

GraphQL требует от бекендеров описать набор функций для получения данных. <!-- .element: class="fragment" -->

И потом, исходя из полученного GraphQL-запроса, сам вызывает только необходимые функции. <!-- .element: class="fragment" -->

-----

### Никакой магии, <!-- .element: class="orange" -->

## GraphQL – это «строгий» <br/>вызывальщик функций

-----

![not my problems](./graphql-rest-rpc.svg) <!-- .element: style="width: 600px;" class="plain"  -->

Авторизация, оптимизация, кэширование <br />– это за пределами GraphQL
