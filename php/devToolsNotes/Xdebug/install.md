1. Загрузить исходники [отсюда](https://xdebug.org/download)
2. Перейти в директорию с исходниками
3. Запустить:

    ```bash
    phpize
    ```

    или

    ```bash
    /usr/bin/php<version>
    ```
4. Запустить:

    ```bash
    ./configure --enable-xdebug
    ```

    или

    ```bash
    ./configure --enable-xdebug --with-php-config=/usr/bin/php-config<version>
    ```
5. Запустить:

    ```bash
    make clean
    ```
6. Запустить:

    ```bash
    make
    ```
7. Запустить:

    ```bash
    sudo make install
    ```
8. Создать `xdebug.ini` в `/etc/php/<version>/mods-available/`, содержимое:
    
    ```ini
    zend_extension=xdebug.so
    ```

    команда:
    
    ```bash
    echo "zend_extension=xdebug.so" > /etc/php/<version>/mods-available/xdebug.ini
    ```
9. Сделать симлинки:

    для fpm:

    ```bash
    sudo ln -sf /etc/php/<version>/mods-available/xdebug.ini /etc/php/<version>/fpm/conf.d/20-xdebug.ini
    ```

    для cli:

    ```bash
    sudo ln -sf /etc/php/<version>/mods-available/xdebug.ini /etc/php/<version>/cli/conf.d/20-xdebug.ini
    ```
10. Перезапустить fpm:

    ```bash
    sudo service php<version>-fpm restart
    ```

## Настройка

[Описание всех настроек](https://xdebug.org/docs/all_settings)

```ini
zend_extension=xdebug.so
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.idekey="php<version>-xdebug"
```

Если в `xdebug.remote_autostart` указать 0, то он не будет автоматически запускаться для дебага.

Тригерить запуск можно будет экстеншеном для бразуера или GET/POST параметром для fpm, для cli можно делать так:

```bash
php -d xdebug.remote_autostart=1 <script>
```

## Настройка в phpstorm

1. Run—Edit Configurations
2. **+**—PHP Remote Debug:

    ```
    Name: Xdebug
    Filter debug connetcion by IDE key: yes
    IDE key(session id): `php<version>-xdebug`
    ```

3. Добавить новый сервер:

    ```
    Host: адрес виртуального хоста
    ```    

## Экстеншен для браузера

[Экстеншен для firefox](https://addons.mozilla.org/en-GB/firefox/addon/xdebug-helper-for-firefox/)

Через него можно запускать дебаг (если не включен `remote_autostart=1`), запускать построение трейсов и профайлер (для них нужно будет настроить тригеры)
