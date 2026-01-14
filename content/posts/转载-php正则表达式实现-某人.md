---
title: '[转载] php正则表达式实现@某人'
categories: [PHP]
slug: reserved-php-regular-expression-implementation-某人
date: 2016-10-25 10:07:00
updated: 2016-10-25 10:07:00
toc: false
---

PHP正则表达式实现@某人 
if(preg_match_all('#@\w+#u', '@张全蛋 含泪质检@三星Note7 被炸飞,听说@炸机 跟@啤酒 更配哦!', $matches)) {
  var_export($matches);
}
//输出
array (
0 => 
array (
0 => '@张全蛋',
1 => '@三星Note7',
2 => '@炸机',
3 => '@啤酒',
),
)
正则表达式 #@\w+#u 中:
`#`是分隔符.
`u`是修饰符,表示Unicode.
`\w`是元字符,在ASCII下等价于[A-Za-z0-9_],在Unicode下表示字符(包括汉字)和数字和下划线.
`+`是量词,表示1个或多个,等价于{1,}
