# Дымовые тесты

Существующая универсальная реализация дымовых тестов позволяет использовать базовые/«дымовые» проверки, для которых не требуется написание сложных тестов или перестройка схемы разработки конфигурации 1С.

Тесты удобно использовать перед выпуском релиза/обновлений конфигурации и/или перед установкой релиза/обновлений в рабочую базу.

Реализация тестов проверяет открытие/закрытие различных форм с учетом прав доступа (права "Использование/Просмотр") для пользователей с различными ролями (полные или ограниченные права). 

А также ввод документов на основании других документов. Подробнее в разделе ниже `Дымовое тестирование ввода документов на основании`

Проверяются следующие виды наиболее активно используемых форм:

+ формы списков и формы выбора справочников
+ форма элементов справочников
+ формы списков и формы выбора документов
+ формы документов
+ формы отчетов
+ формы обработок

Также для каждого справочника проверяются 3 дополнительных теста c учетом прав доступа (права "Просмотр/Интерактивное добавление"):

1. создание нового элемента и открытие его формы
1. копирование существующего элемента и открытие формы созданного элемента
1. открытие формы существующего элемента

Для каждого документа проверяются 3 дополнительных теста c учетом прав доступа (права "Просмотр/Интерактивное добавление"):

1. открытие формы существующего документа (берется последний по дате)
1. перенос существующего документа на текущий день и открытие формы этого документа
1. открытие формы нового документа

## Настройка дымовых тестов под конкретную конфигурацию

В связи с универсальностью дымовых тестов иногда возникает потребность скорректировать запуск дымовых тестов под конкретную конфигурацию.

Например, часть форма не предназначена для работы в интерактивном режиме и т.п.

Не рекомендуется добавлять в исключения формы по другим причинам – например, эти формы редко открываются, не работают в выбранном виде клиента и т.п., т.к. это приводит к ошибкам, которые постоянно существуют, что не всегда верно.

Подобные формы необходимо добавить в исключения.

### Исключения для версии 4.1.Х.Х и выше

Для добавления исключения в корне фреймворка xUnitFor1C, рядом с файлом xddTestRunner.epf, нужно создать файл smoke.json

Это файл json-формата  с различными коллекциями и массивами

#### В нем могут быть 4 коллекции – Справочники, Документы, Отчеты, Обработки

И подразделы/простые массивы-перечисления (для справочников и документов) – Списки, Существующие, Новые, ПеренестиДату (для документов)

В перечислении уже указываются конкретные имена метаданных, которые не нужно открывать, а нужно пропустить.

#### Отдельная настройка `ИсключитьФормыЗависящиеОтОтключенныхФункциональныхОпций`

Она отвечает за исключение форм, которые зависят от отключенных функциональных опций.

Тип: Булево (true или false)

По умолчанию настройка не используется, т.е. проверки функциональных опций не выполняется.

Настройка важна для `толстых` конфигураций, например, ERP или УХ.

#### Примерный текст файла

```json
{
	"smoke": {

    	"ИсключитьФормыЗависящиеОтОтключенныхФункциональныхОпций" : true,

    	"Справочники": {
        	"Списки": [
            	"ПростойСправочник",
	            "СоставПериметра"
    	    ],
        	"Новые": [
            	"ПростойСправочник2",
	            "СоставПериметра"
    	    ]
	    },
    	"Отчеты": [
        	"Отчет1"
	    ],
    	"Обработки": [
        	"xddGuidShow"
	    ]
	}
}
```

#### Исключения для типовых конфигураций 1С, основанных на БСП

Пример файла настройки исключений - https://github.com/xDrivenDevelopment/xUnitFor1C/blob/develop/examples/smoke.bsp.json

### Исключения для версии ниже 4.1.Х.Х (более сложный способ) - не рекомендуется

Описанный ниже способ хуже, чем вышеуказанный метод указания исключений:

+ нужно менять код внешней обработки, что усложняет сопровождение
+ код обработки менять сложнее, чем простой текстовый файл
+ текст файла удобнее держать в репозитории исходников, чем код внешней обработки

