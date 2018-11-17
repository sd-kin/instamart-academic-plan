---
layout: post
title:  "Тезисы"
subject: "Молинаро - SQL Сборник рецептов"
date:   2018-11-11 20:55:21 +0300
---

## Mollinaro - SQL Cookbook notes

### Группировка
  
  > Способ организации подобных строк. Каждая строка результирующего множества представляет одну или более строк с одинаковыми значениями в одном или более заданных полях.

  В математике основным определением группы является тройка (G, •, e), где G – множество, • – бинарная операция над G, и e – член G. (бинарная операция — математическая операция, принимающая два аргумента и возвращающая один результат)

  SQL группа определяется как пара (G, e), где G – результирующее множество самостоятельного или самодостаточного запроса, в котором используется оператор GROUP BY, e – член G, и выполняются
  следующие аксиомы:

  * Для каждого e в G e является уникальным и представляет один или более экземпляров e.
  * Для каждого e в G агрегатная функция COUNT возвращает значение > 0.

  (е можно считать строкой результирующего множества, поскольку фактически эти строки представляют собой группы)

  Следствия:

    * Группы не могут быть пустыми
    * Группы уникальны

### Оконные функции

  выполняют агрегацию заданного набора (группы) строк, но вместо того чтобы возвращать по одному значению на группу могут возвращать несколько значений для каждой группы. Группа строк, подвергающаяся агрегации называется "окно"

  оконные функции выполняются как последний шаг в обработке SQL перед оператором `ORDER BY`

  Для определения сегмента или группы строк, подвергающихся агрегации, используется оператор `PARTITION BY`. Оператор `PARTITION BY` можно рассматривать как «скользящий` GROUP BY`», потому что в отличие от обычного `GROUP BY` группы, создаваемые `PARTITION BY`, в результирующем множестве не являются уникальными. `PARTITION BY` может использоваться для вычисления агрегата заданной группы строк (отсчет начинается заново для каждой новой группы), и тогда будут представлены все экземпляры этого значения в таблице (все члены каждой группы), а не одна группа.

  ```SQL
    select ename, deptno, count(*) over(partition by deptno) as cnt
    from emp
    order by 2
  ```

  ENAME | DEPTNO | CNT
  ----- | ------ | --- 
  CLARK | 10 | 3
  KING  | 10 | 3
  SMITH | 20 | 5
  FORD  | 20 | 5
  SCOTT | 20 | 5
  ALLEN | 30 | 6
  BLAKE | 30 | 6
  JAMES | 30 | 6
  TURNER | 30 | 6

  аналогичный результат можноа получить без оконных функций с помощью

  ```SQL
    select e.ename,
           e.deptno,
           (select count(*) from emp d where e.deptno=d.deptno) as cnt
    from emp e
    order by 2
  ```

  Оператор `PARTITION BY` выполняет вычисления независимо от других оконных функций, осуществляя сегментирование по другим столбцам в том же выражении `SELECT`.


  ```SQL
    select ename,
           deptno,
           count(*) over(partition by deptno) as dept_cnt,
           job,
           count(*) over(partition by job) as job_cnt
    from emp
    order by 2
  ```


  ENAME | DEPTNO | DEPT_CNT | JOB | JOB_CNT 
  ----- | ------ | -------- | --- | -------     
  MILLER | 10 | 3 | CLERK | 4 
  CLARK | 10 | 3 | MANAGER | 3 
  KING | 10 | 3 | PRESIDENT | 1 
  SCOTT | 20 | 5 | ANALYST | 2 
  FORD | 20 | 5 | ANALYST | 2 
  SMITH | 20 | 5 | CLERK | 4 
  JONES | 20 | 5 | MANAGER | 3 
  ADAMS | 20 | 5 | CLERK | 4 
  JAMES | 30 | 6 | CLERK | 4 
  MARTIN | 30 | 6 | SALESMAN | 4 
  TURNER | 30 | 6 | SALESMAN | 4 
  WARD | 30 | 6 | SALESMAN | 4 
  ALLEN | 30 | 6 | SALESMAN | 4 
  BLAKE | 30 | 6 | MANAGER | 3 


  Оператор `ORDER BY`, используемый в операторе `OVER` оконной функции, определяет две вещи:
    
    1. Как упорядочены строки в сегменте.
    2. Какие строки участвуют в вычислениях.

  ```SQL
    select deptno,
           ename,
           hiredate,
           sal,
           sum(sal)over(order by hiredate) as running_total
    from emp
    where deptno=10
  ```

  DEPTNO | ENAME | HIREDATE | SAL | RUNNING_TOTAL
  ------ | ----- | -------- | --- | -------------     
  10 | CLARK | 09JUN1981 | 2450 | 2450
  10 | KING | 17NOV1981 | 5000 | 7450
  10 | MILLER | 23JAN1982 | 1300 | 8750

  считается сумма от начала окна до текущей строки

  Следующий запрос аналогичен предыдущему, но в нем поведение по умолчанию, являющееся результатом применения `ORDER BY HIRE DATE`, явно задается оператором `RANGE BETWEEN`

  ```SQL
    select deptno,
           ename,
           hiredate,
           sal,
           sum(sal)over(order by hiredate range between unbounded preceding and current row) as running_total
    from emp
    where deptno=10
  ```

  Оператор `RANGE BETWEEN` ANSI называет оператором кадрирования

  `ORDER BY` определяет порядок вычисления и также подразумевает кадрирование по умолчанию

  ```SQL
    select deptno,
           ename,
           sal,
           sum(sal)over(order by hiredate range between unbounded preceding and current row) as run_total1,
           sum(sal)over(order by hiredate rows between 1 preceding and current row) as run_total2,
           sum(sal)over(order by hiredate range between current row and unbounded following) as run_total3,
           sum(sal)over(order by hiredate rows between current row and 1 following) as run_total4
    from emp
    where deptno=10
  ```

  DEPTNO | ENAME | SAL | RUN_TOTAL1 | RUN_TOTAL2 | RUN_TOTAL3 | RUN_TOTAL4
  ------ | ----- | --- | ---------- | ---------- | ---------- | ----------       
  10 | CLARK | 2450 | 2450 | 2450 | 8750 | 7450
  10 | KING | 5000 | 7450 | 7450 | 6300 | 6300
  10 | MILLER | 1300 | 8750 | 6300 | 1300 | 1300

  **RUN_TOTAL2** - Вместо ключевого слова `RANGE` в данном операторе кадрирования используется `ROWS`; это означает, что кадр, или окно, будет создано из некоторого количества строк. `1 PRECEDING` говорит о том, что кадр будет начинаться со строки, стоящей непосредственно перед текущей строкой. Диапазон распространяется до `CURRENT ROW`. Таким образом, **RUN_TOTAL2** – это сумма заработных плат текущего и предыдущего, на основании `HIREDATE`, сотрудников.

  **RUN_TOTAL3** - Оконная функция для вычисления **RUN_TOTAL3** выполняет обратное тому, что делалось для **RUN_TOTAL1**. Суммирование начинается с текущей строки и включает не все предыдущие строки, а все последующие строки.

  **RUN_TOTAL4** - Инверсия **RUN_TOTAL2**. Суммирование начинается с текущей строки и включает не одну предыдущую, а одну следующую строку.


### 1 Извлечение записей (стр. 33)

#### Извлечение всех строк и столбцов из таблицы

```SQL
  select * from emp
```

#### Извлечение подмножества строк из таблицы (стр. 34)

```SQL
  select * from emp where deptno = 10
```

#### Выбор строк по нескольким условиям (стр. 34)

```SQL
  select * from emp where deptno = 10 or comm is not null or sal <= 2000 and deptno = 20
```

#### Извлечение подмножества столбцов из таблицы (стр. 35)

```SQL
  select ename, deptno, sal from emp
```

#### Как задать столбцам значимые имена (стр. 36)

```SQL
  select sal as salary, comm as commission from emp 
```


SALARY | COMMISSION
------ | ----------- 
800    |
1600   | 300
1250   | 500
2975   |
1250   | 1300
2850   |
2450   |
3000   |
5000   |
1500   | 0
1100   |
950    |
3000   |
1300   |


#### Обращение к столбцу в предикате WHERE по псевдониму (стр. 37)

Чтобы обратиться к столбцу по псевдониму, необходимо использовать вложенный запрос:

```SQL
  select *
  from (
    select sal as salary, comm as commission
    from emp
  ) x
  where salary < 5000
```

