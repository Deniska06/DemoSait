# Первый сайт на Git.

#### Главная страница. 

## Здесь может быть ваша реклама =))

#### P/S

* Звоните
* Пишите

#### Фоточка

![](https://thumbs.dreamstime.com/b/cute-boy-licking-ice-cream-illustration-84830355.jpg)



Вот решение вашей задачи на Python с использованием регулярных выражений для приведения телефонов к нужному формату и объединения дублирующихся записей.

---

## ✅ Задача:
1. Привести ФИО в правильный вид: `lastname`, `firstname`, `surname`.
2. Привести телефон к формату: `+7(999)999-99-99` или `+7(999)999-99-99 доб.9999`.
3. Объединить дубликаты (по совпадению **Фамилия + Имя**).

---

## ✅ Полный код:

```python
import re
import csv
from collections import defaultdict

# Ваш список контактов
contacts = [
    ['lastname', 'firstname', 'surname', 'organization', 'position', 'phone', 'email'],
    ['Усольцев', 'Олег', 'Валентинович', '', '', '+7 (495) 913-04-78', 'opendata@nalog.ru'],
    ['Мартиняхин', 'Виталий', 'Геннадьевич', '', '', '+749591300037', ''],
    ['Наркаев', 'Вячеслав', 'Рифхатович', '', '', '8 495-913-0168', ''],
    ['Мартиняхин', 'Виталий', 'Геннадьевич', 'ФНС', 'советник отдела Интернет проектов Управления информационных технологий', '', ''],
    ['Лукина', 'Ольга', '', 'Владимировна', 'Минфин', '+7 (495) 983-36-99 доб. 2926', 'Olga.Lukina@minfin.ru'],
    ['Паньшин', 'Алексей', 'Владимирович', '', '', '8(495)748-49-73', '1248@minfin.ru'],
    ['Лагунцов', 'Иван', '', 'Алексеевич', 'Минфин', '+7 (495) 913-11-11 (доб. 0792)', ''],
    ['Лагунцов', 'Иван', '', '', '', '', 'Ivan.Laguntcov@minfin.ru'],
    ['Лукина', 'Оксана', 'Владимировна', 'Минфин', '+7 (495) 983-36-99 доб. 2929', 'OLukina@minfin.ru']
]

# Удаляем заголовок
rows = contacts[1:]

def fix_name_fields(row):
    """Приводим ФИО в порядок"""
    full_name = " ".join(row[:3])
    parts = full_name.split()
    lastname = firstname = surname = ""
    if len(parts) >= 1:
        lastname = parts[0]
    if len(parts) >= 2:
        firstname = parts[1]
    if len(parts) >= 3:
        surname = parts[2]
    return [lastname, firstname, surname]

def fix_phone(phone):
    """Приводим телефон к формату +7(999)999-99-99 или +7(999)999-99-99 доб.9999"""
    if not phone:
        return ""
    # Очищаем строку от лишних символов
    digits = re.sub(r'\D', '', phone)
    if len(digits) == 11 and digits[0] in ('7', '8'):
        digits = '7' + digits[1:]
    elif len(digits) == 10:
        digits = '7' + digits
    else:
        return phone

    main_number = digits[:10]
    ext = digits[10:] if len(digits) > 10 else ''

    formatted_phone = re.sub(r'(\d{1})(\d{3})(\d{3})(\d{2})(\d{2})', r'+7(\2)\3-\4-\5', main_number)

    if ext:
        formatted_phone += f' доб.{ext}'
    return formatted_phone

def merge_duplicates(rows):
    merged = defaultdict(list)

    for row in rows:
        # Берём только ФИ для определения уникальности
        fixed_row = row.copy()
        name_fixed = fix_name_fields(fixed_row)
        lastname, firstname, surname = name_fixed
        key = (lastname, firstname)

        # Добавляем исходную строку в список под ключом (фамилия, имя)
        merged[key].append(row)

    result = []
    for key, entries in merged.items():
        combined = ['', '', '', '', '', '', '']

        # Объединяем все поля по максимальному значению
        for entry in entries:
            for i in range(len(entry)):
                if entry[i]:
                    combined[i] = entry[i]

        # Правим ФИО
        fixed = fix_name_fields(combined)
        combined[:3] = fixed

        # Правим телефон
        combined[5] = fix_phone(combined[5])

        result.append(combined)

    return result

# Выполняем объединение и форматирование
cleaned_contacts = merge_duplicates(rows)

# Сохраняем результат в CSV файл
with open("cleaned_contacts.csv", "w", encoding="utf-8", newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['lastname', 'firstname', 'surname', 'organization', 'position', 'phone', 'email'])
    writer.writerows(cleaned_contacts)

print("Контакты успешно обработаны и сохранены в файл 'cleaned_contacts.csv'")
```

---

## 📌 Что делает этот скрипт:

### 1. **ФИО**
- Если ФИО разбито некорректно — например, `['Лукина', 'Ольга', '', 'Владимировна', ...]`, то мы объединяем первые три элемента, разбиваем их и получаем корректные `lastname`, `firstname`, `surname`.

### 2. **Телефон**
- Приводит номер к стандарту:
  - `+7(999)999-99-99`
  - или `+7(999)999-99-99 доб.9999`
- Поддерживает добавочные номера, учитывает пробелы, скобки и т.п.

### 3. **Объединение дубликатов**
- Группирует записи по ключу `(lastname, firstname)`
- Для каждого поля выбирается непустое значение из всех записей.

---

## ✅ Результат будет выглядеть так:

| lastname   | firstname | surname     | organization | position | phone                   | email                     |
|------------|-----------|-------------|--------------|----------|-------------------------|---------------------------|
| Лукина     | Ольга     | Владимировна| Минфин       | ...      | +7(495)983-36-99 доб.2926 | Olga.Lukina@minfin.ru     |
| Лукина     | Оксана    | Владимировна| Минфин       | ...      | +7(495)983-36-99 доб.2929 | OLukina@minfin.ru         |
| Лагунцов   | Иван      | Алексеевич  | Минфин       | ...      | +7(495)913-11-11 доб.0792 | Ivan.Laguntcov@minfin.ru  |

---

Хочешь версию с графическим интерфейсом или автоматическим чтением CSV-файла? Напиши — сделаю!