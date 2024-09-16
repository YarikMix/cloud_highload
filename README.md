# Проектирование высоконагруженного облачного хранилища

Курсовая работа в рамках 3-го семестра программы по Веб-разработке Образовательного центра VK x МГТУ им.Н.Э.Баумана (ex. "Технопарк") по дисциплине "Проектирование высоконагруженных систем" ставит перед собой цель детального проектирования высоконагруженной системы. 

Темой работы является проектирование сервиса, схожего по функционалу с Облаком mail.

**Вся информация была взята из открытых источников**

## Содержание

- ### [1. Тема, целевая аудитория и функционал](#1_part)
- ### [2. Расчет нагрузки](#2_part)

---

## 1. Тема, целевая аудитория и функционал <a name="1_part"></a>

### Тема

Облако mail - это облачное хранилище данных российской компании VK. Позволяет хранить музыку, видео, изображения и другие файлы в облаке и синхронизировать данные на компьютерах, смартфонах или планшетах, а также делиться ими с другими пользователями Интернета.

### Целевая аудитория

Согласно информации с [hi-tech.mail.ru](https://hi-tech.mail.ru/news/102223-raskryit-obem-polzovatelskih-dannyih-v-oblake-mailru/), [similarweb](https://www.similarweb.com/ru/website/cloud.mail.ru/):

- 1 387 710 124 посещений в месяц
- 4 625 000 активных пользователей в день
- В 2023 году общий объем хранилища — более 600 петабайт данных
- За 2023 год пользователи в общей сложности загрузили более 30 млрд различных файлов
- Самая многочисленная возрастная группа посетителей: 25 - 34 лет
- В среднем, пользователи проводят на ресурсе более 5 минут за сессию
- Среднее количество страниц за визит: 4

Продукт рассчитан на СНГ рынок, поэтому рассмотрим региональную аудиторию подробнее.

#### Демография аудитории Облака mail.ru

![Демография аудитории Облака mail](images/1.png)

#### Количество пользователей Облака mail.ru по странам

| Страна    | Процент пользователей от общего числа |
|-----------|---------------------------------------|
| Россия    | 85.88%                                |
| Беларусь  | 7.06%                                 |
| Казахстан | 3.38%                                 |
| Армения   | 0.97%                                 |
| Турция    | 0.95%                                 |

![Количество пользователей Облака mail по странам](images/2.png)

### Основной функционал

1. Загрузка, скачка и удаление файлов
2. Создание папок для хранения файлов (любого уровня вложенности)
3. Регистрация и аутентификация пользователей
4. Просмотр файлов (фото и документов)
5. Редактирование документов
6. Шеринг файлов и папок

---

## 2. Расчет нагрузки <a name="2_part"></a>

* MAU: 35 000 000
* DAU: 4 625 000
* Среднее время на сайте: 00:04:35
* Среднее количество посещенных страниц за сессию: 3.86

### Объем хранилища и типы файлов
В стандартном тарифном плане объем предоставлемого хранилища 
* 8гб для обычных пользователей 
* 1тб для платных пользоватей

Предположим что у Облака Mail примерно 50млн пользоватей. Допустим, что 150к из них - платные
На основании собственного опыта и опроса знакомых диск в среднем заполнен на 2/3. 
Примерно по пропорции можно закладывать 11GB в среднем на пользователя.

**Общий размер всего хранилища**: ```11GB * 50млн ~ 524PiB.``` - сходится с [hi-tech.mail.ru](https://hi-tech.mail.ru/news/102223-raskryit-obem-polzovatelskih-dannyih-v-oblake-mailru/)

По опросам пользователей, 80% используют диск только для видео и фото. 
Для простоты будем счиать что остальные файлы это документы. 
Поскольку основной алгоритм использования это загрузка на диск фото и видео с телефона для долгострочного хранения. 
Соотношение фото к видео у меня и знакомых на телефоне примерное 8:1, будем брать такую статистику в расчетах.

| Тип файла      | Процент от всех файлов в штуках |
|----------------|---------------------------------|
| Фото           | 71 %                            |
| Видео          | 9 %                             |
| Документы      | 20 %                            |

Оценим средний размер PNG картинки:

```( 1920px * 1080 px * 4b + 33b (header) ) * 0.66 (среднее сжатие DEFLATE) ~= 5.12 MiB``` 

Также есть информация что средний размер фотографии на iPhone в 2023 году был 2-8mb, что примерно бьется с расчетами.
Продолжая ориентироватся что большая часть /фото видео снимается на iPhone возьмем средний размер 1080p/30fps видео снятого на последний - это ```60MiB/min```, средняя продолжительность видео на самом попярном видеохостинге ```11.7min```. 
Исходя из этого средний обьем видео:```60 * 11.7 = 700MiB```
Размер документа docx/pdf   ```10kB + 3kB/page```, будем брать среднем 30KiB.
Итого:

| Тип файла      | Средний размер  |
|----------------|-----------------|
| Фото           | 5.12MiB         |
| Видео          | 700MiB          |
| Документы      | 30KiB           |

### Среднее количество действий пользователя день:

Приведем статистику которая есть от похожего сервиса - [Dropbox](https://gitnux.org/dropbox-statistics/)
- 1.2bln загружаемых файлом в день при DAU 4mln, в пересчете 300 файлов в день на пользователя
- 100k создаваемых ссылок на папки, файлы при DAU 4mln в пересчете 0.6 ссылок в день на пользователя

Просмотр файла также будем считать за скачивание, так как пользователь пользователь просматривая картинку/видео/документ его скачивает.
Пусть около 5% посещений происходит с запросом авторизации. 
Среднее количество посещаемых страниц равно 4, а поскольку на диске только папки являются страницами будем брать это как просмотр содержимого папки. 
Будем считать что просматривают пользователи в 2 раза больше чем загружают, к-во удалений тоже возьмем сравнимое с загрузкой.

| Действие                    | Среднее количество дейстивий в день на пользователя |
|-----------------------------|-----------------------------------------------------|
| Аутентификация              | 0.05                                                |
| Аутентификация по куке      | 1                                                   |
| Загрузка на диск            | 300                                                 |
| Просмотр/скачивание с диска | 600                                                 |
| Удаление                    | 100                                                 |
| Создание ссылки             | 0.6                                                 |
| Просмотр папки              | 4                                                   |


### RPS

DAU = 4.625 M

RPS = количесво действий на пользователя * DAU / 86 400

Файлы будут загрузаться чанками по 10 MiB с клиента, чанкование возможно в основном только для видео в нашем случае это ```700/10 = 70``` запросов на загрузку видео. 

Количество загрузок видео ```300*9% =27```

```RPS = DAU * (0.05 + 300*91% + 27 * 70 + 600 + 100 + 0.6 + 5 + 5) / 86400 = 2873.65 * DAU /86400 ~ 153_800```

Общее
- **RPS** ~ 153_800
- **Пиковый RPS (x2.5)** ~ 384_500

**RPS по типам запросов**

| Запрос                            | RPS    | Пиковый RPS |
|-----------------------------------|--------|-------------|
| Аутентификация                    | 2.7    | 6.8         |
| Аутентификация по куке            | 54     | 135         |
| Загрузка на диск                  | 16_000 | 40_000      |
| Скачивание с диска                | 32_000 | 80_000      |
| Создание сслыки/изменение доступа | 32     | 80          |
| Просмотр папки                    | 216    | 540         |


### Расчет сетевого траффика

DAU = 4.625 M

С помощью инструментов разработчика в браузере можно посмотреть средник трафик на каждое действие пользователя. 
Все запросы не связанные с файлами будем считать размером в 5KiB. 

**Общий трафик**

Трафик будем вычислять по формуле: ```трафик_за_действие * RPS_на_действие```

```Трафик в секунду = (2.7 + 16_000 + 32_000 + 32 + 216) * 5KiB + 16_000 (20% * 30KiB + 9% * 700MiB + 71% * 5.12MiB) ~ 1 Tbit/sec```

**Скачивание**

Скачивание будет производится с s3 так что посчитаем его отдельно.

``` Скачивание в секунду = 32_000 * (20% * 30KiB + 9% * 700MiB + 71% * 5.12MiB) ~ 2 Tbit/sec ```

| Запрос                            | Трафик/s |
|-----------------------------------|----------|
| Аутентификация                    | 13 KiB   |
| Аутентификация по куке            | 267 KiB  |
| Загрузка на диск                  | 1 Tbit   |
| Скачивание с диска                | 2 Tbit   |
| Создание сслыки/изменение доступа | 154 KiB  |
| Просмотр папки                    | 1 Mbit   |

### Технические метрики:

| Метрика                  | Значение     |
|--------------------------|--------------|
| Общий размер хранилища   | 524PiB       |
| Трафик в секунду         | 1 Tbit/s     |
| Пиковый трафик с секунду | 2.5 Tbit/s   |
| RPS                      | 153_800      |
| Пиковый RPS              | 384_500      |


## Используемые источники

* https://hi-tech.mail.ru/news/102223-raskryit-obem-polzovatelskih-dannyih-v-oblake-mailru/
* https://www.similarweb.com/ru/website/cloud.mail.ru/
* https://gitnux.org/dropbox-statistics/
