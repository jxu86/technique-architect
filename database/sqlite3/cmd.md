


```bash
sqlite3
sqlite>.open test.db  # test.db存在就打开数据库，否则创建数据库
sqlite>.show
sqlite>.header on
sqlite>.mode column
sqlite>.timer on
sqlite>.databases
sqlite>CREATE TABLE points(id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, name CHAR(50)  NOT NULL, guild_id INT NOT NULL ,user_id INT NOT NULL unique,points INT NOT NULL);
sqlite>.tables
sqlite>PRAGMA table_info("points");     # 查看表字段信息

# 导出csv
sqlite> .headers on
sqlite> .mode csv
sqlite> .output data.csv
sqlite> SELECT * FROM table_name;
sqlite> .output stdout // 输出重定向


sqlite3 whitelist.sqlite .dump > backup.sql
```