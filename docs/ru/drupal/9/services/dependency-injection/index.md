---
title: Dependency Injection
slug: wiki/9/services/dependency-injection
core: 9
search-keywords:
  - dependency injection
metatags:
  title: 'Drupal 9: Dependency Injection (DI)'
  description: 'Dependency Injection — подход инициализации объектов с автоматической подготовкой зависимостей и их последующей передачей в качестве аргументов.'
category:
  area: Сервисы
  title: Dependency Injection
  order: 3
---

**Dependency Injection (DI)** — подход инициализации объектов с автоматической подготовкой зависимостей и их последующей передачей в качестве аргументов.

В общем смысле, DI - это контейнер, который подготавливает объекты запрошенных сервисов, и передает их экземпляры в конструктор инициализируемого класса, который использует DI.

В Drupal можно встретить несколько реализаций DI, которые зависят от того, с чем мы на данный момент работаем. В каждом случае есть только один правильный подход, и только он заработает.

## Два способа инъекции зависимостей

В данном разделе представлен старый и новый способ инъекции зависимостей **для классов которые наследуются от родителей с собственным конструктором**.

### Старый

Данный подход вы можете заметить как в ядре, так и в контрибе. Данный способ использовался с момента релиза Drupal 8 и применяется до сих пор, но со временем стали понятны его недостатки и неудобства, поэтому был выработан более универсальный способ о котором чуть позже.

Старый подход выглядит примерно следующим образом:

```php
  /**
   * Constructs a new FooBlock object.
   *
   * @param array $configuration
   *   The plugin configuration.
   * @param $plugin_id
   *   The plugin ID.
   * @param $plugin_definition
   *   The plugin definition.
   * @param \Drupal\Core\Session\AccountProxyInterface $account_proxy
   *   The account proxy.
   */
  public function __construct(array $configuration, $plugin_id, $plugin_definition, AccountProxyInterface $account_proxy) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);

    $this->currentUser = $account_proxy;
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    return new static(
      $configuration,
      $plugin_id,
      $plugin_definition,
      $container->get('current_user')
    );
  }
```

В данном подходе, наследник принимает как свои аргументы, так и аргументы родительского класса. В конструкторе аргументы родителя передаются в его конструктор для последующей обработки.

Недостатки данного подхода:

- Необходимо следить за конструктором. Если родительский класс поменяет порядок, количество или тип своих зависимостей, наш класс перестанет работать, и потребует доработки.
- Все классы наследуемые от нашего, также должны следить за нашим конструктором. Такая «матрёшка» может приводит к большому количеству аргументов.

### Современный

Данный подход приходит на замену предыдущему и решает все его проблемы разом. Данный подход постепенно внедряется в ядро начиная с [Drupal 9](../../index.md).

Он имеет два варианта.

**Первый вариант** применяется, когда у родительского класса объявлен статический метод `create()`. В таком случае вызов проходит через него:

```php
  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    $instance = parent::create($container, $configuration, $plugin_id, $plugin_definition);
    $instance->currentUser = $container->get('current_user');

    return $instance;
  }
```

Как вы можете заметить, в данном подходе вызывается родительский `create()` а его результат записывается в экземпляр объекта `$instance`. Затем, мы добавляем в него свойство с нужным нам сервисом.

**Второй вариант** используется в том случае, когда родительский класс не объявляет `create()` (плагины где DI опционален, например блоки). В таком случае мы инициализируем объект сами:

```php
  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    $instance = new static($configuration, $plugin_id, $plugin_definition);
    $instance->currentUser = $container->get('current_user');

    return $instance;
  }
```

В данном случае мы сами инициализируем объект без обращения к родительскому методу, так как его нет.

Как вы можете заметить, конструктор в данном случае не описывается, так как в этом нет необходимости, его сигнатура остаётся неизменной (родительской).

Преимущества нового подхода:

- Позволяет сократить количество кода, благодаря отсутствию необходимости описания конструктора.
- Если у родителя нет конструктора и в нём используется аналогичный, современный подход, это упрощает поддержку кода, так как изменения зависимостей никак не отразится на инициализации его наследников.
- Проще вносить изменения. Добавление новых зависимостей не повлечет никаких проблем, так как наследникам не потребуется следить за этим. Аналогично, появление новых зависимостей от родительского элемента не сломает наш.

## Процедурный код

Данный подход **не является Dependency Injection**, но это то, как иногда приходится обращаться к сервисам и внедрять их в legacy код.

В Drupal существует одноимённый глобальный класс `\Drupal`, который имеет свой собственный Service Container из которого можно вызывать сервисы. Основным методом класса является `::service()`, где в качестве аргумента принимается название сервиса, а в качестве результата, возвращается объект.