Предикат `WHERE` обрабатывается раньше оператора `SELECT`, оператор `FROM` выполняется до предиката `WHERE`. Размещение исходного запроса в операторе `FROM` обеспечивает формирование его результатов до обработки самого внешнего `WHERE`


#### Конкатенация значений столбцов (стр. 38)

```SQL
  select concat(ename, ' WORKS AS A ',job) as msg
  from
  where deptno=10
```

#### Использование условной логики в выражении SELECT (стр. 39)

```SQL
  select ename,
         sal,
         case 
           when sal <= 2000 then 'UNDERPAID'
           when sal >= 4000 then 'OVERPAID'
           else 'OK'
         end as status
  from emp
```

#### Ограничение числа возвращаемых строк (стр. 40)

```SQL
  select * from emp limit 5
```
  

#### Возвращение n случайных записей таблицы (стр. 42)

```SQL
  select ename,job
  from emp
  order by rand()
  limit 5
```

#### Поиск значений NULL (стр. 44)

```SQL
  select * from emp where comm is null
```

#### Преобразование значений NULL в неNULL значения (стр. 44)

```SQL
  select coalesce(comm,0) from emp
```

Функция COALESCE принимает в качестве аргументов одно или более значений. Функция возвращает первое не NULL значение из списка.

#### Поиск по шаблону (стр. 45)

```SQL
  select ename, job
  from emp
  where deptno in (10,20)
  and (ename like '%I%' or job like '%ER')
```

### 2 Сортировка результатов запроса (стр. 47)

#### Возвращение результатов запроса в заданном порядке

```SQL
  select ename,job,sal
  from emp
  where deptno = 10
  order by sal asc
```

#### Сортировка по нескольким полям (стр. 48)

```SQL
  select empno,deptno,sal,ename,job
  from emp
  order by deptno, sal desc
```

#### Сортировка по подстрокам (стр. 49)

```SQL
  select ename,job
  from emp
   order by substr(job,length(job)-2,2)
```

#### Сортировка смешанных буквенноцифровых данных (стр. 50)

Требуется сортировать смешанные буквенно-цифровые данные по числовой или символьной части.

|DATA|
|--------|
|SMITH 20|
|ALLEN 30|
|WARD 30|
|JONES 20|
|MARTIN 30|
|BLAKE 30|
|CLARK 10|
|SCOTT 20|
|KING 10|
|TURNER 30|
|ADAMS 20|
|JAMES 30|
|FORD 20|
|MILLER 10|

Oracle и PostgreSQL

```SQL
  /* СОРТИРУЕМ ПО DEPTNO */
  select data
  from V
  order by replace(data, /* убираем строковую часть, остаётся только номер */
    replace( /* убираем все #, остаётся строковая часть */
      translate( /* замена цифр на # */
        data,'0123456789','##########'),
        '#',
        ''
      ),
    '')
  
  /* СОРТИРУЕМ ПО ENAME */
  select data
  from emp
  order by replace(translate(data,'0123456789','##########'),'#','')
```

MySQL и SQL Server

В настоящее время данные платформы не поддерживают функцию
TRANSLATE, таким образом, решения для этой задачи нет.

#### Обработка значений NULL при сортировке (стр. 53)

В зависимости от того, как должны быть представлены данные (и как конкретная СУБД сортирует значения `NULL`), столбцы, допускающие неопределенные значения, можно сортировать по возврастанию или по убыванию:

```SQL
  select ename,sal,comm
  from emp
  order by 3

  select ename,sal,comm
  from emp
  order by 3 desc
```

Если необходимо сортировать неопределенные значения иначе, чем определенные, например расположить определенные значения по убыванию или возрастанию, а все значения `NULL` вывести поcле

```SQL
  select ename,sal,comm
  from (
    select ename,sal,comm,
    case when comm is null then 0 else 1 end as is null
    from emp
  ) x
  order by is_null desc, comm
```

#### Сортировка по зависящему от данных ключу (стр. 60)

если значение JOB – «SALESMAN», сортировка должна осуществляться по столбцу COMM; в противном случае сортируем по SAL.

```SQL
  select ename,sal,job,comm
  from emp
  order by case when job = 'SALESMAN' then comm else sal end
```

### 3 Работа с несколькими таблицами (стр. 62)

#### Размещение одного набора строк под другим (стр. 62)

```SQL
  select ename as ename_and_dname, deptno
  from emp
  where deptno = 10
  
  union all
  
  select '-----------', null from t1
  
  union all
  select dname, deptno
  from dept
```


ENAME_AND_DNAME |  DEPTNO
--------------- |  ------
CLARK |  10
KING |  10
MILLER |  10
------------ | 
ACCOUNTING |  10
RESEARCH |  20
SALES |  30
OPERATIONS |  40

#### Объединение взаимосвязанных строк (стр. 64)

```SQL
  select e.ename, d.loc
  from emp e, dept d
  where e.deptno = d.deptno
  and e.deptno = 10
```

```SQL
  select e.ename, d.loc
  from emp e
    inner join dept d on (e.deptno = d.deptno)
  where e.deptno = 10
```

> Объединение – это операция, в результате которой строки двух таблиц соединяются в одну.

> Эквиобъединение – это объединение, в котором условием объединения является равенство


####  Поиск одинаковых строк в двух таблицах (стр. 66)

```SQL
  select * from V
```

ENAME | JOB | SAL
----- | --- | ---  
SMITH | CLERK | 800
ADAMS | CLERK | 1100
JAMES | CLERK | 950
MILLER | CLERK | 1300


```SQL
  select e.empno,e.ename,e.job,e.sal,e.deptno
  from emp e, V
  where e.ename = v.ename
    and e.job = v.job
    and e.sal = v.sal
```

```SQL  
  select e.empno,e.ename,e.job,e.sal,e.deptno
  from emp e join V
    on ( e.ename = v.ename
      and e.job = v.job
      and e.sal = v.sal )
```

##### DB2, Oracle и PostgreSQL

Если нет необходимости возвращать столбцы представления V, можно использовать операцию над множествами INTERSECT в сочетании с предикатом IN:

```SQL
  select empno,ename,job,sal,deptno
  from emp
  where (ename,job,sal) in (
    select ename,job,sal from emp
    intersect
    select ename,job,sal from V
  )
```

#### Извлечение из одной таблицы значений, которых нет в другой таблице (стр. 68)

##### DB2 and PostgreSQL

```SQL
  select deptno from dept
  except
  select deptno from emp
```

##### Oracle

```SQL
  select deptno from dept
  minus
  select deptno from emp
```

##### MySQL и SQL Server

```SQL
  select deptno
  from dept
  where deptno not in (select deptno from emp)
```
При использовании оператора `NOT IN` не забывайте о значениях `NULL`. По сути, `IN` и `NOT IN` – операции логического ИЛИ. Формируемый ими результат зависит от того, как интерпретируются значения `NULL` при вычислении логического ИЛИ. В SQL выражению «`TRUE or NULL`» соответствует `TRUE`, а «`FALSE or NULL`» – `NULL`! Полученный результат `NUL`L обеспечит `NULL` при последующих вычислениях. Во избежание проблем с `NOT IN` и значениями `NULL` применяются связанные подзапросы в сочетании с предикатом `NOT EXISTS`.

```SQL 
  select d.deptno
  from dept d
  where not exists ( 
    select null
    from emp e
    where d.deptno = e.deptno )
```

#### Извлечение из таблицы строк, для которых нет соответствия в другой таблице (стр. 72)

```SQL
select d.*
from dept d
  left outer join emp e on (d.deptno = e.deptno)
where e.deptno is null
```

#### Независимое добавление объединений в запрос (стр. 74)

Для получения дополнительной информации без утраты данных, возвращенных в результате исходного запроса, можно использовать внешнее объединение.

```SQL
  select e.ename, d.loc, eb.received
  from emp e 
    join dept d on (e.deptno=d.deptno)
    left join emp_bonus eb on (e.empno=eb.empno)
  order by 2
```

#### Выявление одинаковых данных в двух таблицах (стр. 76)

Функции, осуществляющие вычитание множеств (`MINUS` или `EXCEPT`, в зависимости от СУБД), существенно упрощают задачу по сравнению таблиц. Если ваша СУБД не поддерживает таких функций, можно использовать связанный подзапрос.

```SQL 
  select * from V
```

