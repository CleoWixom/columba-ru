# Русская локализация и обновления upstream

Русский язык живёт в файлах, которых нет в upstream (`torlando-tech/columba`),
поэтому при слиянии обновлений они не затираются и не конфликтуют:

| Файл | Назначение |
|---|---|
| `app/src/main/res/values-ru/strings.xml` | Перевод всех строк из `values/strings.xml` |
| `app/src/main/res/xml/locales_config.xml` | Список языков для выбора в Android 13+ (Настройки → Язык приложения) |
| `app/lint.xml` | `MissingTranslation` понижен до warning, чтобы новая строка upstream без перевода не ломала релизную сборку |
| `scripts/l10n/check_ru.py` | Проверка полноты и корректности перевода |
| `.github/workflows/l10n-ru.yml` | Запуск проверки в CI |

Единственная правка файла upstream: строка `android:localeConfig="@xml/locales_config"`
в `AndroidManifest.xml` (в теге `<application>`). Если после слияния её нет, верните её.

## Как обновляться с upstream

```bash
git remote add upstream https://github.com/torlando-tech/columba   # один раз
git fetch upstream
git merge upstream/main
python3 scripts/l10n/check_ru.py
```

Скрипт выведет:

* **нет перевода** — upstream добавил строки. Он сразу печатает готовые английские
  `<string>`-блоки: скопируйте их в `values-ru/strings.xml` и переведите;
* **лишний ключ** — upstream удалил или переименовал строку: удалите/переименуйте её в `values-ru`;
* **плейсхолдеры** — в переводе потерян или изменён `%1$s` / `%2$d`;
* **не хватает форм plurals** — для русского нужны `one`, `few`, `many`, `other`.

Пока строка не переведена, приложение показывает её по-английски (откат на язык по умолчанию),
сборка не ломается. После перевода запустите скрипт ещё раз: должно быть `ошибок: 0`.

## Ограничение: захардкоженный текст

Часть интерфейса в upstream написана прямо в Kotlin (`Text("...")`, `contentDescription = "..."`,
Toast/Snackbar) и не использует `strings.xml`. Такой текст остаётся английским на любом языке.
Узнать масштаб: `python3 scripts/l10n/check_ru.py --hardcoded`.

Выносить его в ресурсы правкой `.kt`-файлов в форке нужно осознанно: каждая такая правка
конфликтует при каждом слиянии upstream. Предпочтительный путь — предложить вынос
строк в ресурсы в upstream, тогда новые строки придут уже в `values/strings.xml`.
