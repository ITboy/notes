# Servlet3.1 Specification
1. Servlet默认不是线程安全的，servlet容器根据配置选择不同的servlet来处理请求，但是如果是对同一个url的请求，就会交由同一个servlet来处理，servlet默认对于每一个servlet声明只产生一个servlet实例
2. 几种HTTP请求方式
	* doGet 读
	* doPost 修改
	* doPut 增加
	* doDelete 删
	* doHead 只返回doGet的header部分
	* doOptions 返回当前servlet支持的方法
	* doTrace 返回当前request的请求
	