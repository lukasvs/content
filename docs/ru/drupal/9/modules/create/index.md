---
title: Создание модуля
slug: wiki/9/modules/create
core: 9
metatags:
  title: 'Drupal 9: Создание модуля'
  description: 'Создание собственного модуля в Drupal 9.'
search-keywords:
  - как создать свой модуль
  - где писать код
  - куда вставлять код
category:
  area: Модули
  title: Создание модуля
  order: 2
---

Процесс создания модуля достаточно простой и не требует серьезных навыков, тем не менее написание функционала уже может потребовать от вас определенные навыки в PHP.

Минимальный состав модуля — папка, которая станет корнем модуля, а также файл `*.info.yml`, описывающий модуль.

> [!TIP]
> Для быстрого создания всех необходимых файлов и папок модуля, можете использовать команду [Drush](../../../../drush/index.md) `drush generate module`.

## Даем название модулю

У модуля обязательно должно быть название (машинное имя). Оно будет использовано Drupal, для того чтобы взаимодействовать с вашим модулем. Для именования модуля есть определенные правила, которые нельзя нарушать:

- Название должно начинаться с буквы.
- Может содержать только латинские символы в нижнем регистре и нижнее подчеркивание.
- Не может содержать никаких пробелов.
- Не может называться одним из зарезервированных имен: `src`, `lib`, `vendor`, `assets`, `css`, `files`, `images`, `js`, `misc`, `templates`, `includes`, `fixtures`, `drupal`.
- Не может называться также, как модули, поставляемые ядром.

> [!WARNING]
> Крайне не рекомендуется использовать для названия модуля уже занятые имена на drupal.org. Особенно если вы планируете использовать одноименный контриб модуль, или публиковать свой.

## Создание папки для модуля

Например, вы хотите назвать свой будущий модуль `hello_world`, для этого потребуется сначала создать папку с соответствующим названием по одному из пути: `/modules/custom/hello_world` или `/sites/all/modules/hello_world` (только при использовании мультисайтинга). Вы можете не создавать папку `custom` и разместить модуль по пути `/modules/hello_world`, но это крайне не рекомендуется, так как ваши собственные модули для проекта будут перемешаны с контриб модулями.

> [!TIP]
> Лучшей практикой является создание собственных модулей по пути `/modules/custom/NAME`, а модули с drupal.org в `/modules/contrib/NAME`. Это позволит вам исключить загруженные модули из VCS (git), а также поможет легче ориентироваться где код под проект, а где готовые решения. Загружая модули при помощи [Composer](../../../../composer/index.md), они автоматически будут располагаться в `/modules/contrib`.

## Создание .info.yml файла

В папке, созданной для модуля, вы также должны создать файл с названием вашего модуля и `.info.yml` окончанием. Например, для модуля `hello_world` необходимо создать `hello_world.info.yml`. Данный файл обязательно должен иметь машинное название модуля.

В данном файле вы описываете базовую информацию вашего модуля.

Пример:

```yaml
name: Hello World Module
description: Creates a page showing "Hello World".
package: Custom

type: module
core_version_requirement: ^9

dependencies:
  - drupal:link
  - drupal:views
  - paragraphs:paragraphs
  - webform:webform (>=8.x-5.x)

test_dependencies:
 - drupal:image

configure: hello_world.settings

php: 5.6

hidden: true
```

### Структура .info.yml файла