EMPNO | ENAME | JOB | MGR | HIREDATE | SAL | COMM | DEPTNO
----- | ----- | --- | --- | -------- | --- | ---- | ------      
7369 | SMITH | CLERK | 7902 | 17DEC1980 | 800 | 20 |
7499 | ALLEN | SALESMAN | 7698 | 20FEB1981 | 1600 | 300 | 30
7521 | WARD | SALESMAN | 7698 | 22FEB1981 | 1250 | 500 | 30
7566 | JONES | MANAGER | 7839 | 02APR1981 | 2975 | 20 |
7654 | MARTIN | SALESMAN | 7698 | 28SEP1981 | 1250 | 1400 | 30
7698 | BLAKE | MANAGER | 7839 | 01MAY1981 | 2850 | 30 |
7788 | SCOTT | ANALYST | 7566 | 09DEC1982 | 3000 | 20 |
7844 | TURNER | SALESMAN | 7698 | 08SEP1981 | 1500 | 0 | 30
7876 | ADAMS | CLERK | 7788 | 12JAN1983 | 1100 | 20 |
7900 | JAMES | CLERK | 7698 | 03DEC1981 | 950 | 30 |
7902 | FORD | ANALYST | 7566 | 03DEC1981 | 3000 | 20 |
7521 | WARD | SALESMAN | 7698 | 22FEB1981 | 1250 | 500 | 30

Требуется выяснить, имеются ли в этом представлении такие же данные, как и в таблице EMP.


##### DB2 и PostgreSQL

```SQL
  (
    select empno, ename , job, mgr, hiredate, sal, comm, deptno, count(*) as cnt
    from V
    group by empno, ename, job, mgr, hiredate, sal, comm, deptno
    
    except
    
    select empno, ename, job, mgr, hiredate, sal, comm, deptno, count(*) as cnt
    from emp
    group by empno, ename, job, mgr, hiredate, sal, comm, deptno
  )
     
  union all
     
  (
    select empno, ename, job, mgr, hiredate, sal, comm, deptno, count(*) as cnt
    from emp
    group by empno, ename, job, mgr, hiredate, sal, comm, deptno
    
    except
    
    select empno, ename, job, mgr, hiredate, sal, comm, deptno, count(*) as cnt
    from v
    group by empno, ename, job, mgr, hiredate, sal, comm, deptno
  )
```

##### MySQL и SQL Server

```SQL
  select *
  from (
    select e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno, count(*) as cnt
    from emp e
    group by empno, ename, job, mgr, hiredate, sal, comm, deptno
  ) e
  where not exists (
    select null
    from (
      select v.empno, v.ename, v.job, v.mgr, v.hiredate, v.sal, v.comm, v.deptno, count(*) as cnt
      from v
      group by empno, ename, job, mgr, hiredate, sal, comm, deptno
    ) v
    where v.empno = e.empno
      and v.ename = e.ename
      and v.job = e.job
      and v.mgr = e.mgr
      and v.hiredate = e.hiredate
      and v.sal = e.sal
      and v.deptno = e.deptno
      and v.cnt = e.cnt
      and coalesce(v.comm,0) = coalesce(e.comm,0)
  )

  union all

  select *
  from (
    select v.empno, v.ename, v.job, v.mgr, v.hiredate, v.sal, v.comm, v.deptno, count(*) as cnt
    from v
    group by empno, ename, job, mgr, hiredate, sal, comm, deptno
  ) v
  where not exists (
    select null
    from (
      select e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno, count(*) as cnt
      from emp e
      group by empno, ename, job, mgr, hiredate, sal, comm, deptno
    ) e
    where v.empno = e.empno
      and v.ename = e.ename
      and v.job = e.job
      and v.mgr = e.mgr
      and v.hiredate = e.hiredate
      and v.sal = e.sal
      and v.deptno = e.deptno
      and v.cnt = e.cnt
      and coalesce(v.comm,0) = coalesce(e.comm,0)
  )
```

#### Идентификация и устранение некорректного использования декартова произведения (стр. 83)

Требуется возвратить имя каждого служащего 10!го отдела и местонахождение отдела.

```SQL
  select e.ename, d.loc
  from emp e, dept d
  where e.deptno = 10
    and d.deptno = e.deptno
```

Обычно, чтобы не происходило прямого (декартова) произведения, применяется правило n–1, где n представляет количество таблиц в FROM, а n–1 – минимальное число объединений, необходимое во избежание прямого произведения.


#### Осуществление объединений при использовании агрегатных функций (стр. 85)

Необходимо быть очень аккуратным при совместном использовании агрегатных функций и объединений. Обычно избежать ошибок, обусловленных дублированием строк, возникающим при объединении, можно двумя способами: или просто использовать в вызове агрегатной функции ключевое слово `DISTINCT`, чтобы в вычислении участвовали только уникальные экземпляры значений, или сначала провести агрегацию (во вложенном запросе), а потом объединение.

##### MySQL и PostgreSQL

```SQL
  select deptno,
         sum(distinct sal) as total_sal,
         sum(bonus) as total_bonus
  from (
    select e.empno,
           e.ename,
           e.sal,
           e.deptno,
           e.sal*case when eb.type = 1 then .1
            when eb.type = 2 then .2
            else .3
          end as bonus
    from emp e, emp_bonus eb
    where e.empno = eb.empno
      and e.deptno = 10
  ) x
  group by deptno
```

##### DB2, Oracle и SQL Server

```SQL
  select distinct deptno, total_sal, total_bonus
  from (
    select e.empno,
           e.ename,
           sum(distinct e.sal) over (partition by e.deptno) as total_sal,
           e.deptno,
           sum(e.sal*case when eb.type = 1 then .1
            when eb.type = 2 then .2
            else .3 end) over(partition by deptno) as total_bonus
    from emp e, emp_bonus eb
    where e.empno = eb.empno
    and e.deptno = 10
  ) x
```

#### Внешнее объединение при использовании агрегатных функций (стр. 90)

```SQL
  select deptno,
         sum(distinct sal) as total_sal,
         sum(bonus) as total_bonus
  from (
    select e.empno,
     e.ename,
     e.sal,
     e.deptno,
     e.sal*case when eb.type is null then 0
      when eb.type = 1 then .1
      when eb.type = 2 then .2
      else .3
    end as bonus
    from emp e left outer join emp_bonus eb
      on (e.empno = eb.empno)
    where e.deptno = 10
  )
  group by deptno
```

Можно также использовать оконную функцию `SUM OVER`:

```SQL
  select distinct deptno, total_sal, total_bonus
  from (
    select e.empno,
      e.ename,
      sum(distinct e.sal) over(partition by e.deptno) as total_sal,
      e.deptno,
      sum(e.sal*case when eb.type is null then 0
        when eb.type = 1 then .1
        when eb.type = 2 then .2
        else .3
      end) over (partition by deptno) as total_bonus
  from emp e left outer join emp_bonus eb
    on (e.empno = eb.empno)
  where e.deptno = 10
  ) x
```

#### Возвращение отсутствующих данных из нескольких таблиц (стр. 93)

```SQL
  select d.deptno,d.dname,e.ename
  from dept d full outer join emp e
    on (d.deptno=e.deptno)
```

```SQL
  select d.deptno, d.dname, e.ename
    from dept d right outer join emp e
      on (d.deptno=e.deptno)

  union

  select d.deptno,d.dname,e.ename
  from dept d left outer join emp e
    on (d.deptno=e.deptno)
```

#### Значения NULL в операциях и сравнениях (стр. 97)

```SQL
  select ename,comm
  from emp
  where coalesce(comm,0) < ( select comm
    from emp
    where ename = 'WARD' )
```

### Вставка, обновление, удаление (стр. 99)

#### Вставка новой записи (стр. 100)

```SQL
  insert into dept (deptno, dname, loc) values (50, 'PROGRAMMING', 'BALTIMORE')
```

#### Вставка значений по умолчанию (стр. 100)

```SQL
  insert into D values (default)
```

#### Переопределение значения по умолчанию значением NULL (стр. 102)

```SQL
  insert into d (id, foo) values (null, 'Brighten')
```

#### Копирование строк из одной таблицы в другую (стр. 103)

```SQL
  insert into dept_east (deptno, dname, loc)
    select deptno, dname, loc
    from dept
    where loc in ( 'NEW YORK', 'BOSTON' )
```

#### Копирование описания таблицы (стр. 103)

##### DB2

```SQL
  create table dept_2 like dept
```

##### Oracle, MySQL и PostgreSQL

```SQL
  create table dept_2 as select * from dept where 1 = 0
```

#### Вставка в несколько таблиц одновременно (стр. 104)

##### Oracle

