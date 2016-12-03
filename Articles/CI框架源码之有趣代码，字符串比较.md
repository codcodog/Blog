---
title: CI框架源码之有趣代码，字符串比较
date: 2016-11-07 23:33:04
tags: 
- CI
- 字符串比较
categories: Phper
---
出于兴趣，最近学习CI框架的源码，让我发现了一些有趣的东西。  

在核心库中也就是，`/core/compat/hash.php`中看到这样的代码：  
```
if ( ! function_exists('hash_equals'))
{
    /**
     * hash_equals()
     *
     * @link    http://php.net/hash_equals
     * @param   string  $known_string
     * @param   string  $user_string
     * @return  bool
     */
     function hash_equals($known_string, $user_string)
     {
         if ( ! is_string($known_string))
         {
             trigger_error('hash_equals(): Expected known_string to be a string, '.strtolower(gettype($known_string)).' given', E_USER_WARNING);
             return FALSE;
         }
         elseif ( ! is_string($user_string))
         {
             trigger_error('hash_equals(): Expected user_string to be a string, '.strtolower(gettype($user_string)).' given', E_USER_WARNING);
             return FALSE;
         }
         elseif (($length = strlen($known_string)) !== strlen($user_string))
         {
             return FALSE;
         }

         $diff = 0;
         for ($i = 0; $i < $length; $i++)
         {
             $diff |= ord($known_string[$i]) ^ ord($user_string[$i]);
         }

             return ($diff === 0);
         }
}
```
注意看函数后面那几行，也就是从`$diff = 0`开始。  
看到这行代码，*`$diff |= ord($known_string[$i]) ^ ord($user_string[$i]);`*，有点疑惑，然后去Google了下，发现原来是这样子......  

首先在这里必须说一下`按位操作运算`.  
按位操作，四个操作符：  
`~`: 按位非，对该整数的二进制形式逐位取反.例如：  
`~4`, 4的二进制形式位：00000000 00000000 00000000 00000100，逐位取反得：11111111 111111111 11111111 11111011，即是-5.

`|`: 按位或，对两个整数的二进制形式逐位进行逻辑或运算，原理为：1|0=1,0|0=0,1|1=1,0|1=1.  

`&`: 按位与，对两个整数的二进制形式逐位进行逻辑与运算，原理为：1&0=0,0&0=0,1&1=1,0&1=0.  

`^`: 按位异或，对两个整数的二进制形式逐位进行逻辑异或运算，原理为：1^1=0,1^0=1,0^1=1,0^0=0.  

了解到`按位操作运算`之后，再看回这行代码`$diff |= ord($known_string[$i]) ^ ord($user_string[$i]);`就简单明了了.  
循环两个字符串，逐个字符比较（通过ASII码进行`按位异或运算`），然后通过和`$diff = 0`进行`按位或运算`，从而返回两个字符串是否相等.  
