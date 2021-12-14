2006 mysql server has gone away
===============================

### 场景

php `yii` 框架，一个 `console` 任务，执行的时候，报 `2006 mysql server has gone away` 错误.


### 方案

导致报错的原因是，这个 `console` 任务处理的数据太多，大量的错误测试数据发送到 `mysql`，超出了包大小的限制.

```php
...
$task->status = TaskModel::STATUS_FAIL;
$task->message = json_encode($object->errors);
$task->save();
return self::DELETE;
...
```

上面的 `$task->message = json_encode($object->errors);` 文本内容过大，导致 `save()` 之后，超出了 `mysql` 限制的大小，  
从而导致 `2006 mysql server has gone away`.

查看 `mysql` `max_allowed_packet` 的大小：
```
azoya_product_system> show variables like '%max_allowed_packet%';
+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| max_allowed_packet       | 16777216   |
| slave_max_allowed_packet | 1073741824 |
+--------------------------+------------+
```

修改 `max_allowed_packet` 的大小
```
azoya_product_system> set global max_allowed_packet = 1073741824;
```

修改之后，退出 `mysql`，重新登录即可.


或者，限制 `$object->erros` 的大小
```
if (count($object->erros) > 50) {
    $object->erros = array_slice($object->erros, 0, 50);
}
```