```SQL
  insert all
    when loc in ('NEW YORK','BOSTON') then
      into dept_east (deptno,dname,loc) values (deptno,dname,loc)
    when loc = 'CHICAGO' then
      into dept_mid (deptno,dname,loc) values (deptno,dname,loc)
    else
      into dept_west (deptno,dname,loc) values (deptno,dname,loc)
  select deptno,dname,loc
  from dept
```

##### MySQL, PostgreSQL и SQL Server

На момент написания данной книги эти производители не поддерживали вставки в несколько таблиц.

#### Блокировка вставки в определенные столбцы (стр. 106)

Создайте представление, базированное на данной таблице, содержащее столбцы, доступные для вставки. Затем обеспечьте, чтобы вся вставка осуществлялась только посредством этого представления.

```SQL
  create view new_emps as
    select empno, ename, job
    from emp
```

#### Изменение записей в таблице (стр. 107)

```SQL
  update emp
    set sal = sal*1.10
  where deptno = 20
```

#### Обновление в случае существования соответствующих строк в другой таблице (стр. 109)

```SQL
  update emp
  set sal=sal*1.20
  where empno in ( select empno from emp_bonus )
```

#### Обновление значениями из другой таблицы (стр. 109)

##### DB2 и MySQL

```SQL
  update emp e set (e.sal,e.comm) = (select ns.sal, ns.sal/2
    from new_sal ns
    where ns.deptno=e.deptno)
  where exists ( select null
    from new_sal ns
    where ns.deptno = e.deptno )
```

##### PostgreSQL

```SQL
  update emp
  set sal = ns.sal, comm = ns.sal/2
  from new_sal ns
  where ns.deptno = emp.deptno
```

#### Слияние записей (стр. 113)

В настоящее время Oracle является единственной СУБД1, имеющей выражение для решения этой задачи. Это выражение – `MERGE`

```SQL
  merge into emp_commission ec
  using (select * from emp) emp on (ec.empno=emp.empno)
    when matched then
      update set ec.comm = 1000
      delete where (sal < 2000)
    when not matched then
      insert (ec.empno,ec.ename,ec.deptno,ec.comm)
      values (emp.empno,emp.ename,emp.deptno,emp.comm)
```

#### Удаление всех записей из таблицы (стр. 115)

```SQL
  delete from emp
```

#### Удаление определенных записей (стр. 116)

```SQL
  delete from emp where deptno = 10
```

#### Удаление одной записи (стр. 116)

```SQL
  delete from emp where empno = 7782
```

#### Удаление записей, которые нарушают ссылочную целостность (стр. 117)

Требуется удалить записи из таблицы, если они ссылаются на несуществующие записи другой таблицы.

```SQL
  delete from emp
  where not exists (
    select * from dept
    where dept.deptno = emp.deptno
  )
```

```SQL
  delete from emp
  where deptno not in (select deptno from dept)
```

#### Уничтожение дублирующихся записей (стр. 118)

```SQL
  delete from dupes
  where id not in ( select min(id)
    from dupes
    group by name )
```

#### Удаление записей, на которые есть ссылки в другой таблице (стр. 119)

```SQL
  delete from emp
  where deptno in ( select deptno
    from dept_accidents
    group by deptno
    having count(*) >= 3 )
```

### 5 Запросы на получение метаданных (стр. 121)

#### Как получить список таблиц схемы (стр. 121)

```SQL
  select table_name
  from information_schema.tables
  where table_schema = 'SMEAGOL'
```

#### Как получить список столбцов таблицы (стр. 122)

```SQL
  select column_name, data_type, ordinal_position
  from information_schema.columns
  where table_schema = 'SMEAGOL'
  and table_name = 'EMP'
```


#### Как получить список индексированных столбцов таблицы (стр. 123)

#### MySQL

```SQL
  select a.tablename,a.indexname,b.column_name
  from pg_catalog.pg_indexes a, information_schema.columns b
  where a.schemaname = 'SMEAGOL'
  and a.tablename = b.table_name
```

#### PostgreSQL

```SQL
  show index from emp
```

#### Как получить список ограничений, наложенных на таблицу (стр. 125)

##### PostgreSQL, MySQL и SQL Server

```SQL
  select a.table_name,
    a.constraint_name,
    b.column_name,
    a.constraint_type
  from information_schema.table_constraints a,
    information_schema.key_column_usage b
  where a.table_name = 'EMP'
    and a.table_schema = 'SMEAGOL'
    and a.table_name = b.table_name
    and a.table_schema = b.table_schema
    and a.constraint_name = b.constraint_name
```

### 6 Работа со строками (стр. 134)

#### Проход строки (стр. 135)

```SQL
  select substr(e.ename,iter.pos,1) as C
    from (select ename from emp where ename = 'KING') e,
      (select id as pos from t10) iter where iter.pos <= length(e.ename)
```

#### Как вставить кавычки в строковые литералы (стр. 135)

```SQL
  select 'g''day mate' qmarks from t1
```

```SQL
  select 'beavers'' teeth' from t1
```

```SQL
  select '''' from t1
```

#### Как подсчитать, сколько раз символ встречается в строке (стр. 138)

Чтобы определить количество запятых в строке, `10,CLARK,MANAGER` вычтем ее длину без
запятых из исходной длины.

```SQL
  select (length('10,CLARK,MANAGER') - length(replace('10,CLARK,MANAGER',',','')))/length(',')
  as cnt
  from t1
```

#### Удаление из строки ненужных символов (стр. 139)

##### MySQL и SQL Server

```SQL
  select ename,
    replace(
      replace(
        replace(
          replace(
            replace(ename,'A',''),'E',''),'I',''),'O',''),'U','')
    as stripped1,
    sal,
    replace(sal,0,'') stripped2
  from emp
```

##### Oracle и PostgreSQL

```SQL
  select ename, replace(translate(ename,'AEIOU','aaaaa'),'a')
    as stripped1,
    sal,
    replace(sal,0,'') as stripped2
  from emp
```

#### Разделение числовых и символьных данных (стр. 141)

##### PostgreSQL

```SQL
  select replace(
    translate(data,'0123456789','0000000000'),'0','') as ename,
    cast(
      replace(
        translate(lower(data),
          'abcdefghijklmnopqrstuvwxyz',
          rpad('z',26,'z')),'z','') as integer) as sal
  from (
    select ename||sal as data
    from emp
  ) x
```


#### Извлечение инициалов из имени (стр. 150)

##### MySQL

```SQL
  select case
    when cnt = 2 then
      trim(trailing '.' from
        concat_ws('.', substr(substring_index(name,' ',1),1,1), substr(name,
          length(substring_index(name,' ',1))+2,1),
          substr(substring_index(name,' ',1),1,1),
          '.'))
    else
      trim(trailing '.' from
        concat_ws('.',
          substr(substring_index(name,' ',1),1,1),
          substr(substring_index(name,' ',1),1,1)
        )
      )
    end as initials
  from (
    select name,length(name)length(replace(name,' ','')) as cnt
    from (
      select replace('Stewie Griffin','.','') as name from t1
    )y
  )x
```


##### Oracle и PostgreSQL

```SQL
  select replace(
    replace(
      translate(replace('Stewie Griffin', '.', ''),
      'abcdefghijklmnopqrstuvwxyz',
    rpad('#',26,'#') ), '#','' ),' ','.' ) ||'.'
  from t1
```

#### Упорядочивание по частям строки (стр. 154)

Требуется упорядочить строки по последним двум символам имени

```SQL
  select ename
  from emp
  order by substr(ename,length(ename)1,2)
```

#### Упорядочивание по числу в строке (стр. 155)

##### PostgreSQL

```SQL
  select data
  from V
  order by
  cast(
    replace(
    translate(data,
      replace(
        translate(data,'0123456789','##########'),
          '#',''),rpad('#',20,'#')),'#','') as integer)
```
##### MySQL и SQL Server

На момент написания данной книги ни один из данных производителей не поддерживает функцию TRANSLATE.


#### Создание списка с разделителями из строк таблицы (стр. 161)

##### MySQL

```SQL
  select deptno,
    group_concat(ename order by empno separator ',') as emps
  from emp
  group by deptno
```

#### Преобразование данных с разделителями в список оператора IN со множеством значений (стр. 168)

##### MySQL

```SQL
  select empno, ename, sal, deptno
  from emp
  where empno in (
    select substring_index(
      substring_index(list.vals,',',iter.pos),',',1) empno
    from (select id pos from t10) as iter,
      (select '7654,7698,7782,7788' as vals
      from t1) list
      where iter.pos <= (length(list.vals)length(replace(list.vals,',','')))+1
  ) x
```

