## DDL
### CREATE TABLE person_info(id INT NOT NULL auto_increment,name VARCHAR(100) NOT NULL,birthday DATE NOT NULL,phone_number CHAR(11) NOT NULL,country varchar(100) NOT NULL,PRIMARY KEY (id),KEY idx_name_birthday_phone_number (name, birthday, phone_number));

## DML(验证B+Tree)

### INSERT INTO test.person_info (id, name, birthday, phone_number, country) VALUES(1, '1', '2022-12-12', '', '');

### explain SELECT * FROM person_info WHERE birthday = '1990-09-27' AND phone_number = '15123983239' and name = 'Ashburn'; --可以走所有的索引，即使顺序没有按照联合索引（非聚簇索引）顺序，也会被优化器优化，与按照索引顺序一致

### explain SELECT * FROM person_info WHERE id = 1 and (name = '1' and birthday = '2022-12-12' and phone_number ='1'); -- 优先走聚簇索引执行

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND phone_number = '15123983239';

### explain SELECT * FROM person_info WHERE name LIKE '%As%';

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow';

### explain SELECT * FROM person_info WHERE id = 1;

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000';

### explain SELECT * FROM person_info WHERE name = 'Ashburn' and birthday = '1980-01-01' AND phone_number > '15100000000';

### explain SELECT * FROM person_info ORDER BY name, birthday, phone_number limit 10;

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow';

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow' AND birthday > '1980-01-01'; --在B+Tree中只能用到name索引，因为name条件查询结果不固定，所以birthday使用不到索引

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000'; --同上，在B+Tree中只能使用到name与birthday索引，所以phone_number走不到索引
