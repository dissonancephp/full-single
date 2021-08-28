# Symbiotic Web (MVP, Single-file)
### Описание
**Фреймфорк создан с целью упростить интеграцию независимых небольших приложений в другие CMS и фреймворки, а также для расширения функциональности пакетов для композера.** 


Идеология - отдельная экосистема небольших приложений для совместной работы вместе с другими фреймворками и удобной интеграции дополнительного функционала.

Есть много пакетов и отдельно написанных приложений,
 которые поставляют полезный функционал, имеют свою бизнес логику и иногда даже имеют свой отдельный веб интерфейс.
  
  В ларавель пакеты, в симфони бандлы, в различных CMS в виде плагинов и дополнений, и у всех своя реализация роутинга, событий, кеширования и т.д. Взять пакет, написанный для ларавель, и интегрировать его в другой фреймворк или CMS, в большинсте случаев будет проблематично, а в некоторых нереально из-за определенных зависимостей от фреймворка.
  

Самим разработчикам приложений приходится писать адаптацию под каждый фреймворк и CMS, что создает много проблем и не покрывает все известные экосистемы.
 
Также такие приложения приходится интегрировать в систему:
1. Настраивать ACL
2. Интегрировать неоходимые скрипты админку и на фронт
3. Создавать обработчики запросов и структуру в бд
4. Делать связку с файловой системой
5. Делать сохранение настроек и конфигурации

Примеров таких приложений много:
- одностраничные приложения
- текстовые редакторы и их плагины с несколькими уровнями зависимости (плагин для плагина)
- обработчики картинок
- различные оптимизаторы и компрессоры 
- приложения для работы с файлами и базами данных
- чат боты
- административные, аналитические интрументы
- лендинги и другие микро приложения
- .....


Фреймворк оптимизирован для работы с большим количеством приложений, а также для работы в качетсве подсистемы для основного фреймворка. 

Каждое приложение является композер пакетом, 
с дополнительным описанием прямо в файле composer.json.

## Характеристики
- PSR дружественный
- Мало зависимостей (только PSR интерфейсы и PSR-7 имплементация)
- Небольшой вес (370 кб дев версия и формтированием и коментариями, продакт 170 кб)
- Оптимизирован для работы в симбиозе с другими фреймворками
- Многоуровневая система контейнеров (Ядро<-Приложение<-Плагин), с доступом к контейнеру родителю.
- Виртуальная файловая система (прокидывание статики прямо из папки пакета)
- Всем знакомое апи контейнера
- Шаблонизатор Blade(урезанный и кривой пока), + возможность прокинуть свой шаблонизатор.
- Никаких сборщиков статики (Каждый пакет должен иметь уже скомпилировынные файлы).
- Отложенный роутинг (грузятся только роуты запрошенного приложения, определяется по префиксу-поселению).
- Возможность расширять конревые сервисы (Бутстраперы и сервисы).
- У каждого приложения свой сервис контейнер и сервисы.
- Поддержка кеша (PSR-16 Simple Cache) + Кешируемый cервис контейнер.
- Для тех, кто будет тестировать: отличное приключение (почти реверс), абсолютно без документации и все одном файле!!!)))

## Установка
```
composer require symbiotic/full-single 
```

## Запуск

Фреймворк подключается из композера прямо в ваш index.php.
 
 Если вы используете уже фреймворк, то необходимо включить режим симбиоза в конфиге