#### Упорядочение строки в алфавитном порядке (стр. 174)

##### MySQL

```SQL
  select ename, group_concat(c order by c separator '')
  from (
    select ename, substr(a.ename,iter.pos,1) c
    from emp a,
    ( select id pos from t10 ) iter
    where iter.pos <= length(a.ename)
  ) x
  group by ename
```

#### Выявление строк, которые могут быть интерпретированы как числа (стр. 180)

##### MySQL

```SQL
  select cast(group_concat(c order by pos separator '') as unsigned) as MIXED1
  from (
    select v.mixed, iter.pos, substr(v.mixed,iter.pos,1) as c
    from V,
    ( select id pos from t10 ) iter
      where iter.pos <= length(v.mixed) and ascii(substr(v.mixed,iter.pos,1)) between 48 and 57
    ) y
  group by mixed
  order by 1
```

#### Извлечение nной подстроки с разделителями (стр. 187)

##### MySQL

```SQL
  select name
  from (
    select iter.pos,
      substring_index(
        substring_index(src.name,',',iter.pos),',',1) name
    from V src,
      (select id pos from t10) iter,
      where iter.pos <= length(src.name)length(replace(src.name,',',''))
    ) x
  where pos = 2
```

### 7 Работа с числами

#### Вычисление среднего

```SQL
  select avg(sal) as avg_sal from emp
```

#### Поиск минимального/максимального значения столбца

```SQL
  select min(sal) as min_sal, max(sal) as max_sal from emp
```

#### Вычисление суммы значений столбца

```SQL
  select sum(sal) from emp
```

#### Вычисление моды (стр. 213)

мода (mode) в математике – это наиболее часто встречающийся элемент рассматриваемого множества данных

```SQL
  select sal
  from emp
  where deptno = 20
  group by sal
  having count(*) >= all ( select count(*)
    from emp
    where deptno = 20
    group by sal )
```


#### Вычисление медианы (стр. 216)

медиана (median) – это значение среднего члена множества упорядоченных элементов

```SQL
  select avg(sal)
  from (
    select e.sal
    from emp e, emp d
    where e.deptno = d.deptno and e.deptno = 20
    group by e.sal
    having sum(case when e.sal = d.sal then 1 else 0 end) >= abs(sum(sign(e.sal  d.sal)))
  )
```

#### Вычисление доли от целого в процентном выражении (стр. 220)

```SQL
  select (sum(case when deptno = 10 then sal end)/sum(sal))*100 as pct
  from emp
```

#### Агрегация столбцов, которые могут содержать NULL значения (стр. 223)

```SQL
  select avg(coalesce(comm,0)) as avg_comm
  from emp
  where deptno=30
```

#### Вычисление среднего без учета наибольшего и наименьшего значений (стр. 224)

```SQL
select avg(sal)
from emp
where sal not in (
  (select min(sal) from emp),
  (select max(sal) from emp)
)
```

#### Преобразование буквенноцифровых строк в числа (стр. 226)

##### Oracle и PostgreSQL

```SQL
  select cast(
    replace(
      translate( 'paul123f321', 'abcdefghijklmnopqrstuvwxyz', rpad('#',26,'#')),'#','') as integer ) as num
  from t1
```

##### MySQL and SQL Server
Решение не предоставляется, поскольку на момент написания данной книги ни один из этих производителей не поддерживает функцию `TRANSLATE`.


#### Изменение значений в текущей сумме (стр. 228)

```SQL
  select
    case when v1.trx = 'PY'
      then 'PAYMENT'
      else 'PURCHASE'
    end as trx_type,
    v1.amt,
    (select sum(
      case when v2.trx = 'PY'
      then v2.amt else v2.amt
    end
    )
  from V v2
  where v2.id <= v1.id) as balance
  from V v1
```

### 8 Арифметика дат


#### Добавление и вычитание дней, месяцев и лет (стр. 231)

##### MySQL

```SQL

  select hiredate - interval 5 day as hd_minus_5D,
    hiredate + interval 5 day as hd_plus_5D,
    hiredate - interval 5 month as hd_minus_5M,
    hiredate + interval 5 month as hd_plus_5M,
    hiredate - interval 5 year as hd_minus_5Y,
    hiredate + interval 5 year as hd_plus_5Y
  from emp
  where deptno=10

```

#### Определение количества дней между двумя датами (стр. 234)

##### MySQL

```SQL
  select datediff(day,allen_hd,ward_hd)
  from (
    select hiredate as ward_hd
    from emp
    where ename = 'WARD'
  ) x,
  (
    select hiredate as allen_hd
    from emp
    where ename = 'ALLEN'
  ) y
```

#### Определение количества рабочих дней между двумя датами (стр. 236)

Для формирования необходимого числа строк (дней) между двумя датами используйте сводную таблицу T500. Затем подсчитайте количество дней, не являющихся выходными. Добавление дней к каждой дате реализуйте с помощью функции `DATE_ADD`. Воспользуйтесь функцией `DATE_FORMAT` (формат даты) для получения названия дня недели для каждой даты:

##### MySQL

```SQL
  select sum(
    case when date_format(date_add(jones_hd, interval t500.id1 DAY),'%a') in ( 'Sat','Sun' )
    then 0 else 1
  end) as days
  from (
    select 
      max(case when ename = 'BLAKE'
        then hiredate
      end) as blake_hd,
      max(case when ename = 'JONES'
        then hiredate
      end) as jones_hd
    from emp
  where ename in ( 'BLAKE','JONES' )
  ) x,
  t500
  where t500.id <= datediff(blake_hd,jones_hd)+1
```

#### Определение количества месяцев или лет между двумя датами (стр. 241)

##### MySQL

```SQL
  select mnth, mnth/12
  from (
    select (year(max_hd)  year(min_hd))*12 + (month(max_hd)  month(min_hd)) as mnth
    from (
      select min(hiredate) as min_hd, max(hiredate) as max_hd
      from emp
    ) x
  ) y
```

#### Как подсчитать, сколько раз в году повторяется каждый день недели (стр. 246)

Чтобы найти, сколько раз повторяется каждый день недели за год, необходимо:
1. Сгенерировать все возможные даты года.
2. Отформатировать даты таким образом, чтобы в них был указан со!
ответствующий день недели.
3. Подсчитать, сколько раз за год встречается название каждого дня
недели.


#### Определение интервала времени в днях между текущей и следующей записями (стр. 258)

```SQL
  select x.*, datediff(day,x.hiredate,x.next_hd) diff
  from (
    select e.deptno, e.ename, e.hiredate,
      (select min(d.hiredate) from emp d
        where d.hiredate > e.hiredate) next_hd
        from emp e
    where e.deptno = 10
  ) x
```

### 9 Работа с датами (стр. 264)

#### Как определить, является ли год високосным (стр. 265)

проверяется последний день февраля; если он является 29м днем, то текущий год – високосный.

```SQL
  select day(
    last_day(
      date_add(
        date_add(
          date_add(current_date,
            interval dayofyear(current_date) day),
              interval 1 day),
                interval 1 month))) dy
  from t1
```

#### Как определить количество дней в году (стр. 272)

1. Определение первого дня текущего года.
2. Добавление одного года к этой дате (для получения первого дня следующего года).
3. Вычитание текущего года из результата шага 2.

```SQL
  select datediff((curr_year + interval 1 year),curr_year)
  from (
    select adddate(current_date,dayofyear(current_date)+1) curr_year
    from t1
  ) x
```

#### Извлечение единиц времени из даты (стр. 275)

##### MySQL

```SQL
  select date_format(current_timestamp,'%k') hr,
    date_format(current_timestamp,'%i') min,
    date_format(current_timestamp,'%s') sec,
    date_format(current_timestamp,'%d') dy,
    date_format(current_timestamp,'%m') mon,
    date_format(current_timestamp,'%Y') yr
  from t1
```

#### Определение первого и последнего дней месяца (стр. 277)

##### MySQL

```SQL
  select date_add(current_date,
    interval -day(current_date)+1 day) firstday,
    last_day(current_date) lastday
  from t1
```

#### Выбор всех дат года, выпадающих на определенный день недели (стр. 280)


##### MySQL

```SQL
  select dy
  from (
    select adddate(x.dy,interval t500.id1 day) dy
    from (
      select dy, year(dy) yr
        from (
          select adddate(
            adddate(current_date,
             interval dayofyear(current_date) day),
               interval 1 day ) dy
          from t1
        ) tmp1
      ) x,
      t500
    where year(adddate(x.dy,interval t500.id1 day)) = x.yr
    ) tmp2
  where dayname(dy) = 'Friday'
```

