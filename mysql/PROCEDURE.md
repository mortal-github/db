### 储存过程

#### 关于分隔符
`DELIMITER xx`将字符串xx作为新的分隔符。
1. 在mysql使用程序中创建储存过程要先改变分隔符，否则储存过程内的语句将被断开，导致无法创建储存过程。
2. 在mysql使用程序中结束创建储存过程要恢复分隔符，否则储存过程内的语句的分隔符不能起到分隔符作用。

#### 创建储存过程
语法：
```mysql
CREATE PROCEDURE procedure_name(parameter)
BEGIN
    sql statement
END;
```
1. `CREATE PROCEDURE`声明创建一个储存过程，后跟储存过程名。
2. 储存过程名后跟括号
3. 括号内包括参数。可以没有参数，没有参数也必须有括号。
4. `BEGIN END;`之间是储存过程的体。

示例
```msql
DELIMITER //

CREATE PROCEDURE productpricing()
BEGIN
    SELECT AVG(prod_price) AS priceaverage
    FROM products;
END //

DELIMITER ;
```

#### 删除储存过程
- `DROP PROCEDURE`语句删除储存过程，后跟储存过程名
```mysq
DROP PROCEDURE productpricing;
```

#### 使用参数
- 一般的，储存过程并不显示结果。
- 而是把结果返回给指定变量。

1. 参数声明包括3个部分
    - 用法：`IN`,`OUT`,`INOUT`
    - 参数名。
    - 参数数据类型。
2. 在MySQL钟，所有变量需要以`@`开始。（储存过程内部不用@）
3. 储存过程的代码位于`BEGIN`和`END`语句之间。
4. `SELECT ... INTO ... FROM ...`INTO关键字将查询结果放入变量。

示例：
```mysql
DELIMITER //

CREATE PROCEDURE productpricing(
    OUT pl DECIMAL(8,2),
    OUT ph DECIMAL(8,2),
    OUT pa DECIMAL(8,2)
)
BEGIN
    SELECT Min(prod_price)
    INTO pl
    FROM products;
    
    SELECT Max(prod_price)
    INTO ph
    FROM products;

    SELECT AVG(prod_price)
    INTO pa
    FROM products;
END //

CREATE PROCEDURE ordertotal(
    IN onumber INT,
    OUT ototal DECIMAL(8,2)
)
BEGIN
    SELECT SUM(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO ototal;
END //

DELIMITER ;
```

#### 执行储存过程
- `CALL`执行储存过程，后跟储存过程名括号内以@开始的参数变量。
- 调用时，语句并不显示结果
- 它返回以后可以显示（或在其他处理红使用）的变量。
```mysql
CALL productpricing(@pricelow, @pricehigh, @priceaverage);

CALL ordertotal(20005, @total);
```

- 获得值
```mysql
SELECT @pricelow, @pricehigh, @priceaverage;

 SELECT @total;
```

#### 建立智能储存过程
- 注释：`-- `开始
- `DECLARE`关键字声明局部变量，要求指定变量名和数据类型，其后可跟`DEFAULT`来指定默认值。

- `DECLARE`语句的发布存在特定次序，否则将会产生错误信息
    - 局部变量必须在任意游标和句柄之前定义
    - 句柄必须在游标之后定义。

- `COMMENT`关键字值不是不必需，它在`SHOW PROCEDURE STATUS`结果中显示。

- 逻辑控制语句：
    - `IF ... THEN ... END IF`
    - `IF ... THEN ... ELSEIF ... THEN ... END IF`
    - `IF ... THEN ... ELSE ... ENDIF`

- 循环： `REPEAT ... UNTIL done END REPEAT;`循环执行直到done为真。

- 循环条件： `DECLARE CONTINUE HANDLER FRO SQLSTATE '02000' SET done=1;` 语句在结束时设置done为真。
    - `CONTINUE HANDLER` 是在条件出现时被执行的代码，`SQLSTATE '02000`出现时，`SET done=`。
    - `SQLSTATE '020000`是一个未找到条件，当`REPEAT`由于没有更多的供循环而不能继续时，出现这个条件。
    

储存过程示例：
```mysql
DELIMITER //

-- Name: ordertotal
-- Parameters: onumber = order number
--             taxable = 0 if not taxable, 1 if taxable
--             ototal = order total variable

CREATE PROCEDURE ordertotal(
    IN onumber INT,
    IN taxable BOOLEAN,
    OUT ototal DECIMAL(8,2)
)COMMENT 'Obtain order total, optionally adding tax'
BEGIN

    -- Declare variable for total
    DECLARE total DECIMAL(8,2);
    -- Declare tax percentae
    DECLARE taxrate INT DEFAULT 6;

    -- Get the order total
    SELECT SUM(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO total;

    -- Is this taxable?
    IF taxable THEN
        -- yes, so add taxrate to the total
        SELECT total+(total/100*taxrate) INTO total;
    END IF;

    -- And finaly, save to out variable
    SELECT total INTO ototal;

END //

DELIMITER ;
```
结果示例：
```mysql
CALL ordertotal(20005, 0, @total);
SELECT @total;

CALL ordertotal(20005, 1, @total);
SELECT @total;

```

### 检查储存过程
```mysql
-- 显示创建一个储存过程的语句。
SHOW CREATE PROCEDURE ordertotal;

-- 获得所有储存过程的详细信息
SHOW PROCEDURE STATUS;

-- 用LIKE限制过程状态结果
SHOW PROCEDURE STATUS LIKE 'ordertotal';
```