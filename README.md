# Настройка удаленной отладки PHP (XDebug) на Visual Studio Code (VSC)

>Процесс первоначальной настройки локального (XAMPP) XDebug на VSC пропущен, такие мануалы есть в сети. Мне понадобилось к уже настроенному и рабочему VSC подключить отладку удаленного сайта.

Итак, дано: на локале винда с установленным VSC и удаленный Ubuntu-сервер (нулевый на виртуальной машине). 

## Настройка сервера (у меня Ubuntu 18.04)

### Устанавливаем xdebug

До установки xdebug, нам сперва необходимо поставить некоторые другие пакеты. Установим пакет php-dev, который позволит нам компилировать динамические расширения для PHP. Как раз таким расширением является xdebug.

``apt-get install php-dev``

*В мануале, на основе которого я делал настройку, написно ``php5-dev``, но во-первых у меня уже php7, а во-вторых менеджер пакетов сам найдет пакет по-умолчанию для вашей версии Ubuntu (Для Ubuntu 18 это php7).*

Теперь установим PEAR, который также включает PECL. Согласно их вебсайту, PECL — это репозиторий для расширений PHP, который предоставляет каталог всех известных расширений, а также услуги хостинга для скачивания и разработки PHP-расширений. Xdebug является одним из таких расширений.

``apt-get install php-pear``

Теперь мы можем установить/скомпилировать xdebug, используя PECL. Выполним следующую команду:

``pecl install Xdebug``

После успешной установки, в конце выдаст примерно следующее, запомните путь куда установлено расширение, нам он потом понадобится:
```
Build process completed successfully
Installing '/usr/lib/php/20170718/xdebug.so'
```

### Изменяем php.ini 
Мой файл лежит тут: ``/etc/php/7.2/apache2``, учитывайте версию php.

Добавим следующий код в конец php.ini:
```ini
[xdebug]
;сюда вписываем расширение с полным путем, которое мы получили после установки Xdebug
zend_extension="/usr/lib/php/20170718/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_handler=dbgp
xdebug.remote_mode=req
xdebug.remote_port=9000
;тут ip-адрес компа, с которого будете запускать отладку. 
;В виртуальной машине это не проблема: запускайте сеть в режиме моста и получите комп в локальной сети. 
;Если нужно отладка на удаленном сервере, а ваш комп за натом, 
;то нужно будет с помощью SSH пробросить порт 9000 на ваш комп, а тут вписать 127.0.0.1
xdebug.remote_host=[ip-адрес компа, с которого будете запускать отладку]
;эту строку я оставил, хотя VSC ее вроде не посылает
xdebug.idekey="vsc-xdebug"
;а эти отличаются от установок по-умолчанию
xdebug.remote_autostart=1
xdebug.remote_connect_back = 1
```

Перезагружаем Apache
```
service apache2 restart
```

## Настраиваем VSC

Открываем настройку отладчика (напоминаю, что он у меня уже настроен): шестеренка в окне "Отладка". Откроется файл ``launch.json``. В нем в секцию ``"configurations"`` добавляем настройки для нашего сервера:

```json
"configurations": [
    /*это настройки стандартного отладчика*/
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9000
    },
    /*этот блок мы добавляем*/
    {
      "name": "remote XDebug",
      "type": "php",
      "request": "launch",
      "port": 9000,
      "pathMappings": {
        "/var/www/html": "${workspaceRoot}/mysite"
      }
    },
...
```

Главное тут ``"pathMappings"``: мы должны указать VSC какой локальный каталог соответствует удаленному. 

``/var/www/html`` - это каталог на удаленном севрере (какой был по-умолчанию, такой и использую)

``${workspaceRoot}/mysite`` - это локальный каталог VSC, ``${workspaceRoot}`` указывать обязательно, а ``/mysite``, если у вас в VSC в текущем каталоге несколько проектов.

Все готово, теперь для отладки на удаленном сервере нужно в выпадающем списке выбрать ``remote XDebug``, проставить точки останова и запусить отладчик. При открытии сайта в браузере VSC подключится к отладке и остановится на точке останова.

***
* [Установка XDebug на сервер, читать до настройки NetBeans](https://sohabr.net/habr/post/243073/)
* [Настройка удаленной отладки в VSC (по английски)](https://aaroneaton.blog/vs-code-remote-xdebug/)
