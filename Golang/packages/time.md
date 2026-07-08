
## типы
- `time.Time` - представляет дату/время
- `time.Duration` - представляет продолжительность времени. Также поддерживает работу с локалями, часовыми поясами, таймерами и другими штуками.

## Шаблон даты

Шаблон даты используемый для преобразования строки в дату (и наоборот) представлен следующими опциями

| Значение    | опция шаблона                                  |
| ----------- | ---------------------------------------------- |
| Year        | 2006 ; 06                                      |
| Month       | Jan ; January ; 01 ; 1                         |
| Day         | 02 ; 2 ; _2 (For preceding 0)                  |
| Weekday     | Mon ; Monday                                   |
| Hour        | 15 ( 24 hour time format ) ; 3 ; 03 (AM or PM) |
| Minute      | 04 ; 4                                         |
| Second      | 05 ; 5                                         |
| AM/PM Mark  | PM                                             |
| Day of Year | 002 ; __2                                      |

## API

### Создание переменной даты
- `time.Date(year int, mon time.Month, date int, hour int, min int, sec int, time.UTC) time.Time` - получение даты по введенным параметрам 
- `time.Now() time.Time` - получить текущее время

### Преобразования типов
- `time.Parse(layout string, date string) (time.Time, error)` - дата из строки
- `(t time.Time).Format(layout string) string` - строка из даты

### Получение атрибутов даты
- `(t time.Time)Month() time.Month` - получить месяц от даты
- `(t time.Time)Year() int` - получить гот от даты 
- `(t time.Time)WeekDay() Weekday` - получить день недели
- `(t time.Time)Hour() int` - получить часы от даты
- `(t time.Time)Minute() int` - получить минуты от даты
- `(t time.Time)Second() int` - получить секунды от даты


### Сравнение дат
- `(t Time)After(u Time) bool` - true если `t` позднее `u`
- `(t Time)Before(u Time) bool` - true если `t` раньше `u`
- `(t Time)Equal(u Time) bool` - true если `t` == `u`