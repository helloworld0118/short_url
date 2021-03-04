### short-url采用JAVA语言框架springboot 进行短域名服务的web项目。 
##### 缓存用redis 存储用mysql 分表查询框架用sharding-jdbc
```
 
 1)、长地址转换成短地址
	POST：http://${host}/generateShortUrl  
	参数：originUrl  
  实现：  
	1、对originUrl进行正则和长度[最大长度1380参考sina]匹配，提示非合法URL。
  2、每次访问生成snowflake ID 并进行62进制转换减少存储长度，并保证ID不会重复。  
	3、将长地址和短地址关系存储到redis和mysql 中以供查询。	  
2)、短地址转换成长地址  
	GET：http://${host}/${shortCode}  
	参数：shortCode  
	实现：  
	1、对shortCode 进行正则匹配，非合法提示异常。  
	2、从redis 里查shortCode对应长地址，有则直接跳转到长地址，没有则从mysql中再查，有则跳转，没有就提示异常。  
3)、存储设计  
	1、redis： 长地址和短地址各作为key，存储对应的短地址和长地址，默认缓存1小时  
	2、mysql：字段 id[PRIMARY KEY]，short_code，origin_url[最大长度1380]，origin_url_hash[方便加索引]，create_timestamp[用做数据分表]  
4)、防OWASP攻击  
	1、URL进行正则匹配，防注入  
	2、将同一IP同一地址的超N次访问进行拦截，防恶意访问  
4)、高可用，容灾，高并发  
	1、nginx做访问负载均衡，保证高可用  
	2、mysql主从复制，对数据进行容灾  
	3、长链接转换短链接在返回snowflake ID后，后台进行多线程存储同时将热点数据存放在redis来保证高并发  
6)、全球不同地理区域访问体验  
	1、使用springboot i18n给非中文语言为首选的网民提供英文选项  
7)、监控报警  
	1、在linux 服务器添加定时任务执行monitor.py 脚本  


```
