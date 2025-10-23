dfgdfgdfgdfgdfg # fdgdfg

Отлично 👍 — вот полная, чистая и аккуратная версия кода,
где все таблицы написаны по-простому (маленькими буквами, без constraint),
но триггеры оставлены такими же, как у тебя были в прошлый раз —
то есть с inserted / deleted, getdate() и system_user (вариант для MS SQL Server).

⸻


create database varjupaik1;
use varjupaik1;

-- таблица статусов
create table staatus (
    staatusid int primary key,
    nimetus varchar(15) not null
);

-- таблица пород
create table tougu (
    touguid int primary key,
    nimetus varchar(20) not null
);

-- тип корма
create table toidutyyp (
    tyypid int primary key,
    nimetus varchar(20) not null
);

-- корм
create table toit (
    toitid int primary key,
    nimetus varchar(20) not null,
    toidutyyp_tyypid int not null,
    foreign key (toidutyyp_tyypid) references toidutyyp(tyypid)
);

-- работник
create table tootaja (
    tootajaid int primary key,
    taisnimi varchar(30) not null,
    ametikoht varchar(20) not null,
    telefon int not null
);

-- владелец
create table omanik (
    omanikid int primary key,
    taisnimi varchar(30) not null,
    telefon varchar(10) not null,
    email varchar(50) not null
);

-- животное
create table loom (
    loomid int primary key,
    loomnimi varchar(30) not null,
    tougu_touguid int not null,
    staatus_staatusid int not null,
    kasutaja varchar(50),
    foreign key (tougu_touguid) references tougu(touguid),
    foreign key (staatus_staatusid) references staatus(staatusid)
);

-- усыновление (adapteerimine)
create table adapteerimis (
    adapteerimineid int primary key,
    loom_loomid int not null,
    omanik_omanikid int not null,
    kuupaev date not null,
    foreign key (loom_loomid) references loom(loomid),
    foreign key (omanik_omanikid) references omanik(omanikid)
);

-- расписание кормлений
create table soogiaeg (
    id int primary key,
    loom_loomid int not null,
    kuupaev date not null,
    toit_toitid int not null,
    tootaja_tootajaid int not null,
    foreign key (loom_loomid) references loom(loomid),
    foreign key (toit_toitid) references toit(toitid),
    foreign key (tootaja_tootajaid) references tootaja(tootajaid)
);

-- таблица логов
create table logi (
    logiid int identity(1,1) primary key,
    kuupaev datetime,
    andmed varchar(200),
    kasutaja varchar(100)
);

-- данные
insert into staatus values
(1, 'tervislik'),
(2, 'haige'),
(3, 'karanteen');

insert into tougu values
(1, 'devonreks'),
(2, 'ookerkass'),
(3, 'sfinks');

insert into toidutyyp values
(1, 'kuivtoit'),
(2, 'konserv'),
(3, 'eridieet');

insert into toit values
(1, 'royal canin', 1),
(2, 'whiskas', 2),
(3, 'hill', 3);

insert into tootaja values
(1, 'mari tamm', 'veterinaar', 51234567),
(2, 'juhan maasikas', 'hooldaja', 52345678);

insert into omanik values
(1, 'kati kask', '5551234', 'kati@example.com'),
(2, 'peeter puu', '5555678', 'peeter@example.com');

insert into loom values
(1, 'bella', 1, 1, null),
(2, 'simba', 2, 2, null),
(3, 'poku', 3, 1, null);

insert into adapteerimis values
(1, 1, 1, '2024-12-01'),
(2, 2, 2, '2025-01-10');

insert into soogiaeg values
(1, 1, '2025-05-20', 1, 1),
(2, 2, '2025-05-21', 2, 2),
(3, 3, '2025-05-22', 3, 1);

-- удаление старых триггеров (если есть)
if object_id('adapteerimisinsert', 'TR') is not null drop trigger adapteerimisinsert;
if object_id('adapteerimisdelete', 'TR') is not null drop trigger adapteerimisdelete;
if object_id('adapteerimisupdate', 'TR') is not null drop trigger adapteerimisupdate;

-- триггеры (оставлены как в оригинале, с inserted/deleted)

create trigger adapteerimisinsert
on adapteerimis
for insert
as
insert into logi (kuupaev, kasutaja, andmed)
select
    getdate(),
    system_user,
    concat(
        'lisatud andmed: adapteerimineID = ', inserted.adapteerimineid,
        ', loomnimi = ', loom.loomnimi,
        ', omanikID = ', inserted.omanik_omanikid,
        ', kuupaev = ', inserted.kuupaev
    )
from inserted
inner join loom on inserted.loom_loomid = loom.loomid;

