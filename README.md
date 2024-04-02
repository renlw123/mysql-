## innoDB结构


### File Header: 表示页的一些通用信息，占固定的38字节
#### FIL_PAGE_SPACE_OR_CHKSUM 4 字节 页的校验和（checksum值）
#### FIL_PAGE_OFFSET 4 字节 页号
#### FIL_PAGE_PREV 4 字节 上一个页的页号
#### FIL_PAGE_NEXT 4 字节 下一个页的页号
#### FIL_PAGE_LSN 8 字节 页面被最后修改时对应的日志序列位置（英文名是：Log SequenceNumber）
#### FIL_PAGE_TYPE 2 字节 该页的类型
#### FIL_PAGE_FILE_FLUSH_LSN 8 字节 仅在系统表空间的一个页中定义，代表文件至少被刷新到了对应的LSN值
#### FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID 4 字节 页属于哪个表空间

### Page Header:表示数据页专有的一些信息，占固定的56个字节
#### PAGE_N_DIR_SLOTS 2 字节 在页目录中的槽数量
#### PAGE_HEAP_TOP 2 字节 还未使用的空间最小地址，也就是说从该地址之后就是 Free Space
#### PAGE_N_HEAP 2 字节 本页中的记录的数量（包括最小和最大记录以及标记为删除的记录）
#### PAGE_FREE 2 字节 第一个已经标记为删除的记录地址（各个已删除的记录通过 next_record 也会组成一个单链表，这个单链表中的记录可以被重新利用）
#### PAGE_GARBAGE 2 字节 已删除记录占用的字节数
#### PAGE_LAST_INSERT 2 字节 最后插入记录的位置
#### PAGE_DIRECTION 2 字节 记录插入的方向
#### PAGE_N_DIRECTION 2 字节 一个方向连续插入的记录数量
#### PAGE_N_RECS 2 字节 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录）
#### PAGE_MAX_TRX_ID 8 字节 修改当前页的最大事务ID，该值仅在二级索引中定义
#### PAGE_LEVEL 2 字节 当前页在B+树中所处的层级
#### PAGE_INDEX_ID 8 字节 索引ID，表示当前页属于哪个索引
#### PAGE_BTR_SEG_LEAF 10 字节 B+树叶子段的头部信息，仅在B+树的Root页定义
#### PAGE_BTR_SEG_TOP 10 字节 B+树非叶子段的头部信息，仅在B+树的Root页定义
### Infimum + Supremum:两个虚拟的伪记录，分别表示页中的最小和最大记录，占固定的 26 个字节。
### User Records ：真实存储我们插入的记录的部分，大小不固定。
### Free Space ：页中尚未使用的部分，大小不确定。
### Page Directory ：页中的某些记录相对位置，也就是各个槽在页面中的地址偏移量，大小不固定，插入的记录越多，这个部分占用的空间越多。
### File Trailer ：用于检验页是否完整的部分，占用固定的8个字节。

