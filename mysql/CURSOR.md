### 游标
使用游标的步骤
1. **声明游标**，即定义要使用的SELECT语句。
2. **打开游标**，以供使用，实际上就是用步骤1的SELEC语句检索出结果集合。
3. 对于填有数据的游标，根据需要**取出各行**。
4. 结束游标使用时，必须**关闭游标**。

游标的使用规则
1. 关闭游标后，如果没有重新打开，则不能使用。
2. 使用声明过的游标不需要再次声明，用`OPEN`语句打开即可。
3. `CLOSE`释放游标使用的所有内部内存和资源。因此在每个游标不再需要时都应该关闭。
4. 打开游标后，**`FETCH`语句**分别访问它的每一行。
    - `FETCH`语句指定检索什么数据（列），
    - 检索结果储存在什么地方，
    - 向前移动游标中的内部行指针，使下一条`FECTH`语句检索下一行。
5. `REPEAT ... UNTIL done END REPEAT;`循环执行直到done为真。
6. `DECLARE CONTINUE HANDLER FRO SQLSTATE '02000' SET done=1;` 语句在结束时设置done为真。
    - `CONTINUE HANDLER` 是在条件出现时被执行的代码，`SQLSTATE '02000`出现时，`SET done=`。
    - `SQLSTATE '020000`是一个未找到条件，当`REPEAT`由于没有更多的供循环而不能继续时，出现这个条件。
7. `DECLARE`语句的发布存在特定次序，否则将会产生错误信息
    - 局部变量必须在任意游标和句柄之前定义
    - 句柄必须在游标之后定义。

```mysql 
DECLARE name CURSOR
FOR 
select statement 
```语句命名游标，并定义相应的SELECT语句。

示例：注意这个游标局限于储存过程，储存结束后，游标也消失。
```mysql
DELIMITER //

CREATE PROCEDURE processorders()
BEGIN

    -- Declare local variables
    DECLARE done BOOLEAN DEFAULT 0;
    DECLARE o INT;
    DECLARE t DECIMAL(8,2);

    -- Declare the cursor 
    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM orders;

    -- Declare continue handler
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;

    -- Create a table store the results
    CREATE TABLE IF NOT EXISTS ordertotals(
        order_num INT,
        total DECIMAL(8,2)
    );


    -- open the cursor
    OPEN ordernumbers;


    -- Loop through all rows
    REPEAT 

        -- Get order number
        FETCH ordernumbers INTO o;

        -- Get the total for this order
        CALL ordertotal(o,1,t);

        -- Insert order and total into ordertotals
        INSERT 
        INTO ordertotals(order_num, total)
        VALUES(o,t);

    -- END of loop
    UNTIL done END REPEAT;

    -- Close the cursoe
    CLOSE ordernumbers;

END //

DELIMITER ;
```