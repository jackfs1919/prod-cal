Description
===========

Given module allows to use production calendars of different countries, it can be used in common with the standard
module "calendar.Calendar", because the main class "ProdCal" is inherited from it. The methods of class "ProdCal"
allow to calculate number of working days and days off based on holidays and transfers of working days.

pip install prod-cal


Гарантирую что проект будет работать на Python 2.7 и Windows 7, т. к. на этой конфигурации он разрабатывался.

Как собирать пакеты и выкладывать в PyPi я описывать не буду, есть достаточно подробные статьи на эту тему, скажу только что с этой задачей может справиться и новичок, так что если Вы подумывали сделать новый модуль, то не откладывайте это в долгий ящик в этом нет ничего сложного.

Главная цель данной статьи разобрать устройство данного модуля и наметить перспективы для его развития сообществом.

Чтобы не плодить календарей в моём календаре можно использовать все методы стандартного модуля calendar.Calendar.


Состав проекта


После установки проект будет доступен в C:\Python27\Lib\site-packages\prodcal, если вы устанавливали пакет в виртуальное окружение, то ищите его в: <домашний каталог вирт. окружения>\Lib\site-packages\prodcal

Проект можно вообще не устанавливать а скачать его напрямую с сайта PyPi. После чего распаковать и использовать код непосредственно в своём проекте.

Проект состоит из следующих файлов (все с расширением *.py):
config — описывает информацию о поддерживаемых календарях и о календаре выбранном по умолчанию
service — файл со вспомогательными функциями, вроде приведения типов и т.п., некоторые функции из этого файла мы разберём ниже
holidays — файл содержит реализацию основного и пока единственного класса ProdCal
каталог prodcals — содержит наборы календарей и файл prod_dict, который содержит реализацию класса ProdDict (о нём также ниже)


Примеры использования
from procal import ProdCal

my_first_prod_cal = ProdCal()

# Проверяем праздничный день 1 мая
my_first_prod_cal.is_work_day(2016, 5, 1)

# Проверяем рабочий день
my_first_prod_cal.is_work_day(2016, 4, 1)

# Проверяем выходной день
my_first_prod_cal.is_work_day(2016, 4, 2)

# Проверяем перенос празничного дня (рабочий день)
my_first_prod_cal.is_work_day(2016, 2, 20)

# Передаём сразу объект даты
my_first_prod_cal.is_work_day(date(2016, 5, 1)

# Передаём в качестве аргумента строку (today - сегодня)
my_first_prod_cal.is_work_day('today')

# Передаём в качестве аргумента строку (yesterday - вчера)
my_first_prod_cal.is_work_day('yesterday')

# Передаём в качестве аргумента строку (tomorrow - завтра)
my_first_prod_cal.is_work_day('tomorrow')

# Проверяем количество рабочих дней в различных месяцах
my_first_prod_cal.count_work_days([2016, 4, 1], [2016, 4, 30])
my_first_prod_cal.count_work_days([2016, 5, 1], [2016, 5, 31])
my_first_prod_cal.count_work_days([2016, 6, 1], [2016, 6, 30])

# Передаём сразу в формате даты и времени
my_first_prod_cal.count_work_days(date(2016, 4, 1), date(2016, 4, 30))
my_first_prod_cal.count_work_days(date(2016, 5, 1), date(2016, 5, 31))
my_first_prod_cal.count_work_days(date(2016, 6, 1), date(2016, 6, 30))

# Передаём дату начала ввиде текста (today, yesterday, tomorrow)
my_first_prod_cal.count_work_days('today', date(2016, 4, 30))
my_first_prod_cal.count_work_days('yesterday', date(2016, 4, 30))
my_first_prod_cal.count_work_days('tomorrow', date(2016, 4, 30))

# Передаём в качестве конечной даты количество дней от даты начала (включительно)
my_first_prod_cal.count_work_days([2016, 4, 1], 30)
my_first_prod_cal.count_work_days('today', 30)

# Проверяем количество выходных дней в различных месяцах
my_first_prod_cal.count_holidays([2016, 4, 1], [2016, 4, 30])
my_first_prod_cal.count_holidays([2016, 5, 1], [2016, 5, 31])
my_first_prod_cal.count_holidays([2016, 6, 1], [2016, 6, 30])

# Передаём сразу в формате даты и времени
my_first_prod_cal.count_holidays(date(2016, 4, 1), date(2016, 4, 30))
my_first_prod_cal.count_holidays(date(2016, 5, 1), date(2016, 5, 31))
my_first_prod_cal.count_holidays(date(2016, 6, 1), date(2016, 6, 30))

# Передаём дату начала ввиде текста (today, yesterday, tomorrow)
my_first_prod_cal.count_holidays('today', date(2016, 4, 30))
my_first_prod_cal.count_holidays('yesterday', date(2016, 4, 30))
my_first_prod_cal.count_holidays('tomorrow', date(2016, 4, 30))

# Передаём в качестве конечной даты количество дней от даты начала (включительно)
my_first_prod_cal.count_holidays([2016, 4, 1], 30)
my_first_prod_cal.count_holidays('today', 30)

# Рассчитываем конечную дату по рабочим дням
my_first_prod_cal.get_date_by_work_days([2016, 4, 1], 21))
my_first_prod_cal.get_date_by_work_days('today', 21)




Реализация

