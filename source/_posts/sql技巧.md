---
title: sql技巧
date: 2019-08-23 18:24:38
tags:
---

对于数据过滤而言CHECK约束已经算是相当不错了。然而它仍存在一些缺陷，比如说它们是应用到表上面的，但有的时候你可能希望指定一条约束，而它只在特定条件下才生效。

使用SQL标准的WITH CHECK OPTION子句就能完成这点，至少Oracle和SQL Server都实现了这个功能。下面是实现方式：

    CREATE TABLE books (
      id    NUMBER(10)         NOT NULL,
      title VARCHAR2(100 CHAR) NOT NULL,
      price NUMBER(10, 2)      NOT NULL,
    
      CONSTRAINT pk_book PRIMARY KEY (id)
    );
    /
    
    CREATE VIEW expensive_books
    AS
    SELECT id, title, price
    FROM books
    WHERE price > 100
    WITH CHECK OPTION;
    /
    
    INSERT INTO books 
    VALUES (1, '1984', 35.90);
    
    INSERT INTO books 
    VALUES (
    , 
      'The Answer to Life, the Universe, and Everything',
    .90
    );
    
正如你看到的那样，expensive_books 是那些价格大于100块的书。这个视图只会返回第二本书：

SELECT * FROM expensive_books;

上述查询的输出是：

    ID TITLE                                       PRICE
    -- ----------------------------------------- -------
     The Answer to Life, the Universe, and ...   999.9 
不过由于我们使用了CHECK OPTION，我们还能防止用户往"昂贵的书籍"中插入那些廉价的。比如说，我们运行下这个查询：

    INSERT INTO expensive_books 
    VALUES (3, '10 Reasons why jOOQ is Awesome', 9.99);
它是无法生效的。你会看到：

    ORA-01402: view WITH CHECK OPTION where-clause violation

我们也无法将贵的书更新成便宜的：

    UPDATE expensive_books
    SET price = 9.99;
这个查询也会报出同样的ORA-01402错误。

    WITH CHECK OPTION内联
如果你需要局部防止脏数据被插入到表中，你可以使用WITH CHECK OPTION的内联子句：

    INSERT INTO (
      SELECT *
      FROM expensive_books
      WHERE price > 1000
      WITH CHECK OPTION
    ) really_expensive_books
    VALUES (3, 'Modern Enterprise Software', 999.99);
上述查询同样也会导到ORA-01402错误。

使用SQL转换来生成特殊约束
CHECK OPTION对于已存储的视图非常有用，它使得那些无权直接访问底层表的用户能够获得正确的授权，而内联的CHECK OPTION主要是在应用的SQL中间转换层来进行动态SQL的转换。

这个可以通过[jOOQ的SQL转换功能]http://www.jooq.org/doc/latest/manual/sql-building/queryparts/custom-sql-transformation/url)来完成，比如说，你可以在SQL语句中对某个表进行约束，从根本上阻止了非法DML的执行。如果你的数据库没有本地提供行级别的安全性的话，这也是一个实现多租户的不错的方式