#### Определение дат первого и последнего появления заданного дня недели (стр. 287)

##### MySQL

Используйте функцию `ADDDATE`, чтобы получить первый день месяца. После этого путем простых вычислений над числовыми значениями дней недели (Вс.–Сб. соответствуют 1–7) находите первый и последний понедельники текущего месяца:

```SQL
  select first_monday,
    case month(adddate(first_monday,28))
    when mth then adddate(first_monday,28)
    else adddate(first_monday,21)
  end last_monday
  from (
    select case sign(dayofweek(dy)2)
      when 0 then dy
      when 1 then adddate(dy,abs(dayofweek(dy)2))
      when 1 then adddate(dy,(7(dayofweek(dy)2)))
    end first_monday, mth
    from (
      select adddate(adddate(current_date,day(current_date)),1) dy, month(current_date) mth
      from t1
    ) x
  ) y
```

#### Создание календаря (стр. 295)

##### MySQL

```SQL
  select max(case dw when 2 then dm end) as Mo,
    max(case dw when 3 then dm end) as Tu,
    max(case dw when 4 then dm end) as We,
    max(case dw when 5 then dm end) as Th,
    max(case dw when 6 then dm end) as Fr,
    max(case dw when 7 then dm end) as Sa,
    max(case dw when 1 then dm end) as Su
  from (
    select date_format(dy,'%u') wk,
      date_format(dy,'%d') dm,
      date_format(dy,'%w')+1 dw
    from (
      select adddate(x.dy,t500.id1) dy,
        x.mth
      from (
        select adddate(current_date,dayofmonth(current_date)+1) dy,
          date_format(
          adddate(current_date,
          dayofmonth(current_date)+1),
          '%m') mth
        from t1
      ) x, t500
    where t500.id <= 31 and date_format(adddate(x.dy,t500.id1),'%m') = x.mth ) y ) z
  group by wk
  order by wk
```
 
#### Получение дат начала и конца кварталов года (стр. 314)

##### MySQL

```SQL
  select quarter(adddate(dy,1)) QTR, date_add(dy,interval 3 month) Q_start, adddate(dy,1) Q_end
  from (
    select date_add(dy,interval (3*id) month) dy
    from (
      select id, adddate(current_date,dayofyear(current_date)+1) dy
      from t500
      where id <= 4
    ) x
  ) y
```

#### Определение дат начала и окончания заданного квартала (стр. 320)

Год и квартал заданы в формате YYYYQ (четыре разряда – год, один разряд – квартал)

##### MySQL

```SQL
  select date_add(adddate(q_end,day(q_end)+1),
    interval 2 month) q_start,
    q_end
  from (
    select last_day(str_to_date(concat(substr(yrq,1,4),mod(yrq,10)*3),'%Y%m')) q_end
    from (
      select 20051 as yrq from t1 union all
      select 20052 as yrq from t1 union all
      select 20053 as yrq from t1 union all
      select 20054 as yrq from t1
    ) x
  ) y
```

#### Дополнение отсутствующих дат (стр. 327)

##### MySQL

```SQL
  select z.mth, count(e.hiredate) num_hired
  from (
    select date_add(min_hd,interval t500.id1 month) mth
    from (
      select min_hd, date_add(max_hd,interval 11 month) max_hd
      from (
        select adddate(min(hiredate),dayofyear(min(hiredate))+1) min_hd,
          adddate(max(hiredate),dayofyear(max(hiredate))+1) max_hd
        from emp
      ) x
    ) y, t500
  where date_add(min_hd,interval t500.id1 month) <= max_hd) z 
  left join emp e on (z.mth = adddate(date_add(last_day(e.hiredate),interval 1 month),1))
  group by z.mth
  order by 1
```

#### Поиск по заданным единицам времени (стр. 337)

##### DB2 и MySQL

```SQL
  select ename
  from emp
  where monthname(hiredate) in ('February','December') or dayname(hiredate) = 'Tuesday'
```

#### Сравнение строк по определенной части даты (стр. 339)

##### MySQL

```SQL
  select concat(a.ename, ' was hired on the same month and weekday as ', b.ename) msg
  from emp a, emp b
  where date_format(a.hiredate,'%w%M') = date_format(b.hiredate,'%w%M')
    and a.empno < b.empno
  order by a.ename
```

#### Выявление наложений диапазонов дат (стр. 342)

##### MySQL

```SQL
  select a.empno,a.ename,concat('project ',b.proj_id,' overlaps project ',a.proj_id) as msg
  from emp_project a,emp_project b
  where a.empno = b.empno
    and b.proj_start >= a.proj_start
    and b.proj_start <= a.proj_end
    and a.proj_id != b.proj_id
```

### 10 Работа с диапазонами данных (стр. 348)

#### Поиск диапазона последовательных значений (стр. 348)

```SQL
  select v1.proj_id,
  v1.proj_start,
  v1.proj_end
  from V v1, V v2
  where v1.proj_end = v2.proj_start
```


#### Вычисление разности между значениями строк одной группы или сегмента (стр. 354)

##### DB2, MySQL, PostgreSQL и SQL Server

```SQL
  select deptno,ename,hiredate,sal,coalesce(cast(salnext_sal as char(10)),'N/A') as diff
  from (
    select e.deptno,e.ename,e.hiredate,e.sal,
      (select min(sal) from emp d
      where d.deptno=e.deptno
        and d.hiredate =
          (select min(hiredate) from emp d
            where e.deptno=d.deptno and d.hiredate > e.hiredate)) as next_sal
    from emp e
  ) x
```

##### Oracle

```SQL
  select deptno,ename,sal,hiredate,lpad(nvl(to_char(salnext_sal),'N/A'),10) diff
  from (
    select deptno,ename,sal,hiredate,lead(sal)over(partition by deptno order by hiredate) next_sal
  from emp
  )
```

#### Определение начала и конца диапазона последовательных значений (стр. 363)

```SQL
  create view v2
  as
  select a.*,
    case
      when (
        select b.proj_id
        from V b
        where a.proj_start = b.proj_end
      )
      is not null then 0 else 1
    end as flag
  from V a


  select proj_grp,min(proj_start) as proj_start,max(proj_end) as proj_end
  from (
  select a.proj_id,a.proj_start,a.proj_end,
    (select sum(b.flag)
      from V2 b
      where b.proj_id <= a.proj_id) as proj_grp
      from V2 a
    ) x
  group by proj_grp
```

#### Вставка пропущенных значений диапазона (стр. 368)

```SQL
  select y.yr, coalesce(x.cnt,0) as cnt
  from (
    select min_yearmod(cast(min_year as int),10)+rn as yr
    from (
      select (select min(extract(year from hiredate))
      from emp) as min_year,id1 as rn
  from t10) a) y
  left join
    (
      select extract(year from hiredate) as yr, count(*) as cnt
      from emp
      group by extract(year from hiredate)
    ) x
  on ( y.yr = x.yr )
```

#### Формирование последовательности числовых значений (стр. 373)

```SQL
  with x (id) as (
    select 1
    from t1

    union all

    select id+1
    from x
    where id+1 <= 10
  )

  select * from x
```

### Расширенный поиск (стр. 378)

#### Разбиение результирующего множества на страницы (стр. 378)

```SQL
  select sal
  from emp
  order by sal limit 5 offset 0
```

#### Как пропустить n строк таблицы (стр. 381)

```SQL
  select x.ename
  from (
    select a.ename,
      (select count(*)
      from emp b
      where b.ename <= a.ename) as rn
    from emp a
  ) x
  where mod(x.rn,2) = 1
```


#### Использование логики OR во внешних объединениях (стр. 384)

```SQL
  select e.ename, d.deptno, d.dname, d.loc
  from dept d left join emp e
    on (d.deptno = e.deptno
    and (e.deptno=10 or e.deptno=20))
  order by 2
```

#### Выявление строк со взаимообратными значениями (стр. 387)

```SQL
  select distinct v1.*
  from V v1, V v2
  where v1.test1 = v2.test2
    and v1.test2 = v2.test1
    and v1.test1 <= v1.test2
```

#### Как выбрать записи с nым количеством наивысших значений (стр. 389)

```SQL
select ename,sal
from (
  select (select count(distinct b.sal)
    from emp b
    where a.sal <= b.sal) as rnk,
  a.sal,a.ename
  from emp a
)
where rnk <= 5
```

#### Как найти записи с наибольшим и наименьшим значениями (стр. 391)