```php
$config['symbiotic'] = true;
```
##### Инициализация
```php

$basePath = dirname(__DIR__);// корневая папка проекта
include_once $basePath. '/vendor/autoload.php';

$config  = [
    'debug' => true,
    'symbiotic' => true, // Режим симбиоза, если включен и фреймворк не найдет обработчик,
    // то он ничего не вернет и основной фреймворк смодет сам обработать запрос
    'default_host' => 'localhost',// для консоли , но ее пока нет
    'uri_prefix' => 'symbiotic', // Префикс в котором работет фреймворк, если пустой то работае от корня
    'base_path' => $basePath, // базовая папка проекта
    'assets_prefix' => '/assets',
    'storage_path' =>  $basePath . '/storage', // Если убрать то кеш отключится
    'packages_paths' => [
        $basePath . '/vendor', // Папка для приложений
    ],
    'bootstrappers' => [
              \Symbiotic\Develop\Bootstrap\DebugBootstrap::class,/// debug only
              \Symbiotic\Core\Bootstrap\EventBootstrap::class,
              \Symbiotic\SimpleCacheFilesystem\Bootstrap::class,
              \Symbiotic\Packages\PackagesLoaderFilesystemBootstrap::class,
              \Symbiotic\Packages\PackagesBootstrap::class,
              \Symbiotic\Packages\ResourcesBootstrap::class,
              \Symbiotic\Apps\Bootstrap::class,
              \Symbiotic\Http\Bootstrap::class,
              \Symbiotic\Http\Kernel\Bootstrap::class,
              \Symbiotic\View\Blade\Bootstrap::class,
    ],
    'providers' => [
        \Symbiotic\Http\Cookie\CookiesProvider::class,
        \Symbiotic\Routing\SettlementsRoutingProvider::class,
        \Symbiotic\Session\NativeProvider::class,
    ],
    'providers_exclude' => [
        \Symbiotic\Routing\Provider::class,
    ]
];

// Базовая постройка контейнера
$core = new \Symbiotic\Core\Core($config);
// Или через билдер с кешем
$cache = new Symbiotic\SimpleCacheFilesystem\SimpleCache($basePath . '/storage/cache/core');
$core = (new \Symbiotic\Core\ContainerBuilder($cache))
    ->buildCore($config);

// Запуск 
$core->run();
// Дальше может идти код инициализации и отработки другого фреймворка...

```
##### Схема описания расширения и приложения для фреймворка
Берем стандартный пакет композера и добавляем:
```json
{
  "name": "vendor/package",
  "require": {
   // ...
  },
  "autoload": {
   ///
  
  },
// Добавляем описание пакета для фреймворка
  "extra": {
    "symbiotic": {
          "id": "wso.my_package_id", // ID пакета формируется на сайте фреймворка, но можно локально любой ставить
           // Описание приложения, пакет может и не иметь секцию приложения, а быть лишь расширением
          "app": { 
                "id": "my_package_id", // Id приложения, указывается без префикса родительского приложения
                "parent_app": "wso", // ID родительсского приложения, если приложение плагин 
                "name": "WSO Users exporter", // Имя приложения, используется в списке приложений и меню
                "routing": "\\\\MyVendor\\\\MySuperPackage\\\\Routing", // Класс роутинга, не обязательно
                "controllers_namespace": "\\\\Symbiotic\\\\Develop\\\\Controllers", // Базовый неймспейс для контроллеров, не обязательно
                "version": "1.0.0", // Версия, не обязательно, плагины могут проверять и подстаиваться под изменения
                "providers": [ // Провайдеры приложения, не обязательно
                  "MyVendor\\\\MySuperPackage\\\\Providers\\\\AppProvider"
                ],
                // Не обязательно! Наследник от \\Symbiotic\\App\\Application
                "app_class": "MyVendor\\\\MySuperPackage\\\\MyAppContainer" 
          },
    
          // Расширения ядра фреймворка, не обязательно
          "bootstrappers":[
             "MyVendor\\\\MySuperPackage\\\\CoreBootstrap" // Загрузчики
          ],
          "providers" : [
             "MyVendor\\\\MySuperPackage\\\\MyDbProvider" // Провайдеры
          ],
          "providers_exclude" : [
              // Исключение провайдеров из загрузки
              // Например при двух пакетах одной библиотеки позволяет исключить не нужную
          ]     
    }
  }
}

```

#### Пример пакета только со статикой
Всего пару строк:
```json
{
  "name": "vendor/package",
  "require": {
   // ...
  },
  "autoload": {
   // ...
  },
  "extra": {
    "symbiotic": {
          "id": "my_super_theme_2",
          // Можно указать что то одно или все вместе
          "public_path": "assets", // Папка со статикой, относительно корня пакета 
          "resources_path": "my_resources", // Папка c шаблонами и другими файлами, не доступны через http
          // можно прокинуть в веб при необходимости через специальный объект доступа к ресурсам
    }
  }
}

```


#### Пример пакета приложения
При конфигурации приложения можно не указывать пути для статики и ресурсов, тогда будут определены пути по умолчанию:

- public_path = assets
- resources_path = resources

Шаблоны всегда дожны лежать в директории /view/ в папке ресурсов!
```json
{
  "name": "vendor/package",
  "require": {
   // ...
  },
  "autoload": {
   // ...
  },
  "extra": {
    "symbiotic": {
           "app": { 
                "id": "my_package_id", // Id приложения
                "routing": "\\\\MyVendor\\\\MySuperPackage\\\\Routing",
                "controllers_namespace": "\\\\Symbiotic\\\\Develop\\\\Controllers"
          },
    }
  }
}

```

## Примерная структура пакета
Четкой обязательной структуры нет, можно использовать любую. 
```text
vendor/
   -/my_vendor
      -/my_package_name
           -/assets          - Статика
                -/js
                -/css
                -/...
           -/resources       - Ресурсы
                -/views
                -/...
           -/src             - Ваш пакет
               -/Http
                   -/Cоntrollers
                   -/...
               -/ ...
               -/Routing.php
          -/composer.json
```

При необходимости можно поселить все классы для приложения фреймворка в подпапку src/Symbiotic. Так не будет путаницы с функционалом вашего пакета.

```text
vendor/
   -/my_vendor
      -/my_package_name
           -/symbiotic
                   -/assets          - Статика
                        -/js
                        -/css
                        -/...
                   -/resources       - Ресурсы
                        -/views
                        -/...
           -/src                     - Ваш пакет
               -/Symbiotic
                       -/Http
                           -/Cоntrollers
                           -/...
                       -/Routing.php
              -/Ваши папки и файлы ...
              
          -/composer.json
```


