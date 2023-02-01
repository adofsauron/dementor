# Оптимизация изображений

Для кратного ускорения загрузки страницы нужно уменьшать вес изображений, так как они занимают основную часть размера страницы, это позволит значительно улучшить пользовательский опыт взаимодействия с продуктом.

## Сжатие бывает без потерь и с потерями (в качестве)

По сути все форматы так или иначе и есть сжатие без потерь как минимум. Так же изображение может содержать различную метаинформацию, которая не нужна на странице и скорее всего даже не читается ни в каком виде.

Будем считать что без потерь уже сжато, если у вас не .tiff. Мета информация занимает в среднем до 10% от веса картинки

Теперь про "сжатие изображения с потерями", для каждого формата свои алгоритмы, по сути почти только этим и отличаются разные медиа форматы.
Потери чаще всего измеряются в процентах от качества, где 100% это максимум, а 1% минимум, но фактически внутри идут диапазоны, для jpeg их больше, для png меньше. Поэтому в некоторых программах вместо процентов шкалы от 1 до 5 или как у PS от 1 до 12. Например,  для png может не быть никаких различий между 90 и 70%.

### Используемый диапазон, сжатия в диапазоне 40%-90%
* 40% - 60% - при сравнении с оригиналом можно заметить различия, если искать
* 60% - 80% - иногда можно заметить если знать что конкретно искать
* 80% - 90% - только в некоторых ситуациях можно увидеть минимальные различия

### Про вес
* orig.jpeg - 1.169 мб
без мета инфы - 1.168 мб (-0.1%)
90% - 0.184 мб (-84.3%)
70% - 0.097 мб (-91.7%)
50% - 0.068 мб (-94.1%)

* orig.png - 1.548 мб
без мета инфы - 1.358 мб (-12.3%)
70% - 0.314 мб (-79.7%)
40% - 0.204 мб (-86.8%)

## Чем и как сжимать?

* Для дизайнеров подойдут почти все инструменты, при сохранении есть указание качества

* `JPG` нужно сделать прогрессивным и посмотреть можно ли понизить качество, не потеряв сильно во внешнем виде

* `PNG` стоит сделать восьмибитным, если позволяет палитра. Если есть гигантская картинка (типа как на главной у нас) то оставлять её в png точно не стоит. [tinypng](https://tinypng.com/) - переводит png32 в png8

* `SVG` оптимизируем через svgo. Команда для запуска `npm run svg-opt`

* Картинки ещё можно оптимизировать через другие форматы (webp, avif) и подложить через [https://developer.mozilla.org/ru/docs/Web/HTML/Element/picture](https://developer.mozilla.org/ru/docs/Web/HTML/Element/picture) или задать разные разрешения в зависимости от ширины устройства через srcset

* **Imageoptim** - приложение для мака imageoptim, либо консольный imagemagick. Самый популярный и бесплатный инструмент, в котором все учтено. Он очень простой и по умолчанию сжимает без потерь. Он заменяет оригинальное изображение, поэтому сначала экспортируем в папку все что нужно, потом просто все выделяем и переносим в окно программы.
1) Качаем [https://imageoptim.com/mac](https://imageoptim.com/mac)
2) Включаем в настройках "минимизацию с потерями" и ставим все на **70%**. (Оптимальным сжатием для веб считается 60-80, давайте использовать 70%)
3) Все изображения для сайтов сначала закидываем в окно программы
4) В итоге они будут весть в 10 раз меньше, а по качеству так же