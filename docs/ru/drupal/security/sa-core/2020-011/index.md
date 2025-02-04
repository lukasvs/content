---
title: 'SA-CORE-2020-011'
slug: wiki/security/sa-core-2020-011
metatags:
  title: 'Drupal: SA-CORE-2020-011'
  description: 'Умеренно критическое. Исправлено в версиях: 8.8.10, 8.9.6, 9.0.6.'
---

**Проект:** Drupal Core\
**Дата публикации:** 16 сентября 2020\
**Риск безопасности:** Умеренно критическое. 13/25 AC:Basic A:User CI:Some II:Some E:Theoretical TD:Default\
**Уязвимость:** Раскрытие информации

## Описание

Уязвимость найденная в File модуле позволяет злоумышленнику получить доступ к метаданным постоянного приватного файла, к которому у него нет доступа, путём перебора ID файлов.

## Решение

Обновиться до актуальных версий:

- Если используете Drupal 8.8.x, обновитесь до [Drupal 8.8.10](../../../releases/8/8.8.x/8.8.10/index.md).
- Если используете Drupal 8.9.x, обновитесь до [Drupal 8.9.6](../../../releases/8/8.9.x/8.9.6/index.md).
- Если используете Drupal 9.0.x, обновитесь до [Drupal 9.0.6](../../../releases/9/9.0.x/9.0.6/index.md).

## Ссылки

- [SA-CORE-2020-011](https://www.drupal.org/sa-core-2020-011) (англ.), drupal.org, 16 сентября 2020