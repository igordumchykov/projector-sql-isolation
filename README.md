# projector-sql-isolation
Example of transaction isolation levels and issues that may occur in [Postgres](https://www.postgresql.org/) and [MariaDB](https://mariadb.org/)

# Run
```shell
docker compose up -d
```

# Postgres

1. Enter docker container:
```shell
docker exec -it <container id> /bin/bash
```
2. Login into DB:
```shell
psql -U postgres
```
3. Create table:
```postgresql
create table if not exists users
(
    id   int primary key,
    name varchar not null,
    age  int     not null
);
```
4. Insert rows:
```postgresql
insert into users (id, name, age)
values (1, 'Alex', 10),
       (2, 'Bill', 20),
       (3, 'Michel', 30);
```
5. Change transaction isolation levels:
```postgresql
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

# MariaDB

1. Enter docker container:
```shell
docker exec -it <container id> /bin/bash
```
2. Login into DB:
```shell
mariadb -u<user> -p
#eneter password
```
3. Select db:
```mariadb
use users;
```
3. Create table:
```mariadb
create table if not exists users
(
    id   int primary key,
    name varchar(20) not null ,
    age  int not null
);
```
4. Insert rows:
```mariadb
insert into users (id, name, age)
values (1, 'Alex', 10),
       (2, 'Bill', 20),
       (3, 'Michel', 30);
```
5. Disable autocommit:
```mariadb
set autocommit = 0;
```
6. Change transaction isolation levels:
```mariadb
set global transaction isolation level read uncommitted;
set global transaction isolation level read committed;
set global transaction isolation level repeatable read;
set global transaction isolation level serializable;
```

# Scripts for Main DB Issues
## Dirty Reads
-- T1
`begin transaction;
select *
from users where id = 1;`

-- T2
`begin transaction;
update users set age=12 where id=1;`

-- T1
`select *
from users where id = 1;`

-- T2
`rollback;`

-- T1
`select *
from users where id = 1;
commit;`

## Non-repeatable Reads
-- T1
`begin transaction;
select *
from users where id = 1;`

-- T2
`begin transaction;
update users set age=12 where id=1;
commit;`

-- T1
`select *
from users where id = 1;
commit;`

## Phantom Reads
-- T1
`begin transaction;
select *
from users;`

-- T2
`begin transaction;
insert into users (id, name, age) values (7, 'Vlad', 35);
commit;`

-- T1
`select *
from users;
commit;`

## Lost Update
-- T1
`begin transaction;
select *
from users;
update users set age = 50 where id = 1;`

-- T2
`begin transaction;
update users set age = 100 where id = 1;
commit;`

-- T1
`commit;
select *
from users;`

# Postgres Isolation Levels And Issues
(+: may occur, -: may not occur)

Level|Dirty Reads|Non-repeatable Reads|Phantom Reads|Lost Update
-|-|-|-|-
Read Committed (Read Uncommitted)|-|+|+|+
Repeatable Read|-|-|-|-
Serializable|-|-|-|-

# Mariadb Isolation Levels And Issues
(+: may occur, -: may not occur)

Level|Dirty Reads|Non-repeatable Reads| Phantom Reads |Lost Update
-|-|-|---------------|-
Read Uncommitted|+|+| +             |+
Read Committed|-|+| +             |+
Repeatable Read|-|-| +             |-
Serializable|-|-| -             |-

# Stop Containers
```shell
docker compose down
```