- **name**: Название модуля, отображаемое в административном интерфейсе. Например `name: Hello World Module`.
- **type**: Тип "расширения" друпал. Может принимать одно из следующих значений: module, theme, profile. В случае модулей, обязательно указывать "module". Например `type: module`.
- **core_version_requirement**: (обязательно) Указывает версии ядра, с которыми совместима тема оформления. Принимает запись версий с [семантическим версионированием](../../../../semver/index.md), также, в нём можно указать множественные версии. Например: `core_version_requirement: ^8`, `core_version_requirement: ^8 || ^9`, `core_version_requirement: ^8.8 || ^9`.
- **description**: (опционально) Описание модуля, отображаемое на странице списка модулей под названием. Например `description: Example Module description.`.
- **package**: (опционально) Название группы, в которой необходимо расположить модуль в административном интерфейсе. По умолчанию все модули получают значение `Custom`, вы можете указывать собственное название. Например `package: Project name`. _Название группы обрабатывается `t()`, это означает, что строка будет переводимой и необходимо использовать только латинские названия._
- **dependencies**: (опционально) Список модулей, от которых зависит данный модуль. Название зависимостей должно быть в формате `{project}:{module}`, где `{project}` название проекта на drupal.org (например drupal.org/project/views - views) и `{module}` машинное название модуля. Вы также можете указать каждой зависимости ограничения по версии, например `yandex_yml:yandex_yml (>=8.x-1.x)`.
- **test_dependencies**: (опционально) В соответствии с "dependencies" указывается список зависимостей, необходимы для тестирования модуля при помощи DrupalCI, но которые не нужны для реального использования.
- **configure**: Если ваш модуль имеет настройки, вы можете указать название [маршрута](../../routing/index.md) в данном параметре. Для активного модуля с данным значением, в списке модулей будет дополнительная кнопка для перехода в настройки. Например `configure: hello_world.settings`.
- **php**: (опционально) Указание минимальной версии PHP необходимой для работы модуля. Пользователи не смогут включить модуль, если версия, на которой работает сайт, не удовлетворяет данному требованию.
- **hidden**: (опционально) По умолчанию значение `false`, указав `true`, модуль не будет отображаться в списке модулей. Например, это хорошее решение для модулей с тестами или модулей с примерами, которые не предназначены для использования. Вы можете указать `$settings['extension_discovery_scan_tests'] = TRUE` в [settings.php](../../settings-php/index.md) чтобы они начали отображаться.
- **lifecycle**: (опционально) Указывает стабильность модуля. Может принимать одно из следующих значений:
  - **experimental**: Что-то новое, не завершённое. Будет показано предупреждение при попытке включить подобное расширение, но оно будет работать.
  - **stable**: (по умолчанию) Стабильное расширение, никаких дополнительных предупреждений показано не будет.
  - **deprecated**: Что-то на пути к выводу из применения. Пользователь по-прежнему сможет устанавливать подобное расширение, но будут показываться предупреждения.
  - **obsolete**: Поддержка прекращена. Пользователь должен удалить данное расширение. Ранее установленные расширения продолжат работать, но будет показано предупреждение. Установить данное расширение будет невозможно.
- **lifecycle_link**: (обязательно при **lifecycle** равным **obsolete** или **deprecated**) URL-адрес на документацию, где написано, что делать разработчикам и/или веб-мастерам в случае прекращения развития модуля.
- **Запрещенные для использования**: Данные файлы могут также содержать `version` и `project`. Данные значения добавляются в файл автоматически drupal.org, если вы публикуете модуль как полноценный проект. Для избежания проблем, не задавайте данные значение вручную.

## Создание composer.json

Создание composer.json файла позволяет корректно обрабатывать модуль при помощи [Composer](../../../../composer/index.md). Данный файл является обязательным если вы собираетесь публиковать модуль на drupal.org, а также он имеет зависимости отличные от ядра.

При помощи данного файла вы объявляете свой модуль как пакет Composer. Минимально необходимыми значениями являются `name` - машинное имя пакета в формате `drupal/{module_name}` и `type` равный `drupal-module`.

Вы также можете указывать другие необходимые параметры, зависимости, всё как это делается в Composer. Не забудьте также указать зависимости в виде других контриб модулей в `require`.

Пример:

```json
{
    "name": "drupal/mobile_detect",
    "description": "Mobile_Detect is a lightweight PHP class for detecting mobile devices.",
    "type": "drupal-module",
    "homepage": "https://drupal.org/project/mobile_detect",
    "authors": [
        {
            "name": "Matthew Donadio (mpdonadio)",
            "homepage": "https://www.drupal.org/u/mpdonadio",
            "role": "Maintainer"
        },
        {
            "name": "Darryl Norris (darol100)",
            "email": "admin@darrylnorris.com",
            "homepage": "https://www.drupal.org/u/darol100",
            "role": "Co-maintainer"
        }
    ],
    "support": {
        "issues": "https://drupal.org/project/issues/mobile_detect",
        "irc": "irc://irc.freenode.org/drupal-contribute",
        "source": "https://cgit.drupalcode.org/mobile_detect"
    },
    "license": "GPL-2.0-or-later",
    "minimum-stability": "dev",
    "require": {
        "mobiledetect/mobiledetectlib": "~2.8"
    }
}
```

> [!NOTE]
> При генерации модуля с помощью [Drush](../../../../drush/index.md), вам будет предложено сгенерировать данный файл. Если вы пропустили данный шаг, или хотите сгенерировать для уже существующего модуля, воспользуйтесь командой `drush generate composer`.

## Изменения в релизах

- **[Drupal 9.3.0](../../../releases/9/9.3.x/9.3.0/index.md) (08.12.2021):** Добавлена поддержка свойств `lifecycle` и `lifecycle_url` в ***.info.yml** файле.
- **[Drupal 9.2.0](../../../releases/9/9.2.x/9.2.0/index.md) (02.06.2021):** Модули больше не могут указывать значение `major` в ***.info.yml** файле.

## Ссылки

- [Официальная документация по созданию модулей](https://www.drupal.org/docs/8/creating-custom-modules) (англ.)
- [Drupal 8 Hello World: Пишем свой первый модуль](https://niklan.net/blog/66), Niklan, 2014