```SQL
  select ename
  from emp
  where sal in ( (select min(sal) from emp), (select max(sal) from emp) )
```

#### Сбор информации из последующих строк (стр. 393)

```SQL
select ename, sal, hiredate
from (
  select a.ename, a.sal, a.hiredate,
    (select min(hiredate) from emp b
    where b.hiredate > a.hiredate
      and b.sal > a.sal ) as next_sal_grtr,
    (select min(hiredate) from emp b
    where b.hiredate > a.hiredate) as next_hire
from emp a ) x
where next_sal_grtr = next_hire
```

#### Смещение значений строк (стр. 396)

```SQL
  select e.ename, e.sal,
    coalesce(
      (select min(sal) from emp d where d.sal > e.sal),
      (select min(sal) from emp)
    ) as forward,
    coalesce(
      (select max(sal) from emp d where d.sal < e.sal),
      (select max(sal) from emp)
    ) as rewind
  from emp e
  order by 2
```

#### Ранжирование результатов (стр. 400)

##### DB2, Oracle и SQL Server

```SQL
 select dense_rank() over(order by sal) rnk, sal from emp
```

##### MySQL и PostgreSQL

```SQL
  select (select count(distinct b.sal)
    from emp b
    where b.sal <= a.sal) as rnk,
    a.sal
  from emp a
```


#### Исключение дубликатов (стр. 401)

```SQL
  select distinct job from emp
```

```SQL
  select job from emp group by job
```

#### Ход конем (стр. 403)

Требуется получить множество, содержащее имя каждого служащего, отдел, в котором он работает, его заработную плату, дату его приема на работу и заработную плату сотрудника, принятого на работу последним в отделе.
  Значения столбца **LATEST_SAL** определяются в результате «хода конем», поскольку схема поиска их в таблице 
аналогична схеме перемещения шахматного коня.

```SQL
  select e.deptno,
    e.ename,
    e.sal,
    e.hiredate,
    (select max(d.sal)
    from emp d
    where d.deptno = e.deptno
      and d.hiredate =
      (select max(f.hiredate)
      from emp f
      where f.deptno = e.deptno)) as latest_sal
  from emp e
  order by 1, 4 desc
```

#### Формирование простых прогнозов (стр. 411)

Исходя из текущих данных требуется получить дополнительные строки и столбцы, представляющие будущие действия.
Для каждой строки результирующего множества требуется возвратить три строки (строка плюс две дополнительные строки для каждого заказа). Кроме дополнительных строк, должны быть добавлены столбцы с предполагаемыми датами обработки заказов.

##### PostgreSQL

```SQL
  select id, order_date, process_date,
    case when gs.n >= 2
      then process_date+1
      else null
    end as verified,
    case when gs.n = 3
      then process_date+2
      else null
    end as shipped
  from (
    select gs.id,
      current_date+gs.id as order_date,
      current_date+gs.id+2 as process_date
    from generate_series(1,3) gs (id)
  ) orders,
  generate_series(1,3)gs(n)
```

##### MySQL

MySQL не поддерживает функцию для автоматического формирования строк.


### 12 Составление отчетов и управление хранилищами данных (стр. 420)

#### Разворачивание результирующего множества в одну строку (стр. 420)

```SQL
  select sum(case when deptno=10 then 1 else 0 end) as deptno_10,
    sum(case when deptno=20 then 1 else 0 end) as deptno_20,
    sum(case when deptno=30 then 1 else 0 end) as deptno_30
  from emp
```

```SQL
  select max(case when deptno=10 then empcount else null end) as deptno_10
    max(case when deptno=20 then empcount else null end) as deptno_20,
    max(case when deptno=30 then empcount else null end) as deptno_30
  from (
    select deptno, count(*) as empcount
    from emp
    group by deptno
  ) x
```

#### Разворачивание результирующего множества в несколько строк (стр. 423)

```SQL
  select 
    max(case when job='CLERK'
      then ename else null end) as clerks,
    max(case when job='ANALYST'
      then ename else null end) as analysts,
    max(case when job='MANAGER'
      then ename else null end) as mgrs,
    max(case when job='PRESIDENT'
      then ename else null end) as prez,
    max(case when job='SALESMAN'
      then ename else null end) as sales
  from (
    select e.job,
      e.ename,
      (select count(*) from emp d
      where e.job=d.job and e.empno < d.empno) as rnk
      from emp e
    ) x
  group by rnk
```

#### Обратное разворачивание результирующего множества (стр. 431)

```SQL
  select dept.deptno,
    case dept.deptno
      when 10 then emp_cnts.deptno_10
      when 20 then emp_cnts.deptno_20
      when 30 then emp_cnts.deptno_30
    end as counts_by_dept
  from (
    select sum(case when deptno=10 then 1 else 0 end) as deptno_10,
      sum(case when deptno=20 then 1 else 0 end) as deptno_20,
      sum(case when deptno=30 then 1 else 0 end) as deptno_30
    from emp
  ) emp_cnts,
  (select deptno from dept where deptno <= 30) dept
```

#### Обратное разворачивание результирующего множества в один столбец (стр. 433)

##### DB2, Oracle и SQL Server

```SQL
  select case rn
    when 1 then ename
    when 2 then job
    when 3 then cast(sal as char(4))
  end emps
  from (
    select e.ename,e.job,e.sal,
      row_number()over(partition by e.empno
    order by e.empno) rn
    from emp e,
    (select *
    from emp where job='CLERK') four_rows
    where e.deptno=10
  ) x
```

##### PostgreSQL и MySQL

  Данный рецепт призван обратить внимание на применение ранжирующих функций для ранжирования строк,
которое затем используется при разворачивании таблицы. На момент написания данной книги ни PostgreSQL, ни MySQL не поддерживают ранжирующие функции.


#### Разворачивание результирующего множества для упрощения вычислений (стр. 440)

```SQL
  select d20_sal - d10_sal as d20_10_diff,
    d20_sal - d30_sal as d20_30_diff
  from (
    select sum(case when deptno=10 then sal end) as d10_sal,
      sum(case when deptno=20 then sal end) as d20_sal,
      sum(case when deptno=30 then sal end) as d30_sal
    from emp
  ) totals_by_dept
```

#### Создание блоков данных фиксированного размера (стр. 441)

##### DB2, Oracle и SQL Server

```SQL
  select ceil(row_number()over(order by empno)/5.0) grp, empno, ename from emp
```
##### PostgreSQL и MySQL 

```SQL
  select ceil(rnk/5.0) as grp, empno, ename
  from (
    select e.empno, e.ename,
    (select count(*) from emp d
    where e.empno < d.empno)+1 as rnk
    from emp e
  ) x
  order by grp
```

#### Создание заданного количества блоков (стр. 445)

##### Oracle и SQL Server

```SQL
  select ntile(4)over(order by empno) grp, empno, ename from emp
```

##### MySQL и PostgreSQL

```SQL
  select mod(count(*),4)+1 as grp,e.empno,e.ename
  from emp e, emp d
  where e.empno >= d.empno
  group by e.empno,e.ename
  order by 1
```

#### Создание горизонтальных гистограмм (стр. 451)

##### Oracle, PostgreSQL и MySQL

```SQL
  select deptno, lpad('*',count(*),'*') as cnt
  from emp
  group by deptno 
```

#### Создание вертикальных гистограмм (стр. 453)

##### PostgreSQL и MySQL

```SQL
  select max(deptno_10) as d10,
    max(deptno_20) as d20,
    max(deptno_30) as d30
  from (
    select case when e.deptno=10 then '*' else null end deptno_10,
      case when e.deptno=20 then '*' else null end deptno_20,
      case when e.deptno=30 then '*' else null end deptno_30,
      (select count(*) from emp d
      where e.deptno=d.deptno and e.empno < d.empno ) as rnk
    from emp e
  ) x
  group by rnk
  order by 1 desc, 2 desc, 3 desc
```

#### Как возвратить столбцы, не перечисленные в операторе GROUP BY (стр. 456)


```SQL
  select deptno,ename,job,sal,
    case when sal = max_by_dept
      then 'TOP SAL IN DEPT'
    when sal = min_by_dept
      then 'LOW SAL IN DEPT'
    end as dept_status,
    case when sal = max_by_job
      then 'TOP SAL IN JOB'
    when sal = min_by_job
      then 'LOW SAL IN JOB'
    end as job_status
  from (
    select e.deptno,e.ename,e.job,e.sal,
      (select max(sal) from emp d
      where d.deptno = e.deptno) as max_by_dept,
      (select max(sal) from emp d
      where d.job = e.job) as max_by_job,
      (select min(sal) from emp d
      where d.deptno = e.deptno) as min_by_dept,
      (select min(sal) from emp d
      where d.job = e.job) as min_by_job
    from emp e
  ) x
  where sal in (max_by_dept,max_by_job,
  min_by_dept,min_by_job)
```


