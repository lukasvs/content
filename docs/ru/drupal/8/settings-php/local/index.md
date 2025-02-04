---
title: settings.local.php — локальные настройки окружения сайта
slug: wiki/8/settings-local-php
core: 8
metatags:
  title: 'Drupal 8: settings.local.php — локальные настройки окружения сайта'
  description: 'Для чего и как используется файл settings.local.php.'
category:
  area: Настройки окружения
  title: settings.local.php
  order: 2
---

**settings.local.php** — файл с настройками локального окружения сайта. Является дополнением [settings.php](../index.md) и использует все доступные его настройки.

Данный файл заменят, дополняет или выключает определённые настройки и поведение сайта для локальной системы. Используется при разработке проекта.

## Создание файла

По умолчанию данный файл отсутствует. Для него поставляется стандартный пример `example.settings.local.php` который можно найти в директории `/sites`.

Для создания собственного файла достаточно скопировать `example.settings.local.php` в папку сайта, где уже находится основной `settings.php` и переименовать его в `settings.local.php`.

## Подключение

Данный файл подключается из основного файлы `settings.php`. В конце этого файла вы можете найти следующие строки:

```php
# if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
#   include $app_root . '/' . $site_path . '/settings.local.php';
# }
```

Вам достаточно раскомментировать их и настройки из `settings.local.php` начнут загружаться и использоваться. Вы можете менять данное условие и поведение так как вам угодно. Данный файл должен подключаться в самом конце `settings.php` для того чтобы он мог изменять все основные настройки.

> [!WARNING]
> Убедитесь что данный файл не подключается на продакшене. Это приведёт к серьезному снижению производительности.

## Применение

Файл `settings.local.php` применяется на локальных окружения в процессе разработки и заменяет некоторые настройки системы на специальные. Например, отключается кеширование, сжатие и объединение JavaScript и CSS файлов и прочие изменения необходимые для разработки, но не нужны на рабочем проекте.

Эти изменения производятся в `settings.local.php` для того чтобы эти настройки были явно отражены и было удобно переносить проект, делиться изменениями с командой и т.д.

## Настройки по умолчанию

В данном разделе рассмотрены все настройки по умолчанию, который указаны в `example.settings.local.php` файле и подойдут для большинства проектов и задач.

### Assertions

