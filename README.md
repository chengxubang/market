ssm实战购物商城系统
=

程序有问题联系[程序帮](http://suo.nz/530ijn)：QQ1022287044


项目介绍
----
>本系统使用Spring+SpringMVC+MyBatis架构，数据库使用MySQL,开发完成了从商家发布商品，到用户查看商品并下单购买这样的一个闭合的流程。


项目适用人群
----
正在做毕设的学生，或者需要项目实战练习的Java学习者

开发环境
-----
1. jdk 8
2. intellij idea
3. tomcat 8.5.40
4. mysql 5.7

所用技术
-----
1. Spring+SpringMVC+MyBatis
2. layui
3. jsp

 项目架构
----
![](/src/image/项目结构.png)
 
项目截图
----

- 登录

![](/src/image/登录.png)

- 首页
![](/src/image/首页.png)
- 商品详情
![](/src/image/商品详情.png)
- 购物车
![](/src/image/购物车.png)
- 订单详情
![](/src/image/商品详情.png)






框架配置
----
1. applicationContext.xml
```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="
     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.0.xsd
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
     http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd" >
   <context:annotation-config />
	<context:component-scan base-package="com.yzx.service" />
	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	  <property name="driverClassName">  
	      <value>com.mysql.jdbc.Driver</value>  
	  </property>  
	  <property name="url">  
	      <value>jdbc:mysql://localhost:3306/market?characterEncoding=UTF-8</value>
	  </property>  
	  <property name="username">  
	      <value>root</value>  
	  </property>  
	  <property name="password">  
	      <value>root123</value>
	  </property>  	
	</bean>
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="typeAliasesPackage" value="com.yzx.entity" />
		<property name="dataSource" ref="dataSource"/>
		<property name="mapperLocations" value="classpath:com/yzx/mapper/xml/*.xml"/>
		<property name="plugins">
		    <array>
		      <bean class="com.github.pagehelper.PageInterceptor">
		        <property name="properties">
		          <!--使用下面的方式配置参数，一行配置一个 -->
		          <value>
		          </value>
		        </property>
		      </bean>
		    </array>
		  </property>		
	</bean>
    <!--增加tkmybatis注解依赖-->
	<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.yzx.mapper"/>
         <property name="beanName" value="normal"/>
	</bean>
</beans>
```


2. springMVC.xml
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.0.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd 
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">
    <context:annotation-config/>
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>
    <context:component-scan base-package="com.yzx.controller">
          <context:include-filter type="annotation" 
          expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <mvc:annotation-driven />
    <mvc:default-servlet-handler />
    <bean
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass"
            value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>   
```


3. mybatis依赖配置
```
// 继承Mapper，MySqlMapper  其他业务Mapper继续“MyMapper”，可直接操作增删改查方法
public interface MyMapper<T> extends Mapper<T>,MySqlMapper<T>{
}
```

业务代码
----
1.首页代码-controller层
``` 
@RequestMapping(value = {"/","index","main",""})
public ModelAndView index(ModelAndView mav,HttpServletRequest request){
    Person person=(Person)request.getSession().getAttribute("user");
    List<Product> productList= productService.getProductList();
    if(null!=person){
        Trx trx=new Trx();
        if(person.getUserType().equals(0)){	//买家
            trx.setPersonId(person.getId());
        }
        List<Product> productList2=new ArrayList<>();
        List<Trx> trxList=trxService.getTrxList();	//根据当前登录id查询是否购买
        if(null!=trxList&&trxList.size()>0){
            for(Product p:productList){
                for(Trx t:trxList){
                    if(t.getProductId().equals(p.getId())){
                        p.setIsSell(1);
                    }
                }
                if(person.getUserType()==1){	//卖家角色 查询当前商品买过多少
                    Trx t1=new Trx();
                    t1.setProductId(p.getId());
                    List<Trx> list=trxService.getTrxByParm(t1);
                    p.setBuyNum(list.size());
                }
                productList2.add(p);
            }
            mav.addObject("productList",productList2);	//过滤后的商品列表
        }
        mav.addObject("productList",productList);
    }else{
        mav.addObject("productList",productList);	//商品列表
    }
    mav.setViewName("index");
    return mav;
}
```
1.1 首页代码-service层
```
@Service
public class ProductServiceImpl implements ProductService {
	@Autowired
	ProductMapper productMapper;
	//商品列表
	public List<Product> getProductList(){
		return productMapper.selectAll();
	}
}
```

1.2 首页代码-dao层-Mapper
```
//继承MyMapper类，获取超类的增删改查方法
public interface ProductMapper extends MyMapper<Product> {
}
```

1.3 首页代码-jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<head>
    <meta charset="utf-8"/>
    <title>java</title>
    <link rel="stylesheet" href="/css/style.css"/>
</head>
<body>
<div class="n-support">请使用Chrome、Safari等webkit内核的浏览器！</div>
<div class="n-head">
    <div class="g-doc f-cb">
        <c:if test="${not empty user}">
            <div class="user">
                买家你好，<span class="name">${user.nickName}</span>！<a href="logout">[退出]</a>
            </div>

        </c:if>
        <c:if test="${ empty user}">
            请<a href="login">[登录]</a>
        </c:if>
            <ul class="nav">
                <li><a href="index">首页</a></li>
                <c:if test="${not empty user}">
                    <c:if test="${user.userType==0}">
                        <li><a href="account">账务</a></li>
                       <li><a href="shopCart">购物车</a></li>
                    </c:if>
                    <c:if test="${user.userType==1}">
                       <li><a href="publish">发布</a></li>
                    </c:if>
                </c:if>
            </ul>

    </div>
</div>
<div class="g-doc">
    <div class="m-tab m-tab-fw m-tab-simple f-cb">
        <div class="tab">
            <ul>
                <li class="z-sel"><a href="index">所有内容</a></li>
                <c:if test="${ user.userType!=1}">
                    <li><a href="isNoBuy">未购买的内容</a></li>
                </c:if>
            </ul>
        </div>
    </div>
    <div class="n-plist">
        <ul class="f-cb" id="plist">
            <c:forEach items="${productList}" var="product" varStatus="st">
                <li id="p-1">
                    <a href="details?pid=${product.id}" class="link">
                        <div class="img"><img src="${product.image}" alt=""></div>
                        <h3>${product.title}</h3>
                        <div class="price"><span class="v-unit">¥</span><span class="v-value">${product.price}</span></div>
                        <c:if test="${ not empty user}">
                            <c:if test="${ user.userType==0 and product.isSell==1}">
                                <span class="had"><b>已购买</b></span>
                            </c:if>
                            <c:if test="${ user.userType==1 and product.isSell==1}">
                                <span class="had"><b>已售出${product.buyNum}</b></span>
                            </c:if>
                        </c:if>
                    </a>
                    <c:if test="${ user.userType==1 and product.isSell==0}">
                        <span class="u-btn u-btn-normal u-btn-xs del" data-del="5">删除</span>
                    </c:if>
                </li>
            </c:forEach>
        </ul>
    </div>
</div>
<script type="text/javascript" src="/js/global.js"></script>
<script type="text/javascript" src="/js/pageIndex.js"></script>
</body>
</html>
```
2 购物车列表-controller
```
@RequestMapping(value = {"shopCart"})
public ModelAndView shopCart(ModelAndView mav, HttpServletRequest request){
    Person person=(Person) request.getSession().getAttribute("user");
    ShopCart shopCart3=new ShopCart();
    shopCart3.setUid(person.getId()+"");
    List<ShopCart> shopCarts=shopCartService.getShopCartList(shopCart3);
    mav.addObject("shopCarts",shopCarts);
    mav.setViewName("shopCart");
    return mav;
}
```
2.1 购物车列表-jsp页面
```
<%@ page language="java" contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<head>
    <meta charset="utf-8"/>
    <title>java</title>
    <link rel="stylesheet" href="/css/style.css"/>
</head>
<body>
<div class="n-head">
    <div class="g-doc f-cb">
        <c:if test="${not empty user}">
            <div class="user">
                买家你好，<span class="name">${user.nickName}</span>！<a href="logout">[退出]</a>
            </div>

        </c:if>
        <c:if test="${ empty user}">
            请<a href="login">[登录]</a>
        </c:if>
        <ul class="nav">
            <li><a href="index">首页</a></li>
            <c:if test="${not empty user}">
                <c:if test="${user.userType==0}">
                    <li><a href="account">账务</a></li>
                    <li><a href="shopCart">购物车</a></li>
                </c:if>
                <c:if test="${user.userType==1}">
                   <li><a href="publish">发布</a></li>
                </c:if>
            </c:if>
        </ul>

    </div>
</div>
<div class="g-doc">
    <div class="m-tab m-tab-fw m-tab-simple f-cb" id="settleAccount">
        <h2>已添加到购物车的内容</h2>
    </div>
    <table class="m-table m-table-row n-table g-b3">
        <colgroup><col class="img"/><col/><col class="time"/><col class="price"/></colgroup>
        <thead>
        <tr><th>内容图片</th><th>内容名称</th><th>购买时间</th><th>数量</th><th>购买价格</th></tr>
        </thead>
        <tbody>
            <c:forEach items="${shopCarts}" var="cart">
                <tr>
                    <td><a href="details?pid=${cart.pid}"><img src="${cart.image}" alt=""></a></td>
                    <td><h4><a href="details?pid=${cart.pid}">${cart.title}</a></h4></td>
                    <td><span class="v-time">${cart.time}</span></td>
                    <td><span class="v-num">${cart.num}</span></td>
                    <td><span class="v-unit">¥</span><span class="value">${cart.price}</span></td>
                </tr>
            </c:forEach>
        </tbody>
        <tfoot>
        <tr>
            <td><button class="u-btn u-btn-primary" onclick="goout();">退出</button></td>
            <td><button class="u-btn u-btn-primary" onclick="buy();">购买</button></td>
        </tr>
        </tfoot>
    </table>
</div>
<script type="text/javascript" src="/js/global.js"></script>
<script type="text/javascript" src="/js/pageShow.js"></script>
<script type="text/javascript"  src="/js/jquery-3.3.1.js" ></script>

<script type="text/javascript"  >
    function goout() {
        window.history.go(-1);
    }
    function buy(){
        if(confirm("确定购买吗？")){
            $.ajax({
                type:"post",
                url:"saveTrx",
                data:{pid:$("#pid").val()},
                dataType:"json",//返回的
                success:function(data) {
                    if(data==0) {
                        window.location.href="account";
                    } else {
                        alert('用户密码错误，请重新登录');
                    }
                },
                error:function(msg) {
                    alert(msg);
                }
            });
        }
    }
</script>
</body>
</html>
```

项目总结
----
项目做得比较简易，页面偏向简洁，不过有一个完整的商城主流程，此外还编写了freemaker版本的购物商城代码，后续上传源码。

 
