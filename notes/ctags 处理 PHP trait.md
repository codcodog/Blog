ctags 处理 PHP trait
====================

#### 场景
使用 `ctags` 生成 `PHP` 的 `tags`，发现对 `trait` 关键字是没有进行处理的.
```bash
$ ctags --languages=php -R .
```

#### 方案
使用 `--regex-php` 参数，生成 `trait` 关键字的 `tag`
```bash
$ ctags --languages=php --regex-php='/^[ \t]*trait[ \t]+([a-z0_9_]+)/\1/t,traits/i' -R .
```

`--regex-php` 参数说明
```
--regex-php=/regex/tag-name/tag-kind/flags
```

若使用 `ctags` `Universal` 版本，只要在 `--php-kinds` 里指定 `t` 即可.

#### 参考
[Finding PHP Traits With Exuberant Ctags](https://ericjmritz.wordpress.com/2013/05/22/finding-php-traits-with-exuberant-ctags/)