```php
$service = \Drupal::service('example.simple');
```

Класс также содержит методы, которые являются синонимами для вызова конкретных сервисов из ядра, которые чаще всего используются. Ознакомиться с ними вы можете внутри класса.

> [!IMPORTANT]
> Везде, где есть поддержка Dependency Injection данный подход — неправильный и является плохой практикой. Если вы используете данный класс в коде с поддержкой Dependency Injection — задумайтесь о рефакторинге.

## Dependency Injection в сервисах

Dependency Injection у сервисов производится через конструктор, или одно из соответствующих свойств типа `configurator`, `factory` и `calls`.

В общем случае используется DI через конструктор (`__construct()`) класса.

Для этого, при объявлении сервиса указываются необходимые зависимые сервисы в свойстве `arguments`. Например:

```yaml
services:
  example.complex:
    class: Drupal\mymodule\MyObject
    arguments: ['@example.foo', '@example.bar']
```

Затем, данные аргументы принимаются в качестве параметров конструктора в том же самом порядке, в каком они указаны в `arguments`.

```php
<?php

namespace Drupal\mymodule;

/**
 * Provides my special service.
 */
class MyObject {

  /**
   * The foo object.
   *
   * @var Drupal\example\FooInterface
   */
  protected $foo;

  /**
   * The bar object.
   *
   * @var Drupal\example\BarInterface
   */
  protected $bar;

  /**
   * Constructs a new MyObject.
   *
   * @param Drupal\example\FooInterface $foo
   *   The foo object.
   * @param Drupal\example\BarInterface $bar
   *   The bar object.
   */
  public function __construct(FooInterface $foo, BarInterface $bar) {
    $this->foo = $foo;
    $this->bar = $bar;
  }

}
```

## Dependency Injection в плагинах

При работе с [плагинами](../../plugins/index.md) вы также можете использовать Dependency Injection, но только для тех типов плагинов, где предусмотрена поддержка DI. В подавляющем большинстве случаев DI поддерживается всеми плагинами, поэтому, вы можете сразу пробовать использовать DI, но если что-то не получается, возможно плагин не предусмотрел поддержку DI. Подобное поведение обычно делается осмысленно разработчиками. Поэтому, нужно уточнить, с какой целью это сделано. Возможно, вы найдете ответ в менеджере плагинов.

Для того чтобы использовать DI в плагине, необходимо классу данного плагина указать что он реализует интерфейс `Drupal\Core\Plugin\ContainerFactoryPluginInterface`. При инициализации плагинов, Drupal проверяет, реализует ли конкретный класс плагина данный интерфейс или нет, если реализует, он создаёт экземпляр через метод `create()`, в противном случае он просто инициализирует объект без DI.

Данный интерфейс делает необходимым создание статического метода `create()`, который принимает 4 аргумента:

- `$container`: Service Container, при помощи которого вы можете вызывать сервисы.
- `$configuration`: Массив с настройками, переданными менеджером при создании плагина.
- `$plugin_id`: Идентификатор плагина.
- `$plugin_definition`: Массив с тем, что Drupal узнал об этом плагине.

Все плагины, в конечном итоге, наследуются от других классов. Вы должны учитывать данное поведение при внедрении DI. По умолчанию, все плагины принимают, как минимум, `$configuration`, `$plugin_id` и `$plugin_definition`. Таким образом, три данных аргумента являются обязательным для плагинов. Данные аргументы принимаются в базовом плагине, поэтому вам также необходимо их добавить и передать родительскому классу.

Метод `create()` должен вернуть объект (`new self()`, `new static()`) плагина, сохраняя оригинальные аргументы для конструктора плагина.

**Плагин блока без DI:**

```php
<?php

namespace Drupal\foo\Plugin\Block;

use Drupal\Core\Block\BlockBase;

/**
 * Provides an example block.
 *
 * @Block(
 *   id = "foo_block",
 *   admin_label = @Translation("Example"),
 *   category = @Translation("Custom")
 * )
 */
class FooBlock extends BlockBase {

  /**
   * {@inheritdoc}
   */
  public function build() {
    $build['content'] = [
      '#markup' => $this->t('It works!'),
    ];
    return $build;
  }

}
```

**Плагин блока с DI:**

