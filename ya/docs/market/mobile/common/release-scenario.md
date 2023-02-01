## Сценарий релиза

1. **Последний день рабочей недели, 18:00.** Дежурный тестировщик запускает релизный пайплайн
   - Пишет сообщение о старте релиза в telegram чат **Релизы мобилок Маркета** ([ссылка](https://t.me/joinchat/4eW78VqzcmA4YzYy))
   - Запускает пайплайн в ЦУМе ([android](https://tsum.yandex-team.ru/pipe/projects/mobile-blue/release/new/blue-market-android-gitflow-release), [ios](https://tsum.yandex-team.ru/pipe/projects/mobile-blue/release/new/blue-market-ios-gitflow-release))

2. **Последний день рабочей недели, 18:00 - 23:59.** Дежурный разработчик проверяет, что сборка релиза прошла успешно
   - Дожидается завершения кубика сборки в пайплайне
   - Если сборка не прошла, исправляет ее и перезапускает пайплайн

3. **Первый день рабочей недели, утро.** Дежурный тестировщик формирует тест-ран на основе диффектора
   - Получает результаты работы диффектора (подробности у кого:dmpolyakov)
   - Создает тест-ран и скипает в нем кейсы на основе диффектора

4. **Первый, второй день рабочей недели.** Команда тестирования проводит регресс приложения
   - Тестировщик проходит тест-кейсы, за которые отвечает его контур
   - Сообщает дежурному разработчику о найденных ошибках

5. **Первый, второй день рабочей недели.** Дежурный разработчик подготавливает сборку с фиксами найденных ошибок
   - Исправляет найденные ошибки либо согласует их неисправление
   - Уведомляет дежурного тестировщика о готовых фиксах

6. **Второй день рабочей недели.** Дежурный тестировщик проверяет, что тест-ран пройден, все ошибки исправлены

7. **Второй день рабочей недели.** Дежурный разработчик удостоверяется, что релизный пайплайн завершился

7. **Второй день рабочей недели.** Дежурный разработчик (в случае iOS) или дежурный публикатор раскатывают релиз в PlayMarket/AppStore ([Android](https://docs.yandex-team.ru/market-mobile/common/android-release-distribution), [iOS](https://docs.yandex-team.ru/market-mobile/common/ios-release-distribution))

##### Выше представлен сокращенный сценарий релиза приложения. Если информации будет недостаточно, можно почитать [подробный](https://wiki.yandex-team.ru/market/beru/qa-beru/esli-ty-dezhurnyjj/). 