![image](https://github.com/renlw123/mysql-/assets/74169518/d165a886-6152-45f9-a53d-ae16f417e54a) ![image](https://github.com/renlw123/mysql-/assets/74169518/91bd05fb-8302-4fd6-8205-4872f2d65927) ![image](https://github.com/renlw123/mysql-/assets/74169518/ad8c90d2-a27f-40ef-aaaa-4bd5e2775a3e)

![image](https://github.com/renlw123/mysql-/assets/74169518/7f98a0ef-4f8b-4b66-88f2-7a4c1dff507b) ![image](https://github.com/renlw123/mysql-/assets/74169518/caa35810-3914-4930-a3c8-d22a51ecfb6c) ![image](https://github.com/renlw123/mysql-/assets/74169518/0410dbba-68d6-48c0-b862-7ba28862590f)

### 页目录 ![image](https://github.com/renlw123/mysql-/assets/74169518/ca7b127e-32ed-49e5-8608-d2770f0f7ffa) ![image](https://github.com/renlw123/mysql-/assets/74169518/0107e695-303f-47a8-a892-0cd186fe81c5)

## DDL
### CREATE TABLE person_info(id INT NOT NULL auto_increment,name VARCHAR(100) NOT NULL,birthday DATE NOT NULL,phone_number CHAR(11) NOT NULL,country varchar(100) NOT NULL,PRIMARY KEY (id),KEY idx_name_birthday_phone_number (name, birthday, phone_number));

## DML(验证B+Tree)
### 结构导图![image](https://github.com/renlw123/mysql-/assets/74169518/60c869a6-b359-4d98-b7c5-13bdc3c98bba) ![image](https://github.com/renlw123/mysql-/assets/74169518/a629fd64-9986-4c01-a862-48133bcf2010)
### 真实情况![image](https://github.com/renlw123/mysql-/assets/74169518/f02164e0-6cbf-4126-9e5c-c39f10a49c3d) ![image](https://github.com/renlw123/mysql-/assets/74169518/e552abf4-a5d7-49ed-9078-94cba1200b8c)

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

### explain SELECT birthday FROM person_info ORDER BY  birthday   LIMIT 10; -- 也是可以走索引的，注意要关注查询字段与条件字段（这个查询字段属于联合索引）

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow'; -- 可以走索引，由于 B+ 树中的数据页和记录是先按 name 列排序的，所以我们上边的查询过程其实是这样的：找到 name 值为 Asa 的记录。找到 name 值为 Barlow 的记录。找到这些记录的主键值，再到 聚簇索引 中 回表 查找完整的记录。

### explain SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow' AND birthday > '1980-01-01'; --在B+Tree中只能用到name索引，因为name条件查询结果不固定，所以birthday使用不到索引

### explain SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000'; --同上，在B+Tree中只能使用到name与birthday索引，所以phone_number走不到索引

### explain SELECT * FROM person_info WHERE name = 'A' ORDER BY birthday, phone_number LIMIT 10; -- 也是可以走索引的

### explain SELECT * FROM person_info  ORDER BY birthday, phone_number LIMIT 10; -- 但是这样走不了索引（使用联合索引的各个排序列的排序顺序必须是一致的，ORDER BY name, birthday DESC如果一个asc,一个Desc 排序是走不了索引的）

### explain SELECT * FROM person_info WHERE country = 'China' ORDER BY name LIMIT 10; -- 这个查询只能先把符合搜索条件 country = 'China' 的记录提取出来后再进行排序，是使用不到索引。

### explain SELECT * FROM person_info WHERE name = 'A' ORDER BY birthday, phone_number LIMIT 10; -- 虽然这个查询也有搜索条件，但是 name = 'A' 可以使用到索引 idx_name_birthday_phone_number ，而且过滤剩下的记录还是按照 birthday 、 phone_number 列排序的，所以还是可以使用索引进行排序的。

### explain SELECT * FROM person_info ORDER BY UPPER(name), birthday , phone_number  LIMIT 10; -- 使用了函数就不再是单独的列了，所以是用不了索引

### explain SELECT name, birthday, phone_number FROM person_info WHERE name > 'Asa' AND name < 'Barlow'; -- 这样避免了回表，查询结果只包含索引就会省去在聚簇索引中再去查找一次    索引覆盖(查询列表里只包含索引列)

## 选择索引
### 列的基数：例如某一列包含1，2，3，1，2，3，1，2，3九条数据，那么列的基数为3，列的技术越大索引的利用率越好，所以尽量选择基数大的列作为索引
### 列的类型：索引列的类型越小越好，有 TINYINT 、 MEDIUMINT 、 INT 、 BIGINT这么几种，它们占用的存储空间依次递增，我们这里所说的 类型大小 指的就是该类型表示的数据范围的大小。能表示的整数范围当然也是依次递增，如果我们想要对某个整数列建立索引的话，在表示的整数范围允许的情况下，尽量让索引列使用较小的类型，比如我们能使用 INT 就不要使用 BIGINT ，能使用 MEDIUMINT 就不要使用INT ～ 这是因为：数据类型越小，在查询时进行的比较操作越快（这是CPU层次的东东）数据类型越小，索引占用的存储空间就越少，在一个数据页内就可以放下更多的记录，从而减少磁盘 I/O 带来的性能损耗，也就意味着可以把更多的数据页缓存在内存中，从而加快读写效率。这个建议对于表的主键来说更加适用，因为不仅是聚簇索引中会存储主键值，其他所有的二级索引的节点处都会存储一份记录的主键值，如果主键适用更小的数据类型，也就意味着节省更多的存储空间和更高效的 I/O 。
### 索引字符串前缀：KEY idx_name_birthday_phone_number (name(10), birthday, phone_number)， name(10) 就表示在建立的 B+ 树索引中只保留记录的前 10 个字符的编码，这种只索引字符串值的前缀的策略是我们非常鼓励的，尤其是在字符串类型能存储的字符比较多的时候。
### 索引列在表达式中单独出现：1. WHERE my_col * 2 < 42. WHERE my_col < 4/2第1个 WHERE 子句中 my_col 列并不是以单独列的形式出现的，而是以 my_col * 2 这样的表达式的形式出现的，存储引擎会依次遍历所有的记录，计算这个表达式的值是不是小于 4 ，所以这种情况下是使用不到为 my_col 列建立的 B+ 树索引的。而第2个 WHERE 子句中 my_col 列并是以单独列的形式出现的，这样的情况可以直接使用B+ 树索引。所以结论就是：如果索引列在比较表达式中不是以单独列的形式出现，而是以某个表达式，或者函数调用形式出现的话，是用不到索引的。

## 总结
### 上边只是我们在创建和使用 B+ 树索引的过程中需要注意的一些点。本集内容总结如下：1. B+ 树索引在空间和时间上都有代价，所以没事儿别瞎建索引。2. B+ 树索引适用于下边这些情况：全值匹配匹配左边的列匹配范围值精确匹配某一列并范围匹配另外一列用于排序用于分组3. 在使用索引时需要注意下边这些事项：只为用于搜索、排序或分组的列创建索引为列的基数大的列创建索引索引列的类型尽量小可以只对字符串值的前缀建立索引只有索引列在比较表达式中单独出现才可以适用索引为了尽可能少的让 聚簇索引 发生页面分裂和记录移位的情况，建议让主键拥有 AUTO_INCREMENT 属性。定位并删除表中的重复和冗余索引尽量使用 覆盖索引 进行查询，避免 回表 带来的性能损耗。
