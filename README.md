# Problemas de la entrevista
Adjunto el codigo echo en la entrevista estan echos con un "data" de prueba pero funciona igualmente  
igual se pueden resolver con recursion para sustituir los "for"  
* esta es la que tiene el codigo mayor a 25 solo sustituye el 3 y no se tiene por que multiplicar  
cada vez en el for por 1.8 con que se haga sobre el total es mas que suficiente y asi optimiza el codigo  
    ``` python
        data = [1, 2, 3, 4, 5]
        print(sum(list(filter(lambda n: n > 3, data)) * 1.8))
    ```  
* esta es la que tenia el codigo de "in" solo sustituye el "[1, 2, 8]" y "data"  
    ``` python
        data = [1, 2, 3, 4, 5]
        print(len(list(filter(lambda n: n in [1, 2, 8], data))))
    ```

# Seasons problem  
![Seasons problem](seasons%20problem.jpeg)  
* datos de prueba
    ``` sql
    CREATE TABLE customer_orders (
    ORD_ID INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    ORD_DT VARCHAR(30) NOT NULL,
    QT_ORDD VARCHAR(30) NOT NULL
    );
    INSERT INTO customer_orders (ORD_ID, ORD_DT, QT_ORDD)
    VALUES (113, '9/23/19' , 1),
        (114, '1/1/20' , 1 ),
        (115, '12/5/19' , 1 ),
        (116, '9/24/20' , 1 ),
        (117, '1/30/20' , 1 ),
        (118, '5/2/20' , 1 ),
        (119, '4/2/20' , 1 ),
        (120, '10/9/20' , 1 );
    ```
* codigo sql echo en mysql  
    ``` sql
    SELECT ORD_ID,
    case  
        when DATE_FORMAT(STR_TO_DATE(ORD_DT,'%m/%d/%y'),'%m-%d') BETWEEN '03-19' and '06-19' then "Sprint"
        when DATE_FORMAT(STR_TO_DATE(ORD_DT,'%m/%d/%y'),'%m-%d') BETWEEN '06-20' and '09-21' then "Summer" 
        when DATE_FORMAT(STR_TO_DATE(ORD_DT,'%m/%d/%y'),'%m-%d') BETWEEN '09-22' and '12-20' then "September" 
        when DATE_FORMAT(STR_TO_DATE(ORD_DT,'%m/%d/%y'),'%m-%d') BETWEEN '12-20' and '12-31' then "December" 
        when DATE_FORMAT(STR_TO_DATE(ORD_DT,'%m/%d/%y'),'%m-%d') BETWEEN '01-01' and '03-18' then "December" 
        ELSE "Error"
    end as SEASON  FROM customer_orders;
    ```
* mejorando el codigo pasandolo a una funcion 
    ``` sql
    DELIMITER $$
        CREATE FUNCTION SEASON (p_date varchar(100), p_format varchar(100)) 
        RETURNS varchar(100)
        DETERMINISTIC
        BEGIN 
            DECLARE season varchar(100);
            set season = case  
                when DATE_FORMAT(STR_TO_DATE(p_date,p_format),'%m-%d') BETWEEN '03-19' and '06-19' then "Sprint"
                when DATE_FORMAT(STR_TO_DATE(p_date,p_format),'%m-%d') BETWEEN '06-20' and '09-21' then "Summer" 
                when DATE_FORMAT(STR_TO_DATE(p_date,p_format),'%m-%d') BETWEEN '09-22' and '12-20' then "Fall" 
                when DATE_FORMAT(STR_TO_DATE(p_date,p_format),'%m-%d') BETWEEN '12-20' and '12-31' then "Winter" 
                when DATE_FORMAT(STR_TO_DATE(p_date,p_format),'%m-%d') BETWEEN '01-01' and '03-18' then "Winter" 
                ELSE "Error"
            end;
            RETURN season;
        END
    $$
    DELIMITER ;
    ```
* asi quedaria la consulta con la funcion creada pudiendo ser esta funcion reutilizada para cualquier otra consulta
    ``` sql
        SELECT ORD_ID,SEASON(ORD_DT,'%m/%d/%y') as SEASON FROM customer_orders;
    ```
