---
title: 'Drupal 9.4.0'
slug: wiki/drupal/releases/9.4.0
core: 9
metatags:
  title: 'Drupal 9.4.0: Список изменений'
  description: 'Список изменений Drupal 9.4.0.'
---

> [!WARNING]
> Данная версия находится в разработке. Список изменений может меняться и дополняться, даты могут смещаться.

**Дата релиза**: июнь 2022\
**Активная поддержка до**: декабрь 2022\
**Поддержка безопасности до**: июнь 2023

## Хук `THEME_ENGINE_init()` объявлен устаревшим

* [#3158289](https://www.drupal.org/node/3158289) 

Хук `THEME_ENGINE_init()` объявлен устаревшим и будет удалён в [Drupal 10](../../../../10/index.md). Замена данному хуку не предоставляется.

## Для сущностей с поддержкой ревизий, модуль Views теперь предоставляет связи между основной и ревизионной таблицами

* [#2652652](https://www.drupal.org/node/2652652)

Ранее, модуль Views не предоставлял для ревизионных сущностей связи между основной таблицой и таблицой ревизий. Для модулей `block_content` и `node` были «костыли» для решения данной проблемы, но остальным сущностям приходилось решать это вопрос самостоятельно.

Начиная с текущего релиза, Views предоставляет соответствующие связи между основной и ревизионной таблицами для всех типов сущностей что поддерживают ревизии.

## В .htaccess файлы добавлена поддержка настроек PHP 8

* [#3186524](https://www.drupal.org/node/3186524)

В `.htaccess` файлы добавлен новый раздел для настроек параметров PHP 8. Это изменение касается как основного файла `.htaccess`, так и всех прочих что генерируются Drupal для различных ситуаций, например, для `sites/default/files`.

В эти файлы добавлена настройка для PHP 8:

```text
# Имеющийся раздел для PHP 7.
<IfModule mod_php7.c>
  php_value assert.active   0
</IfModule>
# Новые настройки для PHP 8.
<IfModule mod_php.c>
  php_value assert.active   0
</IfModule>
```

Если вы переопределяете данные файлы или изменяете параметры PHP, для PHP 8 вам необходимо скопировать данные настройки и поправить под себя. Если вы используете стандартные настройки, файлы обновятся автоматически.

## Для оптимизации определения сериализаторов и нормализаторов, `NormalizerBase` теперь реализует `CacheableSupportsMethodInterface`

* [#3252872](https://www.drupal.org/node/3252872)

`Drupal\serialization\Normalizer\NormalizerBase` теперь реализует `CacheableSupportsMethodInterface` и возвращает `FALSE` по умолчанию. Это означает, что классы, расширяющие абстрактные классы из ядра продолжат работать как и раньше. Все конкретные реализации теперь имеют метод `::hasCacheableSupportsMethod()`, который возвращает `TRUE`.

Данный интерфейс был представлен в [Symfony 4.1](https://symfony.com/blog/new-in-symfony-4-1-faster-serializer):

> Приложения могут определять множество нормализаторов и денормализаторов, Symfony вызывает `::supportsNormalization()` для каждого из них, каждый раз производя нормализацию или денормализацию объекта. В теории, результат вызова `::supportsNormalization()` зависит от моженства факторов. На практике большинство из них зависит лишь от типа и формата, а данная информация легко кешируется.

Утверждение выше также подходит и для Drupal. Мы можем кешировать результат основываясь на переданном типе и формате, что позволяет сократить количество вызовов. Подобное изменение позволяет добиться увеличения производительности от 5 до 60 процентов!

## Обновлены старые схемы для `uid` поля сущностей, где может использоваться устаревший метод получения ID пользователя

* [#3153455](https://www.drupal.org/node/3153455) Добавлено обновление, которое исправляет

Сайты, установленные на версии ранее чем Drupal 8.6.0, а теперь использующие Drupal 9, могут столкнуться с проблемой, что у некоторых сущностей из ядра поле `uid` использует устаревшее значение для `default_value_callback`. Ранее, использовался метод `{EntityTypeClass}::getCurrentUserId()`, который был объявлен устаревшим в Drupal 8.6.0 и удалён в Drupal 9.

На таких проектах, подобная ситуация может приводить, как к ошибке: «The website encountered an unexpected error. Please try again later. Error: Call to a member function getAccountName() on null», так и предупреждению «call_user_func() expects parameter 1 to be a valid callback, class 'Drupal\node\Entity\Node' does not have a method 'getCurrentUserId' in Drupal\Core\Field\FieldConfigBase->getDefaultValue()».

В данном релизе добавлены обновления для всех сущностей ядра, которые проверяют, что используется современный метод, и если обнаруживается старый, он обновляется на новый.

## Драйвера баз данных, предоставляемые ядром Drupal, перенесены в соответствующие одноимённые модули

* [#3129043](https://www.drupal.org/node/3129043) 

[Ранее](../../../8/8.9.x/8.9.0/index.md) была добавлена поддержка драйверов баз данных, как в пространстве имён модуля, так и в «src» директории ядра. Теперь эта возможность задействована для переноса драйверов баз данных предоставляемых ядром в соответствующие модули. Для каждого драйвера базы данных добавлен новый одноимённый модуль: `mysql`, `pgsql` и `sqlite`.

Для драйверов баз данных в пространстве имён модуля требуется добавлять query-параметр в URL-строку подключения. Например: `pgsql://test_user:test_pass@test_host:5432/test_database?module=pgsql`. Если вы используете один из модулей ядра, добавлять данный query-параметр не обязательно, все ранее сформированные URL-адреса подключений продолжат работать.

Использование `use` с пространством имён содержащим `Drupal\Core\Database\Driver` объявлено устаревшим и будет удалено _до релиза_ Drupal 11. Рекомендуется заменить их использование на новые пространства имён:

* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Install\Tasks` → `Drupal\Core\Database\Install\Tasks`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Connection` → `Drupal\Core\Database\Connection`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Delete` → `Drupal\Core\Database\Query\Delete`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\ExceptionHandler` → `Drupal\Core\Database\ExceptionHandler`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Insert` → `Drupal\Core\Database\Query\Insert`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Schema` → `Drupal\Core\Database\Schema`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Select` → `Drupal\Core\Database\Query\Select`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Truncate` → `Drupal\Core\Database\Query\Truncate`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Update` → `Drupal\Core\Database\Query\Update`
* `use Drupal\Core\Database\Driver\{mysql|pgsql|sqlite}\Upsert` → `Drupal\Core\Database\Query\Upsert`

## JSON:API теперь корректно обрабатывает запросы в режиме обслуживания

* [#3049048](https://www.drupal.org/node/3049048)

В JSON:API внесены множественные улучшения для корректной обработки и поведения запросов в режиме обслуживания.

> [!NOTE]
> До данного изменения, при попытке обратиться к ресурсам JSON:API при включенном режиме обслуживания, в качестве ответа отдавалась HTML страница обслуживания.

### JSON:API теперь отвечает с заголовком `Retry-After` в режиме обслуживания

Начиная с данного релиза, если сайт находится в режиме обслуживания, JSON:API будет отвечать с заголовком `Retry-After`.

Пример ответа в режиме обслуживания:

```json
{
   "jsonapi":{
      "version":"1.0",
      "meta":{
         "links":{
            "self":{
               "href":"http://jsonapi.org/format/1.0/"
            }
         }
      }
   },
   "errors":[
      {
         "title":"Service Unavailable",
         "status":"503",
         "detail":"core workspace is currently under maintenance. We should be back shortly. Thank you for your patience.",
         "links":{
            "via":{
               "href":"http://example.com/jsonapi/base_field_override/base_field_override"
            },
            "info":{
               "href":"http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.5.4"
            }
         }
      }
   ]
}
```

Данный ответ также содержит заголовок `Retry-After` с информацией о том, сколько следует подождать до следующей попытки. По умолчанию рекомендуется время от 5 до 10 секунд. Значение из данного диапазона выбирается случайным образом на каждый запрос для предотвращения эффекта [громового стада](https://en.wikipedia.org/wiki/Thundering_herd_problem).

В связи с данным изменением было добавлено две новые настройки:

* `jsonapi.settings.maintenance_header_retry_seconds.min`: Минимальное значение в секундах для заголовка `Retry-After`.
* `jsonapi.settings.maintenance_header_retry_seconds.max`: Максимальное значение в секундах для ззаголовка `Retry-After`.

Это позволит владельцу приложения иметь контроль над тем, как долго клиенты должны ждать перед повторным обращением. Данные настройки рассчитаны на продвинутых пользователей, в связи с этим, они не доступны в административном интерфейсе.

### Добавлено новое событие `Drupal\Core\Site\MaintenanceModeEvents::MAINTENANCE_MODE_REQUEST`

В ядро было добавлено новое событие `Drupal\Core\Site\MaintenanceModeEvents::MAINTENANCE_MODE_REQUEST` которое позволяет изменить как запрос, так и ответ, который производится в режиме обслуживания.

### Добавлен новый метод `MaintenanceModeInterface::getSiteMaintenanceMessage()`

В интерфейс `MaintenanceModeInterface` добавлен новый метод `::getSiteMaintenanceMessage()` который позволяет получить сообщение, которое должно выводиться в режиме обслуживания сайта.

## Функция `drupal_js_defaults()` объявлена устаревшей

* [#3197553](https://www.drupal.org/node/3197553), [#3256518](https://www.drupal.org/node/3256518) 

В Drupal 7 был добавлен [хук](../../../../9/hooks/index.md) `hook_js_alter()`, для того чтобы для него всегда были данные, была введена функция `drupal_js_defaults()`. В Drupal 8 произошёл переход на [систему библиотек](../../../../9/libraries/index.md), что сделало `hook_js_alter()` очень специфичным хуком для исключительных ситуаций. В связи с этим, функция `drupal_js_defaults()` стала бесполезной и объявлена устаревшей, будет удалена в Drupal 10 без предоставления замены.

## Метод `LanguageManagerInterface::getLanguageSwitchLinks()` теперь возвращает объект или `NULL`

* [#2940121](https://www.drupal.org/node/2940121)

В предыдущих версиях Drupal, `Drupal\Core\Language\LanguageManagerInterface::getLanguageSwitchLinks()` имел документацию, что он возвращает массив ссылок, но `
Drupal\language\ConfigurableLanguageManager::getLanguageSwitchLinks()` возвращал объект, содержащий массив ссылок, или же `FALSE`, если ссылок нет. 

Документация для `Drupal\Core\Language\LanguageManagerInterface::getLanguageSwitchLinks()` была изменена на то, что метод теперь возвращает либо объект с массивом ссылок, либо `NULL`, если ссылок нет.

Если вы используете или реализуете данный метод, вам необходимо обновить свой код.

## На замену опции запроса `return` добавлен новый метод `Connection::lastInsertId()`

* [#3185269](https://www.drupal.org/node/3185269) 

Опция запроса `return` и константы `Database::RETURN_*` объявлены устаревшими. В Drupal 11 они будут удалены и `Connection::query()` будет всегда возвращать объект `StatementInterface`.

Для манипуляций данными в БД (`INSERT`, `DELETE`, `UPDATE`, `UPSET`, `MERGE`, `TRUNCATE`) не принято использовать `Connection::query()`, для этого Drupal предоставляет построители динамических запросов.

В исключительных ситуациях `Connection::query()` использовался для операции вставки (`INSERT`) с целью получения ID последнего добавленного значения в БД. Для подобных случаев, начиная с данной версии, добавлен новый метод `Connection::lastInsertId()`, который возвращает ID последнего добавленного значения.

Для операций обновления и прочих, больше не рекомендуется использовать `Database::RETURN_AFFECTED`, вместо этого рекомендуется включить подсчёт значений путём передачи соответствующего аргумента в конструктор.

Пример, как данное изменения повлияло на `Connection::nextId()` для MySQL драйвера:

**Было:**

```php
$new_id = $this->query('INSERT INTO {sequences} () VALUES ()', [], ['return' => Database::RETURN_INSERT_ID]);
```

**Стало:**

```php
$this->query('INSERT INTO {sequences} () VALUES ()');
$new_id = $this->lastInsertId();
```

## Метод `\Drupal\Core\Validation\DrupalTranslator:transChoice()` объявлен устаревшим

* [#3255250](https://www.drupal.org/node/3255250)

Symfony 4.2 объявил `\Symfony\Component\Translation\TranslatorInterface::transChoice()` устаревшим методом. Drupal реализует данный интерфейс в `\Drupal\Core\Validation\DrupalTranslator:transChoice()`. Таким образом `\Drupal\Core\Validation\DrupalTranslator:transChoice()` также объявлен устаревшим.

## Функция `module_load_include()` объявлена устаревшей

* [#697946](https://www.drupal.org/node/697946) 

Функция `module_load_include()` объявлена устаревшей, вместо неё используйте метод `::loadInclude()` [сервиса](../../../../9/services/index.md) `module_handler`.

**Ранее:**

```php
module_load_include($type, $module, $name = NULL);
```

**Сейчас:**

```php
\Drupal::moduleHandler()->loadInclude($module, $type, $name = NULL);
```

> [!NOTE]
> Обратите внимание, что `ModuleHandler::loadInclude()` и `module_load_include()` имеют разный порядок аргументов.

> [!NOTE]
> `ModuleHandler::loadInclude()` не поддерживает подключение кода для отключенных расширений ([модулей](../../../../9/modules/index.md), [тем оформления](../../../../9/themes/index.md), [установочных профилей](../../../../9/distributions/index.md)).

## Функция `module_load_install()` объявлена устаревшей

* [#2010380](https://www.drupal.org/node/2010380) 

Функция `module_load_install()` объявлена устаревшей, вместо неё используйте метод `::loadInclude()` сервиса `module_handler`.

**Ранее:**

```php
module_load_include($module);
```

**Сейчас:**

```php
\Drupal::moduleHandler()->loadInclude($module, 'install');
```

## Сервисы использующие Laminas\Feed объявлены устаревшими

* [#2979588](https://www.drupal.org/node/2979588), [#2919215]

Сервисы, использующие `Laminas\Feed\Reader` (`feed.reader.*`) и `Laminas\Feed\Writer` (`feed.writer.*`) расширения, объявлены устаревшими. Также объявлен устаревшим сервис `feed.bridge.writer`, который ранее использовался как замена для сервисов `feed.writer.*`.

Замены для `Laminas\Feed\Reader`:

* `feed.reader.dublincoreentry` → `\Drupal::service('feed.bridge.reader')->get('DublinCore\Entry')`
* `feed.reader.dublincorefeed` → `\Drupal::service('feed.bridge.reader')->get('DublinCore\Feed')`
* `feed.reader.contententry` → `\Drupal::service('feed.bridge.reader')->get('Content\Entry')`
* `feed.reader.atomentry` → `\Drupal::service('feed.bridge.reader')->get('Atom\Entry')`
* `feed.reader.atomfeed` → `\Drupal::service('feed.bridge.reader')->get('Atom\Feed')`
* `feed.reader.slashentry` → `\Drupal::service('feed.bridge.reader')->get('Slash\Entry')`
* `feed.reader.wellformedwebentry` → `\Drupal::service('feed.bridge.reader')->get('WellFormedWeb\Entry')`
* `feed.reader.threadentry` → `\Drupal::service('feed.bridge.reader')->get('Thread\Entry')`
* `feed.reader.podcastentry` → `\Drupal::service('feed.bridge.reader')->get('Podcast\Entry')`
* `feed.reader.podcastfeed` → `\Drupal::service('feed.bridge.reader')->get('Podcast\Feed')`

Замены для `Laminas\Feed\Writer` и `feed.bridge.writer`:

Для замены вам потребуется [объявить собственный сервис](../../../../9/services/create/index.md) со следующим содержанием:

```yaml
  feed.bridge.writer:
    class: Drupal\Component\Bridge\ZfExtensionManagerSfContainer
    calls:
      - [setContainer, ['@service_container']]
      - [setStandalone, ['\Laminas\Feed\Writer\StandaloneExtensionManager']]
    arguments: ['feed.writer.']
```

Вы также можете использовать класс `\Laminas\Feed\Writer\StandaloneExtensionManager` напрямую.


## В листинге модулей добавлено визуальное обозначение для устаревших модулей, а попытка включения устаревших модулей или тем будет показывать предупреждение

* [#3215043](https://www.drupal.org/node/3215043) 

Теперь, на странице со списком модулей, будет визуальная индикация для устаревших модулей. Это позволит предупреждать пользователей заранее, что включение модуля может иметь последствия.

Также, при попытке включения таких модулей и тем, будет показываться системное сообщение, предупреждающее о том, что модули или тема объявлены устаревшими и их использование на свой страх и риск.

## Aggregator

* [#2610520](https://www.drupal.org/node/2610520) Улучшена справка о блоке предоставляемом модулем.

## Cache System

* [#2873732](https://www.drupal.org/node/2873732) Внесены улучшения в `CookiesCacheContext`, который мог приводить к ошибке «Array
  to string conversion in CacheContextsManager::convertTokensToKeys()».

## CKEditor 5

* [#255077](https://www.drupal.org/node/255077) Исправлены опечатки в описании для CKEditor 5 фильтра.

## Claro

* [#3214170](https://www.drupal.org/node/3214170) Кнопка «Отмена» теперь отцентрована в off-canvas модальном окне.
* [#3247994](https://www.drupal.org/node/3247994) Исправлена неполадка, из-за которой мог некорректно работать элемент формы `password`.

## Composer

* [#3246595](https://www.drupal.org/node/3246595) Зависимости ядра обновлены на 01.11.21.
* [#3255623](https://www.drupal.org/node/3255623) Удалены замены для пакетов `paragonie/random_compat` и `symfony/polyfill-php70`.

## Configuration System

* [#2343517](https://www.drupal.org/node/2343517) Удалён код и упоминания с `@todo` на задачи, которые решены.

## Database Logging

* [#3246471](https://www.drupal.org/node/3246471) Из класса `DbLogController` удалены свойства, которые объявлены у родительского.

## Database System

* [#3213644](https://www.drupal.org/node/3213644) Внесены улучшения в тест `StatementWrapperLegacyTest::testClientStatementMethod()`.

## Datetime

* [#3251100](https://www.drupal.org/node/3251100) Исправлена неполадка, из-за которой `DateTimeWidgetBase` дважды устанавливал одну и ту же временную зону.

## Extension System

* [#3245622](https://www.drupal.org/node/3245622) Ссылкам на справку и права доступа конкретного модуля, в списке модулей, добавлено указание, для какого модуля они а также `title` аттрибуты.

## Field System

* [#3213023](https://www.drupal.org/node/3213023) `FieldConfig` теперь выводит более полезные и понятные сообщения об
  ошибках.

## Filter

* [#3227821](https://www.drupal.org/node/3227821) Исправлена неполадка в фильтре «Заменять переводы строк соответствующими HTML-тегами», которая могла приводить к поломке SVG элементов.

## Image

* [#3223233](https://www.drupal.org/node/3223233) Улучшены заголовки страниц для форм добавления и редактирования [стилей изображений](../../../../9/image/image-styles/index.md).

## JavaScript

* [#3238860](https://www.drupal.org/node/3238860) Использование `jQuery.map()` заменено на нативную `map()` функцию.
* [#3239500](https://www.drupal.org/node/3239500) Добавлен полифил для `Array.includes()`.

## JSON:API

* [#3199696](https://www.drupal.org/node/3199696) `ResourceObject` теперь учитывает текущий язык и корректно кеширует результаты для мультиязычных ресурсов.

## Layout Builder

* [#3253666](https://www.drupal.org/node/3253666) `LayoutTempstoreRouteEnhancer` теперь использует `
  Drupal\Core\Routing\RouteObjectInterface` вместо `Symfony\Cmf\Component\Routing\RouteObjectInterface`.
* [#3190541](https://www.drupal.org/node/3190541) Исправлена неполадка, из-за которой кеш-контекст `layout_builder_is_active` мог предоставлять некорректные значения.

## Media Library

* [#3173770](https://www.drupal.org/node/3173770) `MediaLibraryFieldWidgetOpener` теперь позволяет использовать другие референс поля, расширяющее поле из ядра и использующие `EntityReferenceFieldItemList`.
* [#3248454](https://www.drupal.org/node/3248454) Внесены улучшения в `MediaLibraryStateTest` для совместимости с Symfony 5.4.

## Migration System

* [#3246053](https://www.drupal.org/node/3246053) Обновлено значение `filesize` файла `ds9.txt` в `file_managed`.
* [#3163663](https://www.drupal.org/node/3163663) Исправлена неполадка из-за которой плагин-обработчик `download` мог приводить к предупреждению «failed to open stream: Too many open file».
* [#3087332](https://www.drupal.org/node/3087332) Плагин-обработчик `d6_url_alias_language` объявлен устаревшим.
* [#3042533](https://www.drupal.org/node/3042533) Внесены улучшения в миграцию словарей таксономии из Drupal 6.
* [#3240109](https://www.drupal.org/node/3240109) Возвращаемый тип данных для `MigrateProcessInterface::transform()` изменён с `string|array` на `mixed`.
* [#3258009](https://www.drupal.org/node/3258009) Удалены два неиспользуемых и сломанных плагина `fieldleft` и `fieldright`.

## Olivero

* [#3186992](https://www.drupal.org/node/3186992) Исправлена неполадка, из-за которой навигационные пункты меню могли выходить за рамки контейнера.

## Standard Profile

* [#3171149](https://www.drupal.org/node/3171149) Стандартный профиль теперь использует стиль изображения `wide` для
  публикаций вместо `large`.

## Update

* [#3253639](https://www.drupal.org/node/3253639) Добавлен новый класс `UpdateUploaderTestBase` с универсальной реализацией `::setUp()` для уменьшения дублирования кода в тестах модуля.

## User

* [#3198010](https://www.drupal.org/node/3198010) Улучшено отображение ошибки о неудачных попытках авторизации, чтобы снизить нагрузку на систему. `UserLoginForm` теперь принимает сервис `bare_html_page_renderer` в качестве аргумента.

## Views

* [#2569381](https://www.drupal.org/node/2569381) Из `Drupal\views\Plugin\views\area\Result` удалён лишний вызов `XSS::adminFilter()`.

## Symfony 6

* [#3232074](https://www.drupal.org/node/3232074) Для классов расширяющих `Normalizer` добавлен тайпхинт `array|string|int|float|bool|\ArrayObject|null` методу `::normalize()`.
* [#3232095](https://www.drupal.org/node/3232095) Произведён рефакторинг сервиса `update.root` для того чтобы он возвращал объект вместо строки.
* [#3232131](https://www.drupal.org/node/3232131) В `DebugClassLoader` добавлены тайпхинты.
* [#3250299](https://www.drupal.org/node/3250299) Внесены улучшения в констрейнты для совместимости с Symfony 6.
* [#3250442](https://www.drupal.org/node/3250442) Код с использованием Prophecy на методы с тайпхинтом на возвращаемое значение `static` заменены на моки для исправления проблем, пока поддержка не появится в Prophecy.

## Прочие изменения

* [#3038596](https://www.drupal.org/node/3038596) В `drupalci.yml` добавлено напоминание о том, что данный файл требуется в ручном изменении при создании ново ветки ядра.
* [#3245820](https://www.drupal.org/node/3245820) Из кода удалены упоминания удалённого `\Drupal\Core\Action\Plugin\Action\PublishAction`.
* [#3250263](https://www.drupal.org/node/3250263) Удалён неиспользуемый файл `core/scripts/test/test.script`.
* [#3251891](https://www.drupal.org/node/3251891) Исправлены неполадки, приводящие к проблемам на ветке Drupal 10.
* [#3251988](https://www.drupal.org/node/3251988) Обновлена документация о возвращаемых типах данных для `JSWebAssert::waitForText()`.
* [#3256451](https://www.drupal.org/node/3256451) Исправлен некорректный хэш в `composer.lock`.
* [#3210129](https://www.drupal.org/node/3210129) Внесены исправления опечаток для слов, начинающихся с «a» до «d» включительно.
* [#3175428](https://www.drupal.org/node/3175428) В документацию [настройки](../../../../9/settings-php/index.md) `trusted_host_patterns` добавлена ссылка на более детальную документацию.
* [#3031130](https://www.drupal.org/node/3031130) Улучшено поведение для `\Drupal\Core\KeyValueStore\MemoryStorage::rename()`. Если старое название и новое идентичны, данные не будут удалены, как это было ранее.
* [#3258014](https://www.drupal.org/node/3258014) Оформление информации об устаревшем коде улучшено для `Drupal\migrate_drupal\Plugin\migrate\field\NodeReference` и `
  Drupal\taxonomy\Plugin\views\argument_validator\Term`.
* [#3256539](https://www.drupal.org/node/3256539) `ContentEntityDeleteForm` больше не помечен как внутренний класс (`@internal`), это означает, что вы теперь можете использовать и наследоваться от этой формы.
