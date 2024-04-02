## DDL
### CREATE TABLE person_info(id INT NOT NULL auto_increment,name VARCHAR(100) NOT NULL,birthday DATE NOT NULL,phone_number CHAR(11) NOT NULL,country varchar(100) NOT NULL,PRIMARY KEY (id),KEY idx_name_birthday_phone_number (name, birthday, phone_number));

## DML(验证B+Tree)

### INSERT INTO test.person_info (id, name, birthday, phone_number, country) VALUES(1, '1', '2022-12-12', '', '');

### explain SELECT * FROM person_info WHERE birthday = '1990-09-27' AND phone_number = '15123983239' and name = 'Ashburn'; -- 可以走所有的索引，即使顺序没有按照联合索引（非聚簇索引）顺序，也会被优化器优化，与按照索引顺序一致

### explain SELECT * FROM person_info WHERE id = 1 and (name = '1' and birthday = '2022-12-12' and phone_number ='1'); -- 优先走聚簇索引执行

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND phone_number = '15123983239'; -- 只能用到 name 列的索引， birthday 和 phone_number 的索引就用不上了，因为 name 值相同的记录先按照birthday 的值进行排序， birthday 值相同的记录才按照 phone_number 值进行排序。

### explain SELECT * FROM person_info WHERE name LIKE 'As%'; -- 是可以走索引的，一个排好序的字符串列其实有这样的特点：先按照字符串的第一个字符进行排序。如果第一个字符相同再按照第二个字符进行排序。如果第二个字符相同再按照第三个字符进行排序，依此类推。也就是说这些字符串的前n个字符，也就是前缀都是排好序的，所以对于字符串类型的索引列来说，我们只匹配它的前缀也是可以快速定位记录的，比方说我们想查询名字以 'As' 开头的记录

### explain SELECT * FROM person_info WHERE name LIKE '%As%'; -- name索引都走不到，因为name字段不确定

### explain SELECT * FROM person_info WHERE id = 1; -- 直接走聚簇索引

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000'; -- 1. 通过条件 name > 'Asa' AND name < 'Barlow' 来对 name 进行范围，查找的结果可能有多条 name 值不同的记录，2. 对这些 name 值不同的记录继续通过 birthday > '1980-01-01' 条件继续过滤。这样子对于联合索引 idx_name_birthday_phone_number 来说，只能用到 name 列的部分，而用不到 birthday 列的部分，因为只有 name 值相同的情况下才能用 birthday 列的值进行排序，而这个查询中通过 name 进行范围查找的记录中可能并不是按照 birthday 列进行排序的，所以在搜索条件中继续以 birthday 列进行查找时是用不到这个 B+ 树索引的。

### explain SELECT * FROM person_info WHERE name = 'Ashburn' and birthday = '1980-01-01' AND phone_number > '15100000000'; -- 1. name = 'Ashburn' ，对 name 列进行精确查找，当然可以使用 B+ 树索引了。2. birthday > '1980-01-01' AND birthday < '2000-12-31' ，由于 name 列是精确查找，所以通过 name ='Ashburn' 条件查找后得到的结果的 name 值都是相同的，它们会再按照 birthday 的值进行排序。所以此时对 birthday 列进行范围查找是可以用到 B+ 树索引的。3. phone_number > '15100000000' ，通过 birthday 的范围查找的记录的 birthday 的值可能不同，所以这个条件无法再利用 B+ 树索引了，只能遍历上一步查询得到的记录。

### explain SELECT * FROM person_info ORDER BY name, birthday, phone_number limit 10; -- 可以走索引，但是如果数据量小的话，mysql会认定走全表扫描效率高，可能不会走索引（但是必须按照索引顺序）

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow'; -- 可以走索引，由于 B+ 树中的数据页和记录是先按 name 列排序的，所以我们上边的查询过程其实是这样的：找到 name 值为 Asa 的记录。找到 name 值为 Barlow 的记录。找到这些记录的主键值，再到 聚簇索引 中 回表 查找完整的记录。

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow' AND birthday > '1980-01-01'; --在B+Tree中只能用到name索引，因为name条件查询结果不固定，所以birthday使用不到索引

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000'; --同上，在B+Tree中只能使用到name与birthday索引，所以phone_number走不到索引

### explain SELECT * FROM person_info WHERE name = 'A' ORDER BY birthday, phone_number LIMIT 10; -- 也是可以走索引的

### explain SELECT * FROM person_info  ORDER BY birthday, phone_number LIMIT 10; -- 但是这样走不了索引
