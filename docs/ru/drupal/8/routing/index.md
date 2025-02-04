---
title: Маршрутизация
slug: wiki/8/routing
core: 8
search-keywords:
  - роуты контроллеры маршруты роутинг
  - как добавить создать собственную свою кастомную страницу
category:
  area: Маршрутизация
  order: 1
  title: Введение
metatags:
  title: 'Drupal 8: Маршрутизация'
  description: 'Как создавать свои собственные страницы, с уникальным адресом, требованиями и поведением.'
---

Маршрутизация в Drupal 8 базируется на системе маршрутизации Symfony. Они имеют идентичный синтаксис объявления маршрутов, с некоторыми дополнительными возможностями специально для Drupal.

Маршрут или роут — путь, который возвращает какое-то определенное содержимое или результат. Когда вы обращаетесь к страницам, Drupal пытается найти подходящий маршрут для вашего запроса, затем выполнить его, а его результат вернуть клиенту, если он не смог подобрать подходящий маршрут, то будет возвращён HTTP 404.

Примерная схема работы маршрутов и контроллеров:

![Drupal 8 routing](https://www.drupal.org/files/Drupal8Routing.png)

## Ссылки

- [Routing](https://symfony.com/doc/current/routing.html) (англ.), Symfony
- [Routing system](https://www.drupal.org/docs/8/api/routing-system) (англ.), drupal.org
