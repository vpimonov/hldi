# hldi
home laser direct imaging

# Протокол взаимодействия хоста с контроллером

## Общие сведения
Контроллер прикидывается HID устройством.
Управляющая программа шлет контроллеру и принимает пакеты по 64 байта.
Каждый отправленный пакет содержит команду и может содержать данные.
Таким образом можно выделить три вида пакетов:
1. Команда без параметра
2. Команда с целочисленным параметром
3. Команда с адресом и массивом байт

Отправка команды это передача пакета вида {0, длина, cmd, data...}. Пакет отсылается контроллеру, затем ожидается ответ. Ответ вида {0, 0, 0, данные} считается подтверждением успешного приема команды, в зависимости от команды данных можетне быть.

## Формат пакета
Пакет состоит из двухбайтового поля длины и полезной нагрузки (массива байт указанной длины).
Первым байтом в нагрузке идет байт команды cmd. Следом варинтты:
1. Ничего
2. Целое число (четыре байта litle ending)
3. Адрес (четыре байта litle ending) и массив данных

### Перечень команд
|cmd|par|data|description|
|---|---|---|---|
|  1| | |Прочитать параметры из контроллера|
|  2| | |Переконфигурировать (применить настройки или сбросить в дефолт?)|
|  3| AAA| |Установить указатель в контроллере на адрес AAA, возможны дополнительные спец. эффекты ?|
|  4| AAA|LLL|Выдать с адреса AAA массив длиной LLL. Формат {0, len+6, cmd, A1,A2,A3,A4, L1,L2,L3,L4}, где A1-A4 байтовое представление адреса adr, L1-L4 длины.|
|  5| AAA|BUF|Принять по адресу AAA массив BUF. Формат {0, len+6, cmd, A1,A2,A3,A4, байты..блока, 0, 0}, где A1-A4 байтовое представление адреса adr|
|  6| | |Стоп перемещений|
|  7| NNN||Перемещение по X на NNN|
|  8| NNN||Перемещение по Y на NNN|
|  9| NNN||Установить кол-во шагов по X. Разрешение ?|
| 10| NNN||Установить кол-во шагов по Y. Разрешение ?|
| 11| NNN||Установить скорость по X.|
| 12| NNN||Установить скорость по Y.|
| 13| NNN||Включить лазер с мощностью NNN. Время во включенном состоянии отслеживается хостом.|
| 14| AAA||Установить адрес буфера для экспозиции = AAA ?|
| 15| NNN||Установить X=NNN|
| 16| NNN||Установить Y=NNN|

## Пример пакета для команды с параметром (int)
Пакет: {0, 6, cmd, i1, i2, i3, i4} отсылается контроллеру (где i1-i4 байтовое представление int), затем ожидается ответ. Ответ вида {0, 0, 0} считается подтверждением приема команды.

## Запрос состояния командой cmd=4 и адресом adr
Пакет вида команда  {0, len+6, cmd, A1,A2,A3,A4, байты..блока, 0, 0} отсылается контроллеру (где A1-A4 байтовое представление адреса adr). Ответ вида {0, 0, 0} считается подтверждением приема команды.

## Отправка блока данных до 56 байт с командой cmd=5 и адресом adr
Пакет вида команда  {0, len+6, cmd, A1,A2,A3,A4, байты..блока, 0, 0} отсылается контроллеру (где A1-A4 байтовое представление адреса adr). Ответ вида {0, 0, 0} считается подтверждением приема команды.

## Параметры контроллера
```
uint32_t pcnf; //адрес ПИД коэффициентов 3 x int32_t (P, I, D)
uint32_t lbuf; //адрес буфера лазера (используется и как буфер ПИД-а и для обновления прошивки)
uint32_t sbuf; //адрес переменных состояния 4 x int32_t (SPX, SPY, SPS, Flags)
uint16_t slbuf; //тоже что-то для лазера
uint8_t verl; // MCU
uint8_t verh; // MCU
```

## Обновление прошивки?
| команда | параметр | данные |
|---------|----------|--------|
| 3 | 0 | |
| 5 | adr | boot[] |
| 3 | 0x200007ed |
| 3 | 0 | |

## Сброс МК ? 
| команда | параметр | данные |
|---------|----------|--------|
| 3 | 0x05fa0000 | |





 
