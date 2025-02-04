---
title: Создание темы оформления
slug: wiki/8/themes/create
core: 8
metatags:
  title: 'Drupal 8: Создание темы оформления'
  description: 'Создание собственной темы оформления. Структура её *.info.yml файла, минимальный состав, требования и прочие нюансы.'
search-keywords:
  - как создать свою тему
  - где верстать
  - где держать верстку
  - куда ложить верстку
  - как поменять оформление сайта
category:
  area: Темы оформления
  title: Создание темы
  order: 1
---

Создание темы оформления состоит из нескольких простых, обязательных шагов — создание папки для темы и файла *.info.yml с описанием темы. Данный процесс похож на создание собственного [модуля](../../modules/index.md), с некоторыми особенностями для темы.

## Придумываем название

Первым делом необходимо придумать название будущей темы оформления. Оно будет использовано Drupal, для того чтобы взаимодействовать с вашей темой оформления. Для именования темы оформления есть определенные правила, которые нельзя нарушать:

- Название должно начинаться с буквы.
- Может содержать только латинские символы в нижнем регистре и нижнее подчеркивание.
- Не может содержать никаких пробелов.
- Не может называться одним из зарезервированных имён: `src`, `lib`, `vendor`, `assets`, `css`, `files`, `images`, `js`, `misc`, `templates`, `includes`, `fixtures`, `drupal`.
- Не может называться также, как модули или темы, поставляемые ядром.

> [!WARNING]
> Крайне не рекомендуется использовать для названия темы оформления уже занятые имена на drupal.org. Это касается как модулей, так и тем.

> [!IMPORTANT]
> Drupal отличает темы от модулей, это значит, что в пределах одного сайта может быть как модуль, с названием `example`, так и тема оформления. Не допускайте подобного, так как у вас будут проблемы с [хуками](../../hooks/index.md) из-за того, что вы работаете в одинаковом неймспейсе, а также множество других проблем, которые появятся не сразу. Пожалуйста, придумайте уникальное название в пределах сайта, как минимум.

## Создание папки

После того как придумали название темы оформления, вам нужно создать папку с данным именем в **/themes/custom/my_theme_name**, либо в **/themes/my_theme_name**.

> [!TIP]
> Лучшей практикой является создание собственных тем по пути **/themes/custom**, так как их можно добавить в систему контроля версий (git), при этом исключить контрибные темы, которые могут являться базовыми или административными. Устанавливая темы при помощи [Composer](../../../../composer/index.md), темы оформления будут автоматически находиться в **/themes/contrib**.

## Создание *.info.yml файла

В созданной папке, вам необходимо добавить ***.info.yml** файл, в котором описывается тема. В качестве название файла вы должны использовать машинное имя темы. Например, если вы создали папку **my_theme_name**, то файл должен называться **my_theme_name.info.yml**.

Пример содержимого файла:

```yaml
name: My awesome theme
type: theme
description: 'A cuddly theme that offers extra fluffiness.'
core: 8.x
libraries:
  - my_theme_name/global-styling
base theme: classy
regions:
  header: Header
  content: Content
  sidebar_first: 'Sidebar first'
  footer: Footer
```

### Структура *.info.yml файла

