修改默认加密为 aes-256-cfb，

ALTER TABLE user MODIFY COLUMN method varchar(64) NOT NULL DEFAULT 'aes-256-cfb'

mysql 简单教程：

mysql -uUSERNAME -pPASSWD
use your_db
select * from table_name
ALTER TABLE user MODIFY COLUMN method varchar(64) NOT NULL DEFAULT 'aes-256-cfb'

https://github.com/orvice/ss-panel/issues/397

https://stackoverflow.com/questions/11312433/how-to-alter-a-column-and-change-the-default-value
