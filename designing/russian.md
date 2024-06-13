# VQL
Это описание обычного языка запросов (VQL) для систем управления базами данных (СУБД).  

**_Этот репозиторий находится на этапе разработки_**  

***Информация:*** 
VQL - это простой императивный язык для работы с базами данных, свободный от излишеств и помогающий создавать простые, понятные и эффективные запросы.  

***Ревизия:*** 
VQL-24 0.0.1  

## Список терминов: 
|Термин|Описание|
|:-|:-|
|БД|база данных|
|СУБД|система управления базами данных|
|Запись|строка двумерного массива (таблицы) в котором храняться данные|
|Ячейка|элемент записи|
|Столбец|совокупность всех ячеек находящихся в одной позиции во всех записях|


## Основные особенности.

- База данных -- это организованная коллекция данных, предназначенная для хранения информации.
- Коллекции данных можно представлять как двумерные массивы или таблицы. 
- Строка таблицы может называться "запись". 
- Запись состоит из ячеек, которые в контексте таблицы формируют столбцы. 
- Столбцы должны иметь имена и определяют имена для ячеек в записях. 
- Имена БД, таблиц, записей и столбцов (ячеек) должны соответствовать формату `^[a-zA-Z][a-zA-Z0-9_-]*`
- Записи могут иметь дополнительные служебные ячейки, имена которых должны начинатся с символа подчеркивания "_", то есть соответствовать формату  `^_[a-zA-Z0-9_-]+`
- Каждая запись в рамках таблицы должна иметь уникальный идентификатор "_id". 
- Ячейки имена которых имеют в своем составе постфикс "_id", являются ячейками для связи таблиц, где левая часть имени (до "_id"), является названием таблицы. 
- Ячейки не имеют типа, так как данные в ячейках это просто набор байт, и данные в ячейкам могут интерпретироваться как строки, которые можно преобразовывать в любой тип на стороне клентского ПО. 
- Результаты выборок данных желательно представлять в формате JSON.

## VQL позволяет:

- создавать, удалять и изменять БД и таблицы 
- описывать структуру данных 
- определять данные в БД и управлять ими 
- получать доступ к данным в СУБД 
- устанавливать разрешения на доступ к данным и на манипулирование данными 


## Язык запросов 

### Базовые концепции

- Запрос представляет собой программу 
- Запрос соистоит из логических блоков, называемых операциями 
- Каждая операция пишется на отдельно строке без конечных символов, либо с разелителем "\n" в конце каждой операции 
- VQL предполагает использование переменных, которые должны начинаться с символа $ и соответствовать шаблону `\$[a-zA-Z][a-zA-Z0-9_-]+` 
    - Переменные предназначены для временного хранения данных во время выполнения запроса 
    - Любая переменная кроме данных должна хранить информацию об статусе ошибки при её получении 
    - Переменные не требуют предварительного объявления или инициализации 
    - Переменная считается объявленной и иницализированной после любого упоминания этой переменной в левом блоке операции присваивания 
- Операции могут использовать конвееры, то есть данные от выполнения одного запроса передаются как входные в следующий запрос 
    - Конвеер обозначается символом "|" между запросами и соответствует шаблону `\s*|\s*` 
    - Запросы в конвеере могут находится как на разных строках, так и на одной 
    - Ответ из левой части конвеерной операции передается в правую часть и заполняет поля аргументов входной функции слева на право 
    - Если количество возвращаемых значение из левой части конвеера превышает количество аргументов правой части, то заполняются только те значения, которые предусмотренны правой частью конвеерной операции 
    - Если возвращаемых значений конвеерной операции недостаточно для заполнения аргументов, то формируется ошибка и выполнение всего запроса прекращается 
    - Недостаток заполнения аргументов правой части конвеерной операции, можно компенсировать непосредственным заполнением аргументов 
    - Полный конвеер может содержать много конвеерных операций, в которых данные передаются от одной операции к другой