- **name**: Название темы оформления, отображаемое в административном интерфейсе. Например `name: My awesome theme`.
- **type**: Тип "расширения" друпал. Может принимать одно из следующих значений: module, theme, profile. В случае тем оформления, обязательно указывать "theme". Например `type: theme`.
- **core**: Версия ядра, для которой предназначена данная тема оформления. Для Drupal 8 это `8.x`. Например `core: 8.x`.
- **description**: (опционально) Описание темы оформления, отображаемое на странице списка тем под названием. Например `description: The theme for my website!`.
- **package**: (опционально) Название группы, в которой необходимо расположить тему в административном интерфейсе. По умолчанию все темы получают значение `Custom`, вы можете указывать собственное название. Например `package: Project name`. _Название группы обрабатывается `t()`, это означает, что строка будет переводимой и необходимо использовать только латинские названия._
- **php**: (опционально) Указание минимальной версии PHP необходимой для работы модуля. Пользователи не смогут включить модуль, если версия, на которой работает сайт, не удовлетворяет данному требованию.
- **libraries**: (опционально) Список [библиотек](../../libraries/index.md), которые необходимо добавлять на всех страницах темы. Аккуратно используйте данное значение. Подключать все и сразу может серьезно сказаться на производительности.
- **libraries-override**: (опционально) Позволяет переопределить библиотеки или специфичные для неё файлы, или выключить их работу.
- **libraries-extend**: (опционально) Позволяет указать какие библиотеки необходимо подключать, если на странице появилась определенная библиотека.
- **base theme**: (обязательно, начиная с [Drupal 8.8.0](../../../releases/8/8.8.x/8.8.0/index.md)) Позволяет указать название базовой темы, от которой будет наследоваться ваша тема. Если вы не загрузили необходимую базовую тему, рекомендуется использовать одну из стандартных: `classy` или `stable`. Вы можете полностью отключить наследование указав `false`. Если вы до конца не понимаете данный параметр, пропустите его и продолжайте со значением по умолчанию - `stable`.
- **hidden**: (опционально) По умолчанию значение `false`, указав `true`, модуль не будет отображаться в списке тем оформления. Например, это хорошее решение для тем оформления с тестами или стартовых наборов тем, которые не предназначены для использования. Вы можете указать `$settings['extension_discovery_scan_tests'] = TRUE` в [settings.php](../../settings-php/index.md) чтобы они начали отображаться.
- **engine**: (опционально) Позволяет задать какой шаблонизатор использовать для данной темы. По умолчанию все темы имеют значение равное `twig` и указывать ничего не нужно. Прежде чем менять данный параметр, убедитесь что указанный вами шаблонизатор объявлен в Drupal.
- **logo**: (опционально) Путь до логотипа, относительно корня темы (*.info.yml). По умолчанию Drupal ищет **logo.svg**, но вы можете поменять это значение, например, если хотите чтобы он использовал png логотип: `logo: images/logo.png`.
- **screenshot**: (опционально) Путь до скриншота, относительно корня темы (*.info.yml). По умолчанию Drupal попробует загрузить файл **screenshot.png** из корня темы. Вы можете подготовить свой файл, он должен быть в размере 588х438px. Данный скриншот показывается на странице тем оформления.
- **regions**: (опционально) Список регионов темы оформления. Если вы решили задать данное значение, вы обязательно должны объявить регион **content**. Регионы задаются в формате `region_key: Region label`.
- **regions_hidden**: (опционально) Список машинных имён регионов, которые необходимо удалить, если вы не указали **regions** и они унаследовались от **base theme**.
- **features**: (опционально) Список настроек, которые будут доступны в форме настройки темы. Доступные значения по умолчанию: `comment_user_verification`, `comment_user_picture`, `favicon`, `logo`, `node_user_picture`.
- **ckeditor_stylesheets**: (опционально) Список CSS файлов, которые необходимо добавить к фрейму CKEdtior.
- **experimental**: (опционально, добавлено в [Drupal 8.8.0](../../../releases/8/8.8.x/8.8.0/index.md)) Помечает тему как "экспериментальная". Для таких тем будет добавлено уведомление, как у модулей, предупреждающих о возможных проблемах.
- **dependencies**: (опционально, добавлено в [Drupal 8.9.0](../../../releases/8/8.9.x/8.9.0/index.md)) Список модулей, от которых зависит данная тема оформления. Название зависимостей должно быть в формате `{project}:{module}`, где `{project}` название проекта на drupal.org (например drupal.org/project/views - views) и `{module}` машинное название модуля. Вы также можете указать каждой зависимости ограничения по версии, например `yandex_yml:yandex_yml (>=8.x-1.x)`.
- **Запрещенные для использования**: Данные файлы могут также содержать `version` и `project`. Данные значения добавляются в файл автоматически drupal.org, если вы публикуете модуль как полноценный проект. Для избежания проблем, не задавайте данные значение вручную.

## Изменения в релизах

- **[Drupal 8.7.8](../../../releases/8/8.7.x/8.7.8/index.md) (02.10.2019):** Добавлена поддержка `core_version_requirement` в `.info.yml`.
- **[Drupal 8.8.0](../../../releases/8/8.8.x/8.8.0/index.md) (04.12.2019):** Параметр `base theme` `.info.yml` файла теперь обязателен для заполнения.
- **[Drupal 8.8.0](../../../releases/8/8.8.x/8.8.0/index.md) (04.12.2019):** Добавлен опциональный параметр `experimental` в `.info.yml`.
- **[Drupal 8.9.0](../../../releases/8/8.9.x/8.9.0/index.md) (03.06.2020):** Добавлен опциональный параметр `dependencies` в `.info.yml`.

## Ссылки

- [Defining a theme with an .info.yml file](https://www.drupal.org/docs/8/theming-drupal-8/defining-a-theme-with-an-infoyml-file) (англ.), drupal.org
- [Разработка тем оформления для Drupal 8](https://iamdroid.net/ru/blog/drupal-8-theme-development-prepare), I am droid, 2017
