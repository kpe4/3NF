
## Часть 1. Теоретический анализ

### 1. Определение первичного ключа

В качестве первичного ключа можно использовать составной ключ:
**(student_id, subject_id, exam_date)**
То есть одна запись однозначно определяется студентом, предметом и датой экзамена.
Например:
* `(1, 1, 2026-01-15)` — Петров П., математика
* `(1, 3, 2026-01-25)` — Петров П., информатика
Почему другие варианты не подходят:
* `student_id` не подходит, потому что один студент может сдавать несколько предметов;
* `subject_id` потому что один предмет сдают разные студенты;
* `teacher_id` потому что один преподаватель может работать с несколькими студентами;
* `exam_date` потому что в один день могут сдавать экзамен несколько студентов;
* `(student_id, subject_id)` если студент может сдавать один и тот же предмет повторно в разные даты.
---
### 2. Функциональные зависимости

```text
student_id -> student_name
student_id -> group_id

group_id -> group_name

teacher_id -> teacher_name

subject_id -> subject_name

(student_id, subject_id, exam_date) -> grade
```
---
### 3. Транзитивные зависимости

Транзитивная зависимость возникает, когда один неключевой атрибут зависит от другого неключевого атрибута.

В таблице имеются следующие транзитивные зависимости:

```text
student_id -> group_id -> group_name
```

То есть:

* `student_id` определяет группу студента;
* `group_id` определяет название группы.

Также:

```text
subject_id -> teacher_id -> teacher_name
```

То есть:

* `subject_id` определяет преподавателя;
* `teacher_id` определяет имя преподавателя.

Таким образом, имеются зависимости:

```text
student_id -> group_name
subject_id -> teacher_name
```

через промежуточные атрибуты.

---

### 4. Нормальная форма исходной таблицы

Исходная таблица находится в первой нормальной форме

потому что
1. все значения являются атомарными;
2. в каждой ячейке находится одно значение;
3. нет повторяющихся групп столбцов.

Однако таблица не находится во 2НФ, потому что первичный ключ составной:

```text
(student_id, subject_id, exam_date)
```
---

# Часть 2. Практическая нормализация до 3НФ

Для устранения избыточности исходную таблицу разделим на следующие таблицы:

1. `Groups` — группы студентов;
2. `Students` — студенты;
3. `Teachers` — преподаватели;
4. `Subjects` — предметы;
5. `StudentGrades` — результаты экзаменов.
---

## 1. Таблица Groups
Хранит информацию о группах.

| group_id | group_name |
| -------- | ---------- |
| G-1      | Группа А   |
| G-2      | Группа Б   |

### SQL

```sql
CREATE TABLE Groups (
    group_id VARCHAR(10) PRIMARY KEY,
    group_name VARCHAR(100) NOT NULL UNIQUE
);
```
---
## 2. Таблица Students

Хранит информацию о студентах.

| student_id | student_name | group_id |
| ---------: | ------------ | -------- |
|          1 | Петров П.    | G-1      |
|          2 | Иванов И.    | G-1      |
|          3 | Сидоров С.   | G-2      |
|          4 | Петров П.    | G-1      |
|          5 | Кузнецов К.  | G-2      |

### SQL
```sql
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100) NOT NULL,
    group_id VARCHAR(10) NOT NULL,
    
    FOREIGN KEY (group_id)
        REFERENCES Groups(group_id)
);
```
---
## 3. Таблица Teachers

Хранит информацию о преподавателях.

| teacher_id | teacher_name |
| ---------: | ------------ |
|          5 | Петрова М.   |
|          7 | Смирнов А.   |
|          8 | Козлова Е.   |

### SQL
```sql
CREATE TABLE Teachers (
    teacher_id INT PRIMARY KEY,
    teacher_name VARCHAR(100) NOT NULL
);
```
---
## 4. Таблица Subjects

Хранит информацию о предметах и преподавателях, которые их ведут.

| subject_id | subject_name | teacher_id |
| ---------: | ------------ | ---------: |
|          1 | Математика   |          5 |
|          2 | Физика       |          7 |
|          3 | Информатика  |          8 |

### SQL
```sql
CREATE TABLE Subjects (
    subject_id INT PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL UNIQUE,
    teacher_id INT NOT NULL,
    
    FOREIGN KEY (teacher_id)
        REFERENCES Teachers(teacher_id)
);
```
---
## 5. Таблица StudentGrades

Хранит непосредственно результаты сдачи экзаменов.

| student_id | subject_id | exam_date  | grade |
| ---------: | ---------: | ---------- | ----: |
|          1 |          1 | 2026-01-15 |     4 |
|          2 |          2 | 2026-01-20 |     3 |
|          3 |          1 | 2026-01-18 |     5 |
|          4 |          3 | 2026-01-25 |     5 |
|          5 |          2 | 2026-01-22 |     4 |

### SQL
```sql
CREATE TABLE StudentGrades (
    student_id INT NOT NULL,
    subject_id INT NOT NULL,
    exam_date DATE NOT NULL,
    grade INT NOT NULL,

    PRIMARY KEY (student_id, subject_id, exam_date),

    FOREIGN KEY (student_id)
        REFERENCES Students(student_id),

    FOREIGN KEY (subject_id)
        REFERENCES Subjects(subject_id)
);
```
---
# ER-диаграмма

Связи между таблицами можно представить следующим образом:

```text
┌─────────────────────┐
│       Groups        │
├─────────────────────┤
│ PK group_id         │
│    group_name       │
└──────────┬──────────┘
           │
           │ 1 : N
           ▼
┌─────────────────────┐
│      Students       │
├─────────────────────┤
│ PK student_id       │
│    student_name     │
│ FK group_id         │
└──────────┬──────────┘
           │
           │ 1 : N
           ▼
┌─────────────────────────────┐
│       StudentGrades         │
├─────────────────────────────┤
│ PK,FK student_id            │
│ PK,FK subject_id            │
│ PK exam_date                │
│    grade                    │
└──────────────┬──────────────┘
               │
               │ N : 1
               ▼
┌─────────────────────┐
│      Subjects       │
├─────────────────────┤
│ PK subject_id       │
│    subject_name     │
│ FK teacher_id       │
└──────────┬──────────┘
           │
           │ N : 1
           ▼
┌─────────────────────┐
│      Teachers       │
├─────────────────────┤
│ PK teacher_id       │
│    teacher_name     │
└─────────────────────┘
```
N : 1 - много к одному
# Итоговый список таблиц

После нормализации получили **5 таблиц**:

| Таблица         | Назначение                     |
| --------------- | ------------------------------ |
| `Groups`        | хранение групп                 |
| `Students`      | хранение студентов             |
| `Teachers`      | хранение преподавателей        |
| `Subjects`      | хранение предметов             |
| `StudentGrades` | хранение результатов экзаменов |
