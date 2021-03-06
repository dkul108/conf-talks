# Дизайн GraphQL-схем — строим схемы правильно (версия 2)

GraphQL-схема может обернуться головой болью и кучей дополнительного кода для разработчиков. Поэтому, чем удобнее схема, тем быстрее, легче и с меньшим количеством ошибок будут разработаны ваши клиентские приложения.

Рекомендации и правила, озвученные в этой статье, были выработаны за 3 года использования GraphQL как на стороне сервера (при построении схем), так и на клиентской стороне (написания GraphQL-запросов и покрытием клиентского кода статическим анализом).

Версия 2, доработанная с учетом правок от комьюнити и ревью от GraphQL-гуру Михаила Новикова и Ивана Гончарова.

-------

1. Правила именования
1.1. Используйте camelCase для именования GraphQL-полей и аргументов.
1.2. Используйте UpperCamelCase для именования GraphQL-типов.
1.3. Используйте CAPITALIZED_WITH_UNDERSCORES для именования значений ENUM-типов.
2. Правила типов
2.1. Используйте кастомные скалярные типы, если вы хотите объявить поля или аргументы с определенным семантическим значением.
2.2. Используйте Enum для полей, которые содержат определенный набор значений.
3. Правила полей (Output)
3.1. Давайте полям понятные смысловые имена, а не то как они реализованы.
3.2. Делайте поля обязательными NonNull, если данные в поле возвращаются при любой ситуации.
3.3. Достигайте максимального описания обязательных полей.
4. Правила аргументов (Input)
4.1. Группируйте взаимосвязанные аргументы вместе в новый input-тип.
4.2. Используйте строгие скалярные типы для аргументов, например DateTime вместо String.
4.3. Помечайте аргументы как NonNull, если они обязательны для выполнения запроса.
5. Правила списков
5.1. Для фильтрации списков используйте аргумент filter, который содержит в себе все доступные фильтры.
5.2. Для сортировки списков используйте аргумент sort, который должен быть Enum или [Enum!].
5.3. Для ограничения возвращаемых элементов в списке используйте аргументы limit со значением по умолчанию и skip.
5.4. Для пагинации используйте аргументы page, perPage и возвращайте output-тип с полями items с массивом элементов и pageInfo с метаданными для удобной отрисовки страниц на клиенте.
5.5. Для бесконечных списков (infinite scroll) используйте Relay Cursor Connections Specification.
6. Правила Мутаций
6.1. Используйте Namespace-типы для группировки мутаций в рамках одного ресурса!
6.2. Выходите за рамки CRUD – cоздавайте небольшие мутации для разных логических операций над ресурсами.
6.3. Рассмотрите возможность выполнения мутаций сразу над несколькими элементами (однотипные batch-изменения).
6.4. У мутаций должны быть четко описаны все обязательные аргументы, не должно быть вариантов либо-либо.
6.5. У мутации вкладывайте все переменные в один уникальный input аргумент.
6.6. Мутация должна возвращать свой уникальный Payload-тип.
6.7. В ответе мутации возвращайте поле с типом Query.
6.8. Мутации должны возвращать поле errors с типизированными пользовательскими ошибками.
