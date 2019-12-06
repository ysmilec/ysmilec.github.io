---
title: sqlilabs的环境搭建的Bug
date: 2019-5-2
tags: sqlilabs
categories: sqlilabs

---

> 我是在phpstudy下搭建的环境遇到了几个小问题，在网上搜都没有百度到，所以想记录下来。

**下载**

网址：https://github.com/Audi-1/sqli-labs

**安装**

将项目下载下来以后解压缩，拷到phpStudy\PHPTutorial\WWW目录下，然后在网页上输入http://localhost/sqli-labs-master/即可。

**Setup/reset Datsbase for labs**

我在进行这一步时出现了问题，提示我：_Fatal error: Uncaught Error: Call to undefined function mysql_connect()_ ，然后就是各种上网找教程，各种方法都试了没有用。最后将php版本从7切换到5既然神奇的好了。最后在php的官网中得知php 7里的connect()函数改名字了。。。

**lesson 1出现问题**

本来以为数据库都配置好了应该没问题了吧，可谁知既然出现了该回显错误却不会显得问题，加入`echo "你的sql语句是：$sql"`发现居然把我输入的
`?id= 1'`用`\`把我的`'`转义了。顿时我是一脸蒙蔽，lesson不该有这样的智商啊，而且看源码也没有加转义啊。在网上找了很多都没有人遇到这种问题。郁闷了一晚上，最后将php版本从5.2.17调到5.3.29。没想到居然神奇的好了。。。无法想象。然后通过问一个学长得知了原因。

**主要原因是在magic_quotes_gpc这个函数上。为GPC(Get/Post/Cookie)操作设置magic_quotes状态。当magic_quotes为on，所有的'(单引号)、“(双引号)、\(反斜杠)和NUL's被一个反斜杠自动转义。warning:本特性自5.3.0起废弃并将自PHP5.4.0起移除。查看php.ini确实如此。**

```
$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysql_error());
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}
```

对于PHP这个改变，大佬给出的解释是**内核自作主张引入其他字符往往会在具体情况下产生意想不到的漏洞，所以还是交给用户比较好。**