create trigger adapteerimisdelete
on adapteerimis
for delete
as
insert into logi (kuupaev, kasutaja, andmed)
select
    getdate(),
    system_user,
    concat(
        'kustutatud andmed: adapteerimineID = ', deleted.adapteerimineid,
        ', loomnimi = ', loom.loomnimi,
        ', omanikID = ', deleted.omanik_omanikid,
        ', kuupaev = ', deleted.kuupaev
    )
from deleted
inner join loom on deleted.loom_loomid = loom.loomid;

create trigger adapteerimisupdate
on adapteerimis
for update
as
insert into logi (kuupaev, kasutaja, andmed)
select
    getdate(),
    system_user,
    concat(
        'vanad andmed: adapteerimineID = ', deleted.adapteerimineid,
        ', loomnimi = ', loom.loomnimi,
        ', omanikID = ', deleted.omanik_omanikid,
        ', kuupaev = ', deleted.kuupaev,
        ' || uued andmed: adapteerimineID = ', inserted.adapteerimineid,
        ', loomnimi = ', loom.loomnimi,
        ', omanikID = ', inserted.omanik_omanikid,
        ', kuupaev = ', inserted.kuupaev
    )
from deleted
inner join inserted on deleted.adapteerimineid = inserted.adapteerimineid
inner join loom on inserted.loom_loomid = loom.loomid;

-- проверка
select * from logi;


⸻

✅ Что здесь сделано:
	•	Все таблицы переписаны просто, маленькими буквами, без constraint-ов.
	•	Все связи (foreign key) заданы коротко и понятно.
	•	Триггеры полностью сохранены как в твоей прошлой версии — с getdate(), system_user, inserted, deleted, concat() и без begin / end.
	•	Подходит для MS SQL Server.

Хочешь, я сделаю ту же структуру, но чтобы работала в MySQL (с now() и current_user()) — без изменения логики?
Отличный вопрос 👍

Короткий ответ:
👉 **Да, в большинстве СУБД блоки `BEGIN ... END` в триггерах нужны, если у тебя внутри триггера больше одного выражения SQL.**
Если только одно — обычно можно без них.

---

### Подробно:

#### 🔹 **1. MySQL**

* Если в теле триггера **одно SQL-выражение**, можно написать без `BEGIN ... END`.
* Если **несколько**, обязательно использовать блок `BEGIN ... END` и **определить разделитель** (`DELIMITER`), например:

```sql
DELIMITER //
CREATE TRIGGER before_insert_user
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
    SET NEW.updated_at = NOW();
END//
DELIMITER ;
```

Без `BEGIN ... END` тут будет ошибка, потому что MySQL считает, что тело триггера — одно выражение.

---

#### 🔹 **2. PostgreSQL**

* В PostgreSQL тело триггера — **всегда функция** (обычно на PL/pgSQL), и внутри неё **всегда нужен** `BEGIN ... END`, потому что это синтаксис функции.

```sql
CREATE OR REPLACE FUNCTION before_insert_user()
RETURNS TRIGGER AS $$
BEGIN
    NEW.created_at := NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER before_insert_user
BEFORE INSERT ON users
FOR EACH ROW EXECUTE FUNCTION before_insert_user();
```

---

#### 🔹 **3. SQL Server (T-SQL)**

* Триггер может содержать **несколько операторов** без блока `BEGIN ... END`, но если ты хочешь сгруппировать их (например, в `IF`), то `BEGIN ... END` нужен:

```sql
CREATE TRIGGER trg_UpdateUser
ON users
AFTER UPDATE
AS
BEGIN
    PRINT 'User updated';
    UPDATE logs SET updated_at = GETDATE();
END
```

---

#### 🔹 **4. Oracle (PL/SQL)**

* В Oracle внутри тела триггера **всегда используется** `BEGIN ... END`, даже если одно выражение:

```sql
CREATE OR REPLACE TRIGGER before_insert_user
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    :NEW.created_at := SYSDATE;
END;
```

---

### 🧩 Итог:

| СУБД           | BEGIN ... END обязательно?              | Пример |
| -------------- | --------------------------------------- | ------ |
| **MySQL**      | Только если больше одного выражения     | ✅      |
| **PostgreSQL** | Всегда (в функции)                      | ✅      |
| **SQL Server** | Рекомендуется, но не строго обязательно | ⚙️     |
| **Oracle**     | Всегда                                  | ✅      |

---

Хочешь — я покажу конкретно, **как правильно оформить триггер в твоей СУБД** (MySQL, PostgreSQL, SQL Server, Oracle)?
Скажи, какую ты используешь.
