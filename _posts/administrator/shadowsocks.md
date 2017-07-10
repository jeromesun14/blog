修改默认加密为 aes-256-cfb，
ALTER TABLE user MODIFY COLUMN method varchar(64) NOT NULL DEFAULT 'aes-256-cfb'
