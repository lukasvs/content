---
title: События
slug: wiki/8/events
core: 8
metatags:
  title: 'Drupal 8: События'
  description: 'События в Drupal 8 позволяют подписываться на определенные этапы работы системы.'
category:
  area: События
  title: Введение
  order: 1
---

**События** (англ. Events) — способ внедрения в обработку запроса на различных этапах работы Drupal.

Каждое событие имеет уникальное имя в виде строки, а также объекта, который содержит всю необходимую информацию для данного события.

События предоставляются как Symfony, ядром Drupal, так и сторонними модулями.

Их можно разделить на 3 части:

- **Event** — объект события, содержит всю важную информацию для своего события.
- **Event Dispatcher** — [сервис](../services/index.md), позволяющий производить вызов события.
- **Event Subscriber** — сервис, подписывающийся на различные события и обрабатывающий их.

## Ссылки

- [Drupal 8: Events — создание и использование событий](https://niklan.net/blog/170), Niklan, 2018