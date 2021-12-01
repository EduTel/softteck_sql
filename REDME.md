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