Структура производственного календаря

Все производственные календари находятся в подкаталоге prodcals в виде отдельных файлов. Формат названия файла соотв. буквенному коду страны по ISO в нижнем регистре. Например, росс. производственный календарь находится в файле ru.py.

Файл содержит два словаря: NON_WORK_DAY_DICT и WORK_DAY_DICT, они имеют одинаковую структуру, первый словарь описывает нерабочие дни (праздничные), а второй описывает переносы рабочих дней на выходные. Словари не содержат указания на «стандартные» нерабочие дни субботу и воскресенье.
Календарь описывают два вложенных словаря: в год вкладываются месяцы, значением месяца является список дней.
Для удобства работы с календарём был сделан отдельный класс ProdDict (унаследован от стандартного словаря) в котором реализован метод is_value, который возвращает True или False в зависимости от наличия в словаре переданного значения. На вход данный класс принимает только даты. Реализация класса ProdDict описана в файле prod_dict (расположен в подкаталоге prodcals).

Реализация класса ProdCal

Данный класс может быть создан и без указания каких-либо аргументов, в этом случае будет использован календарь по умолчанию (российский). Если требуется указать какой календарь использовать, то необходимо передать именованный аргумент locale=<значение>, где значение — это код страны по ISO в любом регистре. Пример для создания производственного календаря Украины:
from prodcal import ProdCal
my_prod_cal = ProdCal(locale='UA')

В настоящий момент поддерживаются календари следующих стран: Беларусь, Грузия, Казахстан, Россия, Украина.

Методы класса ProdCal
is_work_day

Вход: дата, список (с int), кортеж аргументов, строка (поддерживает только: 'today', tomorrow', 'yesterday')
Выход: bool

Описание: проверяет заданную дату на предмет того рабочий ли сегодня день.

Примечание: для удобства в этом и всех других методах реализована возможность передавать в качестве аргументов даты в удобном формате, как это реализовано описано в разделе, описывающим сервисные функции.

count_work_days, count_holidays

Вход: дата начала, дата окончания (периода), формат дат описан выше.
Выход: int

Описание: подсчитывает количество рабочих дней в заданном периоде (в случае count_work_days), а в случае count_holidays количество выходных дней.

get_date_by_work_days

Вход: дата начала, int
Выход: date

Описание: вычисляет конечную дату по заданному числу рабочих дней.

Описание сервисных функций

Напомню, что сервисные функции находятся в файле service.py.
Простейшая функция get_date_today преобразует переданное значение в необходимую дату, реализация самая незатейливая (пытливым умам предлагаю переписать под более эффективную конструкцию, например выбор из словаря).

def get_date_today(day):
    today = datetime.today().date()
    if 'today' == day:
        return today
    elif 'yesterday' == day:
        return today - timedelta(days=1)
    elif 'tomorrow' == day:
        return today + timedelta(days=1)
    raise ValueError('Unknown string format', day)


Магия возможности использования дат в различных форматах (если так корректно выражаться) реализована в функции cast.
Реализация функции cast
def cast(start_date, end_date):
    if isinstance(start_date, (tuple, list)) and isinstance(end_date, (tuple, list)):
        start_date, end_date = date(*start_date), date(*end_date)

    if isinstance(start_date, str):
        start_date = get_date_today(start_date)
    elif isinstance(start_date, (tuple, list)):
        start_date = date(*start_date)

    if isinstance(end_date, (tuple, list)):
        end_date = date(*end_date)
    elif isinstance(end_date, int):
        end_date = calc_days_by_int(start_date, end_date)

    if isinstance(start_date, date) and isinstance(end_date, date):
        pass
    else:
        raise ValueError("Unknown format for parse")


Вся идея очень простая, проверяем тип переданных аргументов и приводим всё к дате и возвращаем её. Если не разобрались бросаем исключение.

Ещё интересным местом является функция get_prodcals, которая по переданному значению подгружает из подкаталога prodcals нужный календарь. Возможность этого обеспечивается с помощью функции import_module() из стандартной библиотеки importlib, которая интерпретирует переданную строку как путь к модулю. Например: import_module('prodcal.prodcals.ru') эквивалентно from prodcals import ru. Главный смысл использования этой функции в том, чтобы не указывать явно какие календари загружать, что несколько облегчает дальнейшую поддержку.

Поддержка новых календарей

Поддержка новых календарей обеспечивается с помощью добавления в файл config.py данных о новых календарях, написании тестов и загрузки календаря в подкаталог prodcals. Кроме этого делать больше ничего не нужно.

https://calendar.yoip.ru/work/2025-proizvodstvennyj-calendar.html
[http://basicdata.ru/](http://basicdata.ru/online/calend/)
https://habr.com/ru/articles/776832/
https://github.com/ogt/workdays
https://github.com/dateutil/dateutil/


------------------------------------
- shebang. Вы знаете, для чего он? Если да, то почему он в каждом файле?
- использовать import_module — довольно плохая практика
- path = self[day.year][day.month] — тоже плохо. Потому что ProdCal().is_work_day(2015, 2, 2) выдаст KeyError: 2015, а должен что-то другое
- нет юникода: ProdCal().is_work_day(u'tomorrow') выдаст AttributeError: 'NoneType' object has no attribute 'year'
- локаль надо по-умолчанию брать системную, а не хардкодить в конфиге.



