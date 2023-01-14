[profiler](https://xdebug.org/docs/profiler)

Профайлинг с Xdebug не лучшая идея, т.к. он накладывает большой оверхеад, но если очень нужнно то можно воспользоватся.

```ini
xdebug.profiler_enable=1
```
Будет профилировать всё

Если нужно профайлить только конкретные запросы:
```ini
xdebug.profiler_enable=0
xdebug.profiler_enable_trigger=1
xdebug.profiler_enable_trigger_value=<profiler_triger>
```

`<profiler_triger>` можно передать в GET/POST параметре, или поставить [Экстеншен для firefox](https://addons.mozilla.org/en-GB/firefox/addon/xdebug-helper-for-firefox/)

## Результаты

Результаты сохраняются в `/tmp`, файлы называются `cachegrind.out.%p`

можно изменить в парметрах:
```ini
xdebug.profiler_output_dir=
xdebug.profiler_output_name=
```

Смотреть результаты можно в 

* Phpstorm: Tools—Analyze Xdebug Profile Snapshot
* [KCacheGrind](https://kcachegrind.github.io/html/Home.html)