#### Вычисление простых подсумм (стр. 462)

В данном рецепте под «простой подсуммой» подразумевается результирующее множество, содержащее значения, полученные в результате агрегации одного столбца, и общую сумму таблицы.

```SQL
  select coalesce(job,'TOTAL') job,
  sum(sal) sal
  from emp
  group by job with rollup
```


#### Вычисление подсумм для всех возможных сочетаний (стр. 466)

```SQL
  select deptno, job, 'TOTAL BY DEPT AND JOB' as category, sum(sal) as sal
  from emp
  group by deptno, job

  union all

  select null, job, 'TOTAL BY JOB', sum(sal)
  from emp
  group by job

  union all

  select deptno, null, 'TOTAL BY DEPT', sum(sal)
  from emp
  group by deptno

  union all

  select null,null,'GRAND TOTAL FOR TABLE', sum(sal)
  from emp
```


#### Использование выражений `CASE` для маркировки строк (стр. 478)

```SQL
  select ename,
    case when job = 'CLERK'
      then 1 else 0
    end as is_clerk,
    case when job = 'SALESMAN'
      then 1 else 0
    end as is_sales,
    case when job = 'MANAGER'
      then 1 else 0
    end as is_mgr,
    case when job = 'ANALYST'
      then 1 else 0
    end as is_analyst,
    case when job = 'PRESIDENT'
      then 1 else 0
    end as is_prez
  from emp
  order by 2,3,4,5,6
```


#### Создание разреженной матрицы (стр. 480)

```SQL
  select case deptno when 10 then ename end as d10,
    case deptno when 20 then ename end as d20,
    case deptno when 30 then ename end as d30,
    case job when 'CLERK' then ename end as clerks,
    case job when 'MANAGER' then ename end as mgrs,
    case job when 'PRESIDENT' then ename end as prez,
    case job when 'ANALYST' then ename end as anals,
    case job when 'SALESMAN' then ename end as sales
  from emp
```


#### Группировка строк по интервалам времени (стр. 481)

```SQL
  select ceil(trx_id/5.0) as grp,
    min(trx_date) as trx_start,
    max(trx_date) as trx_end,
    sum(trx_cnt) as total
  from trx_log
  group by ceil(trx_id/5.0)
```


#### Агрегация разных групп/сегментов одновременно (стр. 485)

```SQL
  select e.ename,
    e.deptno,
    (select count(*) from emp d
    where d.deptno = e.deptno) as deptno_cnt,
    job,
    (select count(*) from emp d
    where d.job = e.job) as job_cnt,
    (select count(*) from emp) as total
  from emp e
```


#### Агрегация скользящего множества значений (стр. 487)

```SQL
  select e.hiredate,e.sal,
    (select sum(sal) from emp d
  where d.hiredate between e.hiredate90
    and e.hiredate) as spending_pattern
  from emp e
  order by 1
```


### Иерархические запросы (стр. 500)

#### Представление отношений родительпотомок (стр. 501)


```SQL
  select concat(a.ename, ' works for ',b.ename) as emps_and_mgrs
  from emp a, emp b
  where a.mgr = b.empno
```


#### Представление отношений потомокродительпрародитель (стр. 505)

```SQL
  select a.ename||'>'||b.ename||'>'||c.ename as leaf___branch___root
  from emp a, emp b, emp c
  where a.ename = 'MILLER'
  and a.mgr = b.empno
  and b.mgr = c.empno
```


#### Создание иерархического представления таблицы (стр. 510)

##### MySQL

```SQL
  select emp_tree
  from (
    select ename as emp_tree
    from emp
    where mgr is null

  union

  select concat(a.ename,'  ',b.ename)
  from emp a
  join
    emp b on (a.empno=b.mgr)
  where a.mgr is null

  union

  select concat(a.ename,'  ',b.ename,'  ',c.ename)
  from emp a
  join
    emp b on (a.empno=b.mgr)
  left join
    emp c on (b.empno=c.mgr)
  where a.ename = 'KING'

  union

  select concat(a.ename,'  ',b.ename,'  ',c.ename,'  ',d.ename)
  from emp a
  join
    emp b on (a.empno=b.mgr)
  join
    emp c on (b.empno=c.mgr)
  left join
    emp d on (c.empno=d.mgr)
  where a.ename = 'KING'
  ) x
  where tree is not null
  order by 1
```


#### Выбор всех дочерних строк для заданной строки (стр. 519)

```SQL
/* находим EMPNO служащего JONES */
select ename,empno,mgr
from emp
where ename = 'JONES'
```

ENAME | EMPNO | MGR
----- | ----- | --- 
JONES | 7566 | 7839

```SQL
/* есть ли служащие, находящиеся в прямом подчинении у JONES? */
select count(*)
from emp
where mgr = 7566
```

COUNT(*) |
------- |
2 |

```SQL
/* у JONES двое подчиненных, найдем их EMPNO */
select ename,empno,mgr
from emp
where mgr = 7566
```
ENAME | EMPNO | MGR
----- | ----- | ---  
SCOTT | 7788 | 7566
FORD | 7902 | 7566

```SQL
/* есть ли подчиненные у SCOTT или FORD? */
select count(*)
from emp
where mgr in (7788,7902)
```

COUNT(*) |
------- |
2 |

```SQL
/* у SCOTT и FORD двое подчиненных, находим их EMPNO */
select ename,empno,mgr
from emp
where mgr in (7788,7902)
```

ENAME | EMPNO | MGR
----- | ----- | ---  
SMITH | 7369 | 7902
ADAMS | 7876 | 7788

```SQL
/* есть ли подчиненные у SMITH или ADAMS? */
select count(*)
from emp
where mgr in (7369,7876)
```

COUNT(*) |
-------  |
0        |

Теперь, когда известна глубина, можно приступать к обходу иерархии сверху вниз. Сначала дважды выполним рефлексивное объединение таблицы EMP. Затем произведем обратное разворачивание вложенного представления Х, чтобы преобразовать три столбца и две строки в один столбец и шесть строк

```SQL
  select distinct
    case t100.id
      when 1 then root
      when 2 then branch
      else leaf
    end as JONES_SUBORDINATES
  from (
    select a.ename as root,
      b.ename as branch,
      c.ename as leaf
    from emp a, emp b, emp c
    where a.ename = 'JONES'
      and a.empno = b.mgr
      and b.empno = c.mgr
  ) x,
  t100
  where t100.id <= 6
```

#### Определение узлов: ветвления, концевого, корневого (стр. 523)

```SQL
select e.ename,
  (select sign(count(*)) from emp d
    where 0 =
    (select count(*) from emp f
    where f.mgr = e.empno)) as is_leaf,
  (select sign(count(*)) from emp d
  where d.mgr = e.empno
    and e.mgr is not null) as is_branch,
  (select sign(count(*)) from emp d
  where d.empno = e.empno
    and d.mgr is null) as is_root
  from emp e
  order by 4 desc,3 desc
```


### 14 Всякая всячина  (стр. 532)

#### Создание отчетов с перекрестными ссылками с помощью оператора SQL Server `PIVOT` (стр. 532)

```SQL
  select [10] as dept_10,
    [20] as dept_20,
    [30] as dept_30,
    [40] as dept_40
  from (select deptno, empno from emp) driver
  pivot (
    count(driver.empno)
    for driver.deptno in ( [10],[20],[30],[40] )
  ) as empPivot
```

#### Обратное разворачивание отчета с помощью оператора SQL Server `UNPIVOT` (стр. 534)

```SQL
  select DNAME, CNT
  from (
    select [ACCOUNTING] as ACCOUNTING,
      [SALES] as SALES,
      [RESEARCH] as RESEARCH,
      [OPERATIONS] as OPERATIONS
    from (
      select d.dname, e.empno
      from emp e,dept d
      where e.deptno=d.deptno
 
  ) driver
  pivot (
    count(driver.empno)
    for driver.dname in
     ([ACCOUNTING],[SALES],[RESEARCH],[OPERATIONS])
  ) as empPivot
  ) new_driver
  unpivot (cnt for dname in (ACCOUNTING,SALES,RESEARCH,OPERATIONS)
  ) as un_pivot
```
