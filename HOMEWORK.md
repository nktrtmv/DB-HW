# Домашнее задание: Работа с PostgreSQL

## 1. Создание таблиц

### Таблица Workers
```sql
CREATE TABLE Workers (
  W_id SERIAL PRIMARY KEY,
  lastname TEXT,
  firstname TEXT
);
```

### Таблица Operations
```sql
CREATE TABLE Operations (
  Op_id SERIAL PRIMARY KEY,
  opname TEXT,
  cost DECIMAL
);
```

### Таблица Events
```sql
CREATE TABLE Events (
  Ev_id SERIAL PRIMARY KEY,
  W_id INTEGER,
  Op_id INTEGER,
  Date DATE,
  Number INTEGER,
  FOREIGN KEY (W_id) REFERENCES Workers(W_id),
  FOREIGN KEY (Op_id) REFERENCES Operations(Op_id)
);
```

## 2. Проверка вставки и удаления записей

Для демонстрации ограничений, попробуйте вставить и удалить запись в таблицу `Events` без предварительной вставки в таблицы `Workers` и `Operations` при наличии ограничений по ссылочной целостности (наличия FOREIGN KEY) и без ограничения по ссылочной целостности.

### 2.1.
```sql
INSERT INTO Events (Ev_id, W_id, Op_id, Date, Number) VALUES (1, 1, 1, '2023-10-10', 25);
```
В таблицах `Workers` и `Operations` нет записей с заданными ключами, поэтому получаем ошибку:
```
[23503] ERROR: insert or update on table "events" violates foreign key constraint "events_w_id_fkey"
Detail: Key (w_id)=(1) is not present in table "workers".
```

### 2.2.
Теперь добавим записи в таблицы `Workers` и `Operations`:
```sql
INSERT INTO Workers (lastname, firstname) VALUES ('Smith', 'John');
INSERT INTO Operations (opname, cost) VALUES ('Operation1', 100);
```
И попробуем снова вставить запись в таблицу `Events`:
```sql
INSERT INTO Events (Ev_id, W_id, Op_id, Date, Number) VALUES (1, 1, 1, '2023-10-10', 25);
```
Теперь запись добавлена: ![Alt text](<Screenshot 2023-10-12 at 09.45.08.png>)

### 2.3. 
Попробуем удалить запись из `Workers` или `Operations`, на которую ссылается `Events`:
```sql
DELETE FROM Workers WHERE W_id = 1;
```
Получаем ошибку, потому что нарушаем внешние ключи:
```
[23503] ERROR: update or delete on table "workers" violates foreign key constraint "events_w_id_fkey" on table "events"
Detail: Key (w_id)=(1) is still referenced from table "events".
```

### 2.4.
Теперь уберем ссылочную целостность.
Удалим внешние ключи из таблицы `Events`:
```sql
ALTER TABLE Events DROP CONSTRAINT events_w_id_fkey;
ALTER TABLE Events DROP CONSTRAINT events_op_id_fkey;
```
Теперь мы можем свободно вставлять записи в `Events` без записей в `Workers` и `Operations`:
```sql
INSERT INTO Events (EV_id, W_id, Op_id, Date, Number) VALUES (2, 2, 2, '2023-10-10', 25);
```

![Alt text](<Screenshot 2023-10-12 at 09.51.58.png>)

Мы также можем свободно удалять записи из `Workers` и `Operations`, даже если на них есть ссылки в `Events`:
```sql
DELETE FROM Workers WHERE W_id = 1;
```
![Alt text](<Screenshot 2023-10-12 at 09.53.02.png>)

## 3. Ограничения для поля Number в таблице Events

```sql
ALTER TABLE Events
ADD CONSTRAINT chk_number CHECK (Number > 0 AND Number < 300),
ALTER COLUMN Number SET DEFAULT 20;
```

![Alt text](<Screenshot 2023-10-12 at 09.54.58.png>)

## 5. Добавление поля в таблицу Workers

```sql
ALTER TABLE Workers ADD COLUMN age INTEGER;
```

![Alt text](<Screenshot 2023-10-12 at 09.56.15.png>)

## 6. Добавление внешнего ключа в таблицу Events

В пункте **2.4.** внешние ключи были удалены, добавим их обратно

```sql
ALTER TABLE Events ADD FOREIGN KEY (W_id) REFERENCES Workers(W_id);
ALTER TABLE Events ADD FOREIGN KEY (Op_id) REFERENCES Operations(Op_id);
```
![Alt text](<Screenshot 2023-10-12 at 10.00.57.png>)

Важно уточнить, что прежде добавления внешних ключей – были удалены все записи из всех таблиц, которые нарушают внешние ключи

## 7. Изменение типа колонки в таблице Events

```sql
ALTER TABLE Events ALTER COLUMN Number TYPE BIGINT;
```

![Alt text](<Screenshot 2023-10-12 at 10.01.53.png>)