sqlite3 导出 csv 文件
======================

### 场景
把 `sqlite3` 一个表的数据导出为 `csv` 文件存储.

### 操作
```bash
$ sqlite3 xxx.db

sqlite> .headers on
sqlite> .mode csv
sqlite> .output data.csv
sqlite> select * from `TableName`;
sqlite> .quit
```

`.mode` 设置输出模式
- csv 逗号分隔的值
- column 左对齐的列
- html HTML 的 `<table>` 代码
- insert TABLE 表 SQL 插入语句
- line 每行一个值
- list 由 `.separator` 字符串分隔的值
- tabs 由 Tab 分隔的值
- tcl TCL 列表元素

### 参考文章
[Export SQLite Database To a CSV File](http://www.sqlitetutorial.net/sqlite-tutorial/sqlite-export-csv/)  
[使用sqlite3 命令行 导出表数据](https://www.jianshu.com/p/2980342c7be6)