- VQL поддерживает операции присваивания, сравнения, логические операции, арифметические операции и строковые конкатенации 
    - Присваивание обозначается символом "=" 
        - Слева от знака присваивания находится принимающая переменная и ничто другое 
            - Если слева от знака присваивания нет переменной, то данные передаются стандартной зарезервированной переменной $result 
            - Если переменная $result использовалась ранее, то данные этой переменной будут перезаписаны 
        - Справа от знака присваивания находится источник даннах 
            - Источником данных может быть другая переменная 
            - Источником данных может быть пользовательская функция 
            - Источником данных может быть стандартная директива 
            - Источником данных может быть конвеер после выполнения всех его конвеерных операций 
    - Операции сравнения 
        - Больше: ">" 
        - Меньше: "<" 
        - Равно: "==" 
        - Больше или равно: ">=" 
        - Меньше или равно: "<=" 
    - Логические операции 
        - Логическое И: "AND" 
        - Логическое ИЛИ: "OR" 
        - Логическое НЕ: "NOT" 
    - Арифметические операции поддерживают только целочисленные вычисления 
        - Сложение: "+" 
        - Вычитание: "-" 
        - Умножение: "*" 
        - Деление без остатка: "/" 
        - Остаток от деления: "%" 
        - Возведение в степень: "^" 
    - Строковая конкатенация 
        - Сложение строк: "." 
- Строка операции может начинаться с любого количества символов пробела или табуляции, эти символы не несут никакого лексического значения и нужны только для красивого оформления тела запроса 
- Между операндами в операции может быть любое количество символов пробела или табуляции 
- Любой незадекларированный набор символов является атомом, то есть его имя в исходном тексте запроса и его значение равны и являются константой 
    - Любое число само по себе является атомом, например, запись 123456789 является и именем числа и его значением 
    - Любой набор символов без пробела и без табуляции является строковой константой 
    - Для использования строки с пробелами и другими специальными символами внутри, нужно изспользовать обрамляющие кавычки (двойные или одинарные) 
- Любая переменная хранит не только значение, но и: 
    - Признак удачности операции в результае которой было получено хранимое значение (true, false) 
    - Ошибка если признак выполенния предыдущей операции был false
- Пользовательские функции и стандартные дерективы возвращают
    - Данные как результат операции 
    - Признак удачности операции 
    - Ошибку, либо её отсутствие "nil" 

### Типы данных

VQL не требует типизации переменных, либо типизации данных, но сами типы данных существуют в самой минималистичной форме 

|Тип|Обозначение|Описание|
|:-|:-|:-|
|Atom|[a-zA-Z][a-zA-Z0-9_-]+|Любой обозначение являющееся как собственным именем, так и собственным значением|
|Number|[0-9]+|Число, разновидность атома являющееся как собственным именем, так и собственным значением|
|String|""|Набор любых символов, которые могут иметь атомарный вид, либо могут быть заключены в кавычки (одинарные или двойные) если требуется использовать в значении специальные символы|
|Array|[]||


### Стандартные директивы и зарезервированные слова 

Сигнатура стандартных деректив: название_директивы(аргумент1, аргумент2, другие_аргументы) возвращаемое_значение

#### DDL — язык определения данных (Data Definition Language) 

|Директива|Аргументы|Возвращаемое значение|Описание|
|:-|:-|:-|:-|
|create-db|имя [string]|признак выполнения (true, false)|Создание базы данных|
|create-table|имя [string], названия_полей [array]|признак выполнения|Создание таблицы|

#### DML — язык изменения данных (Data Manipulation Language) 

|Директива|Аргументы|Возвращаемое значение|Описание|
|:-|:-|:-|:-|

#### DCL — язык управления данными (Data Control Language) 

|Директива|Аргументы|Возвращаемое значение|Описание|
|:-|:-|:-|:-|

### Пользовательские функции

Сигнатура пользовательских функций: 

## Старое как в SQL 

### DDL — язык определения данных (Data Definition Language) 

CREATE	Создает новую таблицу, представление таблицы или другой объект в БД
ALTER	Модифицирует существующий в БД объект, такой как таблица
DROP	Удаляет существующую таблицу, представление таблицы или другой объект в БД


### DML — язык изменения данных (Data Manipulation Language) 

SELECT	Извлекает записи из одной или нескольких таблиц
INSERT	Создает записи
UPDATE	Модифицирует записи
DELETE	Удаляет записи


### DCL — язык управления данными (Data Control Language) 

GRANT	Наделяет пользователя правами
REVOKE	Отменяет права пользователя
AUTH    Производит авторизацию пользователя