Исключения задаются непосредственно в файле-теста `Tests\Smoke\Тесты_ОткрытиеФормКонфигурации.epf`

В конце файла есть набор методов 

```bsl
+ //{ блок переопределения исключений, чтобы не открывать формы
+ Функция ПолучитьСписокИсключений_Справочники_Списки() Экспорт	
+ Функция ПолучитьСписокИсключений_Справочники_Существующие() Экспорт
+ Функция ПолучитьСписокИсключений_Справочники_Новые() Экспорт
+ Функция ПолучитьСписокИсключений_Документы_Списки() Экспорт
+ Функция ПолучитьСписокИсключений_Документы_Существующие() Экспорт
+ Функция ПолучитьСписокИсключений_Документы_ПеренестиДату() Экспорт
+ Функция ПолучитьСписокИсключений_Документы_Новые() Экспорт
+ Функция ПолучитьСписокИсключений_Отчеты() Экспорт
+ Функция ПолучитьСписокИсключений_Обработки() Экспорт
+ //} конец блока
```

Формат этих методов
```bsl
Функция ПолучитьСписокИсключений_Справочники_Списки() Экспорт
	Результат = Новый СписокЗначений;
	
	Результат.Добавить("ирАлгоритмы"); // Аналогично добавляем наименования нужных метаданных
	
	Возврат Результат;
КонецФункции
```

Нужно добавить имя метаданного-исключения в соответствующий метод в виде указанного кода.

## Дымовое тестирование ввода документов на основании

Данная обработка может быть использована и в [VanessaBehavior](https://github.com/silverbulleters/vanessa-behavior) и в [xUnit](https://github.com/xDrivenDevelopment/xUnitFor1C).
Запускать данный набора тестов рекомендуется базе данных в которой уже есть заполненные документы.

### Дымовое тестирование xUnit

Для заполнения списка исключений документов из проверки их необходимо заполнить в модуле документа обработки в процедуре ```ПолучитьСписокИсключений_ДокументыПроведенные``` и/или ```ПолучитьСписокИсключений_ДокументыНеПроведенные```

### Дымовое тестирование VanessaBehavior

Для возможностей запуска дымового тестирования можно использовать данную обработку, как пример сниппетов для генерации feature файлов и использования сниппетов. 
В обработке используется несколько сниппетов:

```
Я открываю форму документа "Документ" заполненного на основании проведенного "ДокументОснование"
Я открываю форму документа "Документ" заполненного на основании не проведенного "ДокументОснование"
Я открываю форму документа "Документ" заполненного на основании проведенного "ДокументОснование" номер "НомерДокументаОснования" от "ДатаДокументаОснования"
Я открываю форму документа "Документ" заполненного на основании не проведенного "ДокументОснование" номер "НомерДокументаОснования" от "ДатаДокументаОснования"
```

Данный сниппет получает форму, открывает ее и потом закрывает. В теории проверяем возможность работы процедур "ПриСозданииНаСервере", "ПриОткрытии", "ОбработкаЗаполнения"

### Быстрый старт для типовых конфигураций VanessaBehavior

Для быстрого старта необходимо открыть данныю обработку в режиме предприятия и нажать кнопку "Генерация фич", после генерации необходимых feature файлов, предложит выбрать каталог где будут созданны feature файлы в разрезе документов оснований.

Если стоит галочка "Указывать документ основание", то происходит указание номера и даты документа на основании которого будет создаваться документ.

Файлы создаются по имени документа основания, включают все документы которые можно создать на основании документа основания. Документом основания выбирается последний проведенный и не проведенный документ. Этого достаточно для первого старта, в дальнейшем предполагается, что при добавлении новых документов разработчик сам подкорретирует feature файл с необходимым документом.

Предполагается, что перегенерация для типовых конфигураций будет происходить только для репозитория git вы всегда сможете увидеть только добавленные формы в фича файлах, а те которые исправляли сможете вернуть на правильное поведение.