# Detecting changes  
![Detecting changes](detecting%20changes.jpeg)  
* datos de prueba
    ``` sql
        CREATE TABLE weather (
            id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            my_date VARCHAR(30) NOT NULL,
            was_reiny BOOLEAN
        );
        INSERT INTO weather (my_date, was_reiny)
            VALUES ('1/1/20' , FALSE),
                ('1/2/20' , TRUE ),
                ('1/3/20' , TRUE),
                ('1/4/20' , FALSE ),
                ('1/5/20' , FALSE ),
                ('1/6/20' , TRUE ),
                ('1/7/20' , FALSE ),
                ('1/8/20' , TRUE ),
                ('1/9/20' , TRUE ),
                ('1/10/20' , TRUE );
    ```
* Se crea un vista para mayor comodidad y reutilizacion de la consulta
    ``` sql
    CREATE VIEW changes_weather AS
        SELECT wf_fecha, wf_was_reiny FROM (
            SELECT
                wf.fecha as wf_fecha,
                wf.was_reiny as wf_was_reiny,
                DATE_FORMAT(STR_TO_DATE(wa.my_date,'%m/%d/%y'),'%Y-%m-%d')  as wa_fecha,
                wa.was_reiny as wa_was_reiny
            FROM (SELECT DATE_FORMAT(STR_TO_DATE(my_date,'%m/%d/%y'),'%Y-%m-%d') AS fecha, was_reiny FROM weather where was_reiny=TRUE) wf
            INNER JOIN weather as wa
            ON  DATE_SUB(wf.fecha, INTERVAL 1 DAY) = DATE_FORMAT(STR_TO_DATE(wa.my_date,'%m/%d/%y'),'%Y-%m-%d')
        ) data_union WHERE wf_was_reiny=TRUE and wa_was_reiny=FALSE
    ```
* al final asi quedaria la consulta
    ``` sql
        SELECT * FROM changes_weather;
    ```

# customer orden status
![customer orden status](customer%20orden%20status.jpeg)  
* datos de prueba
    ``` sql
        CREATE TABLE customer_ord_lines (
            id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            order_number VARCHAR(30) NOT NULL,
            ithem_name VARCHAR(30) NOT NULL,
            mystatus VARCHAR(30) NOT NULL
        );
        INSERT INTO customer_ord_lines (order_number, ithem_name, mystatus)
        VALUES ('ORD_1567' ,'LAPTOP' , 'SHIPPED'),
                ('ORD_1567' ,'MOUSE' , 'SHIPPED'),
                ('ORD_1567' ,'KEYBOARD' , 'PENDING'),
                ('ORD_1234' ,'GAME' , 'SHIPPED'),
                ('ORD_1234' ,'BOOK' , 'CANCELLED'),
                ('ORD_1234' ,'BOOK' , 'CANCELLED'),
                ('ORD_9834' ,'SHIRT' , 'SHIPPED'),
                ('ORD_9834' ,'PANTS' , 'CANCELLED'),
                ('ORD_7654' ,'TV' , 'CANCELLED'),
                ('ORD_7654' ,'DVD' , 'CANCELLED');
    ```
* Se crea un vista para mayor comodidad y reutilizacion de la consulta
    ``` sql
    CREATE VIEW customer_ord_lines_mystatus AS
        SELECT order_number, CASE
            WHEN INSTR(GROUP_CONCAT(DISTINCT(mystatus) SEPARATOR ','), "PENDING") THEN "PENDING"
            WHEN INSTR(GROUP_CONCAT(DISTINCT(mystatus) SEPARATOR ','), "SHIPPED") THEN "SHIPPED"
            WHEN INSTR(GROUP_CONCAT(DISTINCT(mystatus) SEPARATOR ','), "CANCELLED") THEN "CANCELLED"
            ELSE "Error"
        END
	    FROM customer_ord_lines group by order_number;
    ```
* al final asi quedaria la consulta
    ``` sql
        select * from customer_ord_lines_mystatus;
    ```