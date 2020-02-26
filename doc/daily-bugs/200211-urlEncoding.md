http get请求，若请求参数中有特殊字符，需要使用urlEncoding的方式对请求进行编码。

- 编码整个url然后使用这个url—>encodeURI()
- 编码url中的参数—>encodeURIComponent()

在Javascript中，切记对参数encoding需要使用encodeURIComponent()，否则则不生效。

踩坑总结：

1. web应用前端请求后台接口时，需要在js中进行参数的encoding，后端请求服务接口时，需要在java代码中进行encoding，缺一不可。
2. 另外，使用http post请求会是解决这个问题比较好的方式。