Drupal использует сравнения ([assert](http://php.net/assert)), для того чтобы сообщать об ошибках использования API. Когда обнаружена проблема, запрос будет провален и выдана ошибка.

Данная настройка включает сравнения, которые по умолчанию отключены, чтобы в процессе разработки замечать ошибки на ранних стадиях.

> [!NOTE]
> Если вы используете PHP 7.0 или выше, то рекомендуется установить настройку `zend.assertions=1` в `php.ini` файле (данную настройку нельзя задавать через `.htaccess` или `ini_set()`). На продакшен сайте убедитесь что данная опция отключена.

**Пример:**

```php
assert_options(ASSERT_ACTIVE, TRUE);
\Drupal\Component\Assertion\Handle::register();
``` 

### Сервисы для разработки

В данном файле подключается файл `development.services.yml` с сервисами и параметрами для разработки, которые будут загружены в контейнер.

**Пример:**

```php
$settings['container_yamls'][] = DRUPAL_ROOT . '/sites/development.services.yml'
```

### Подробное логирование

Для конфигурации `system.logging` задаётся степень детализации ошибок на подробные (`verbose`). Это значит, что все ошибки будут выводиться, включая стэк вызова.

```php
$config['system.logging']['error_level'] = 'verbose';
```

### Агрегация JS и CSS

В конфигурации `system.performance` отключается обработка JavaScript и CSS файлов. Это значит что о ни не будут объединяться, сжиматься и кешироваться. Так вы сможете видеть изменения на сайте сразу после их внесения без необходимости сброса кеша.

```php
$config['system.performance']['css']['preprocess'] = FALSE;
$config['system.performance']['js']['preprocess'] = FALSE;
```

### Кеш рендера

Кеш рендера отвечает за хранение результатов рендера рендер массивов. В процессе активной разработки он может доставлять серьезные неудобства, поэтому его рекомендуется отключать.

При помощи данной настройки вы можете отключить кеширование результатов перенаправляя весь кеш в Null хранилище, который ничего не хранит.

> [!NOTE]
> Вы должны проверять работоспособность сайта и его функционала с включенным кешем рендера, для того чтобы убедиться, что кеш собирается корректно.

**Пример:**

```php
$settings['cache']['bins']['render'] = 'cache.backend.null';
```

### Кеш миграций

Кеш миграций хранит в себе информацию о всех объявленных миграциях. Если вы активно пишите миграции, вы можете переключить хранение миграций в память, чтобы они обнаруживались сразу как только вы их объявляете.

> [!NOTE]
> Это поведение влияет только на официальный способ создания миграций (те что создаются в `/MODULENAME/migrations`). Если вы используете Migrate Plus который проксирует миграции, данная настройка не окажет никакого влияния.

**Пример:**

```php
$settings['cache']['bins']['discovery_migration'] = 'cache.backend.memory';
```

### Кеш страниц

Кеш страниц, который предоставляется модулем Internal Page Cache, хранит в себе готовые результаты страниц. В процессе активной разработки он может доставлять серьезные неудобства, поэтому его рекомендуется отключать.

> [!NOTE]
> Вы должны проверять работоспособность сайта и его функционала с включенным Internal Page Cache, для того чтобы убедиться, что кеш собирается корректно.

```php
$settings['cache']['bins']['page'] = 'cache.backend.null';
```

### Динамический кеш страниц

Динамический кеш страниц, который предоставляется модулем Dynamic Page Cache, хранит в себе готовые результаты страниц, в зависимости от роли пользователя и прочих контекстов.

> [!NOTE]
> Вы должны проверять работоспособность сайта и его функционала с включенным Dynamic Page Cache, для того чтобы убедиться, что кеш собирается корректно.

```php
$settings['cache']['bins']['dynamic_page_cache'] = 'cache.backend.null';
```

### Включить обнаружение тестовых расширений

По умолчанию доступны только расширения ([модули](../../modules/index.md), [темы оформления](../../themes/index.md)) которые не помечены как тестовые. При помощи данной настройки, вы можете включить их обнаружение и отображение в административном интерфейсе.

**Пример:**

```php
$settings['extension_discovery_scan_tests'] = TRUE;
```

### Доступ к ребилду

Drupal очень серьезно кеширует всю информацию что собирает. Даже с отключением различных кешей выше, он будет собираться. Например, информация о хуках, темплейтах, расширениях и т.д. кешируется даже в процессе разработки, так как оказывает серьезное влияние на производительность, а требуется нечасто.

Для того чтобы эти кеши были очищены нужно запустить ребилд системы. Для этого с Drupal поставляется файл `rebuild.php`, который не будет делать ничего, пока данная настройка отключена.

В процессе активной разработки вам потребуется регулярно запускать ребилд системы, поэтому данное поведение активируется. Для того чтобы запустить ребилд вам будет достаточно напрямую обратиться к файлу `https://example.com/rebuild.php`.

> [!TIP]
> Вы можете использовать команду [Drush](../../../../drush/index.md) `drush cache:rebuild` для полного ребилда кеша. Она будет работать даже с отключенной опцией.

**Пример:**

```php
$settings['rebuild_access'] = TRUE;
```

### Отключение принудительного изменения прав на запись для директории сайта

По умолчанию Drupal регулярно проверяет важные директории и сбрасывает им права на запись. Это может вызвать неудобства в процессе разработки, поэтому данные проверки рекомендуется отключать.

```php
$settings['skip_permissions_hardening'] = TRUE;
```

## Смотрите также

- [settings.php](../index.md) — основные настройки окружения.