```php
<?php

namespace Drupal\foo\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Plugin\ContainerFactoryPluginInterface;
use Drupal\Core\Session\AccountProxyInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * Provides an example block.
 *
 * @Block(
 *   id = "foo_block",
 *   admin_label = @Translation("Example"),
 *   category = @Translation("Custom")
 * )
 */
class FooBlock extends BlockBase implements ContainerFactoryPluginInterface {

  /**
   * The current user.
   *
   * @var \Drupal\Core\Session\AccountProxyInterface
   */
  protected $currentUser;

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    $instance = new static($configuration, $plugin_id, $plugin_definition);
    $instance->currentUser = $container->get('current_user');

    return $instance;
  }

  /**
   * {@inheritdoc}
   */
  public function build() {
    $build['content'] = [
      '#markup' => $this->currentUser->getAccountName(),
    ];

    return $build;
  }
}
```

В качестве зависимости в данном примере внедрен сервис `current_user`.

> [!NOTE]
> Обратите внимание, что при инициализации объекта передаются все необходимые родительскому классу (`BlockBase`) аргументы. У некоторых блоков родители также могут запрашивать сервисы, и вам необходимо их также передавать. Для этого изучите `__construct()` родительского объекта плагина.

## Dependency Injection в формах

При работе с [формами](../form-api/form-api.md), Dependency Injection очень похож на тот что у плагинов, только без аргументов с данными для плагинов.

Класс формы, в котором необходимо использовать DI, должен реализовывать интерфейс `Drupal\Core\DependencyInjection\ContainerInjectionInterface`.

Он также добавляет необходимость описать статический метод `create()`, только, в отличие от DI плагинов, он принимает один единственный аргумент `$container`.

По умолчанию, если вы наследуетесь от `FormBase`, то там уже прописана данная реализация, а также пустой вызов `create()`. Таким образом, вам нет необходимости указывать интерфейс вашей форме, если она наследуется от `FormBase`.

**Пример формы без DI:**

```php
<?php

namespace Drupal\foo\Form;

use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;

/**
 * Provides a Foo form.
 */
class FooForm extends FormBase {

  /**
   * {@inheritdoc}
   */
  public function getFormId() {
    return 'foo';
  }

  /**
   * {@inheritdoc}
   */
  public function buildForm(array $form, FormStateInterface $form_state) {

    $form['message'] = [
      '#type' => 'textarea',
      '#title' => $this->t('Message'),
      '#required' => TRUE,
    ];

    $form['actions'] = [
      '#type' => 'actions',
    ];
    $form['actions']['submit'] = [
      '#type' => 'submit',
      '#value' => $this->t('Send'),
    ];

    return $form;
  }

  /**
   * {@inheritdoc}
   */
  public function validateForm(array &$form, FormStateInterface $form_state) {
    if (mb_strlen($form_state->getValue('message')) < 10) {
      $form_state->setErrorByName('name', $this->t('Message should be at least 10 characters.'));
    }
  }

  /**
   * {@inheritdoc}
   */
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $this->messenger()->addStatus($this->t('The message has been sent.'));
    $form_state->setRedirect('<front>');
  }

}
```

**Пример формы с DI:**

```php
<?php

namespace Drupal\foo\Form;

use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Session\AccountProxyInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * Provides a Foo form.
 */
class FooForm extends FormBase {

  /**
   * The current user.
   *
   * @var \Drupal\Core\Session\AccountProxyInterface
   */
  protected $currentUser;

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    $instance = parent::create($container);
    $instance->currentUser = $container->get('current_user');

    return $instance;
  }


  /**
   * {@inheritdoc}
   */
  public function getFormId() {
    return 'foo';
  }

  /**
   * {@inheritdoc}
   */
  public function buildForm(array $form, FormStateInterface $form_state) {

    $form['message'] = [
      '#type' => 'textarea',
      '#title' => $this->t('Message'),
      '#required' => TRUE,
      '#default_value' => $this->currentUser->getAccountName(),
    ];

    $form['actions'] = [
      '#type' => 'actions',
    ];
    $form['actions']['submit'] = [
      '#type' => 'submit',
      '#value' => $this->t('Send'),
    ];

    return $form;
  }

  /**
   * {@inheritdoc}
   */
  public function validateForm(array &$form, FormStateInterface $form_state) {
    if (mb_strlen($form_state->getValue('message')) < 10) {
      $form_state->setErrorByName('name', $this->t('Message should be at least 10 characters.'));
    }
  }

  /**
   * {@inheritdoc}
   */
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $this->messenger()->addStatus($this->t('The message has been sent.'));
    $form_state->setRedirect('<front>');
  }

}
```

## Dependency Injection в деривативах (Derivatives)

Деривативы являются частью системы плагинов, поэтому у них также немного отличается DI, но похож на предыдущие.

Деривативы, которым требуется Dependency Injection, должны реализовывать интерфейс `Drupal\Core\Plugin\Discovery\ContainerDeriverInterface`.

Он также требует создание статического метода `create()`, и принимает два аргумента:

