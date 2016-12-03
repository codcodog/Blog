---
title: 把SESSION存储在数据库中
date: 2016-07-10 23:53:06
tags:
- SESSION自定义存储
categories: phper
---
之前遇到这样的一个情景：
一个项目在线上运行，由于当时没有把ＳＥＳＳＩＯＮ存放在Ｍｅｍｅｒｃａｃｈｅｄ/Ｒｅｄｉｓ上，也没有放在数据库中，而是直接生成文件存放在服务器上。　　
本来是没有什么问题的，但由于注册人数过多，导致生成了大量的ｓｅｓｓｉｏｎ文件，那个存放ｓｅｓｓｉｏｎ的目录下有几千个ｓｅｓｓｉｏｎ文件，从而导致服务器来取ｓｅｓｓｉｏｎ文件时，（在一个目录下检索几千个小文件）系统崩溃，把整个线上程序都拖垮了。　　

所以，现在在这里记录下，ＳＥＳＳＩＯＮ存放在数据库的一个方法。　
主要用到 `session_set_save_handler()` 和 `SessionHandlerInterface`。
不多说，看代码。

这是自定义类来处理ＳＥＳＳＩＯＮ
```
/** Implementing SessionHandlerInterface is mandatory as of PHP 5.4
* and will fail in previous versions.
*/
class DBHandler implements SessionHandlerInterface {
	protected $dbh;
public function open($save_path, $name) {
	try {
		$this->connect($save_path, $name);
		return true;
	} catch (PDOException $e) {
		return false;
	}
}

public function close() {
	return true;
}
public function destroy($session_id) {
	$sth = $this->dbh->prepare("DELETE FROM sessions WHERE session_id = ?");
	$sth->execute(array($session_id));
	return true;
}

public function gc($maxlifetime) {
	$sth = $this->dbh->prepare("DELETE FROM sessions WHERE last_update < ?");
	$sth->execute(array(time() - $maxlifetime));
	return true;
}

public function read($session_id) {
	$sth = $this->dbh->prepare("SELECT session_data FROM sessions WHERE session_id = ?");
	$sth->execute(array($session_id));
	$rows = $sth->fetchAll(PDO::FETCH_NUM);
	if (count($rows) == 0) {
		return '';
	} else {
		return $rows[0][0];
	}
}

public function write($session_id, $session_data) {
	$now = time();
	$sth = $this->dbh->prepare("UPDATE sessions SET session_data = ?,last_update = ? WHERE session_id = ?");
	$sth->execute(array($session_data, $now, $session_id));
	if ($sth->rowCount() == 0) {
		$sth2 = $this->dbh->prepare('INSERT INTO sessions (session_id,
session_data, last_update) VALUES (?,?,?)');
		$sth2->execute(array($session_id, $session_data, $now));
	}
}

public function createTable($save_path, $name, $connect = true) {
	if ($connect) {
		$this->connect($save_path, $name);
	}
	$sql=<<<_SQL_
CREATE TABLE sessions (
	session_id VARCHAR(64) NOT NULL,
	session_data MEDIUMTEXT NOT NULL,
	last_update TIMESTAMP NOT NULL,
	PRIMARY KEY (session_id)
)
_SQL_;
	$this->dbh->exec($sql);
}

protected function connect($save_path) {
/* Look for user and password in DSN as "query string" params */
	$parts = parse_url($save_path);
	if (isset($parts['query'])) {
		parse_str($parts['query'], $query);
		$user = isset($query['user']) ? $query['user'] : null;
		$password = isset($query['password']) ? $query['password'] : null;
		$dsn = $parts['scheme'] . ':';
		if (isset($parts['host'])) {
			$dsn .= '//' . $parts['host'];
		}
		$dsn .= $parts['path'];
		$this->dbh = new PDO($dsn, $user, $password);
	} else {
		$this->dbh = new PDO($save_path);
	}
	$this->dbh->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_EXCEPTION);
// A very simple way to create the sessions table if it doesn't exist.
	try {
		$this->dbh->query('SELECT 1 FROM sessions LIMIT 1');
	} catch (Exception $e) {
		$this->createTable($save_path, NULL, false);
	}
}
}
```
下面是通过　`session_set_save_handler`　来调用上面的类，来把ｓｅｓｓｉｏｎ存储在数据库中 
```
include __DIR__ . '/db.php';
ini_set('session.save_path', 'sqlite:/tmp/sessions.db');
session_set_save_handler(new DBHandler);
session_start();
// 现在可以使用 $_SESSION 保存以及获取数据了
if (! isset($_SESSION['visits'])) {
	$_SESSION['visits'] = 0;
}
$_SESSION['visits']++;
print 'You have visited here '.$_SESSION['visits'].' times.';
```

文章参考：
《ＰＨＰＣｏｏｋＢｏｏｋ》中 **11.5 Storing Sessions in a Database**
ＰＨＰ Ｍａｎｕａｌ