- `$container`: Service Container, как и во всех остальных случаях.
- `$base_plugin_id`: Идентификатор плагина, для которого описывается деривативы.

**Пример деривативы без DI:**

```php
<?php

namespace Drupal\foo\Plugin\Derivative;

use Drupal\Component\Plugin\Derivative\DeriverBase;

/**
 * Provides a Foo deriver.
 */
class FooDeriver extends DeriverBase {

  /**
   * {@inheritdoc}
   */
  public function getDerivativeDefinitions($base_plugin_definition) {
    $this->derivatives['custom'] = $base_plugin_definition;

    return $this->derivatives;
  }
}
```

**Пример деривативы с DI:**

```php
<?php

namespace Drupal\foo\Plugin\Derivative;

use Drupal\Component\Plugin\Derivative\DeriverBase;
use Drupal\Core\Entity\EntityTypeBundleInfoInterface;
use Drupal\Core\Plugin\Discovery\ContainerDeriverInterface;
use Drupal\Core\StringTranslation\TranslatableMarkup;
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * Provides a Foo deriver.
 */
class FooDeriver extends DeriverBase implements ContainerDeriverInterface {

  /**
   * The entity type bundle info.
   *
   * @var \Drupal\Core\Entity\EntityTypeBundleInfoInterface
   */
  protected $entityTypeBundleInfo;

  /**
   * QueueWorkerDeriver constructor.
   *
   * @param string $base_plugin_id
   *   The base plugin id.
   * @param \Drupal\Core\Entity\EntityTypeBundleInfoInterface $entity_type_bundle_info
   *   The entity type bundle info.
   */
  public function __construct(EntityTypeBundleInfoInterface $entity_type_bundle_info) {
    $this->entityTypeBundleInfo = $entity_type_bundle_info;
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container, $base_plugin_id) {
    return new static(
      $container->get('entity_type.bundle.info')
    );
  }

  /**
   * {@inheritdoc}
   */
  public function getDerivativeDefinitions($base_plugin_definition) {
    $node_types = $this->entityTypeBundleInfo->getBundleInfo('node');

    foreach ($node_types as $type => $type_info) {
      $this->derivatives[$type] = $base_plugin_definition;

      $admin_label = new TranslatableMarkup('Last content for content type "@node_type_label"', [
        '@node_type_label' => $type_info['label'],
      ]);
      $this->derivatives[$type]['admin_label'] = $admin_label;
    }

    return $this->derivatives;
  }
}
```

## Dependency Injection в хендлерах сущностей

При работе с хендлерами сущности, вы также можете использовать Dependency Injection.

В данном случае классу хендлера необходимо реализовывать интерфейс `Drupal\Core\Entity\EntityHandlerInterface`. Он также похож по своей работе на предыдущие способы DI, но использует метод `createInstance()`, вместо `create()`, так как `create()` используется для создания сущностей.

Данный метод принимает два аргумента:

- `$container`: Service Container, как и везде.
- `$entity_type`: Объект типа сущности, для которого данный хендлер вызван.

**Пример хендлера без DI:**

```php
<?php

namespace Drupal\foo\Handler;

/**
 * Provides a Foo handler.
 */
class FooHandler {

  /**
   * Returns greetings for user.
   *
   * @return string
   *   The greetings.
   */
  public function hello() {
    return 'Hello, user!';
  }

}
```

**Пример хендлера с DI:**

```php
<?php

namespace Drupal\foo\Handler;

use Drupal\Component\Render\FormattableMarkup;
use Drupal\Core\Entity\EntityHandlerInterface;
use Drupal\Core\Entity\EntityTypeInterface;
use Drupal\Core\Session\AccountProxyInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * Provides a Foo handler.
 */
class FooHandler implements EntityHandlerInterface {

  /**
   * The current user.
   *
   * @var \Drupal\Core\Session\AccountProxyInterface
   */
  protected $currentUser;

  /**
   * Constructs a new FooForm object.
   *
   * @param \Drupal\Core\Session\AccountProxyInterface $account_proxy
   *   The account proxy.
   */
  public function __construct(AccountProxyInterface $account_proxy) {
    $this->currentUser = $account_proxy;
  }

  /**
   * {@inheritdoc}
   */
  public static function createInstance(ContainerInterface $container, EntityTypeInterface $entity_type) {
    return new static(
      $container->get('current_user')
    );
  }

  /**
   * Returns greetings for user.
   *
   * @return string
   *   The greetings.
   */
  public function hello() {
    return new FormattableMarkup('Hello, @username!', [
      '@username' => $this->currentUser->getAccountName(),
    ]);
  }
}
```
