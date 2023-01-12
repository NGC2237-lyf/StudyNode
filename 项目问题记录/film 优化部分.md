film 优化部分

性能优化：

1、防爬虫：限制分页查询次数

2、用户数据脱敏

3、微信公众号登录并与前端对接

4、熟练使用内网穿透



## 1、防爬虫：限制分页查询次数

善意爬虫：

善意爬虫都会遵守**robots协议**，我们可以在**网站的根目录下**存放一个**ASCII编码**的文本文件，告诉搜索引擎哪些页面不能爬取，搜索引擎的蜘蛛便会遵照协议，不爬取指定页面的内容。

恶意爬虫：

### 1-1、**限制User-Agent字段**

User-Agent字段能识别用户所使用的操作系统、版本、CPU、浏览器等信息

通过分辨请求源位置，识别是否为爬虫。

-   但是效果不好：恶意爬虫请求的User-Agent字段中带上baidu字符，伪装成百度爬虫，则无法拦截

-   如果该字段中表明为浏览器等使用的爬虫，使用DNS正向和反向查找的方法可以确定发起请求的IP地址是否与其声明的一致，则可以将其进行判别。

目前也有许多开源的项目使用上述方法检测网络爬虫，例如CrawlerDetect 就是github上的一个开源项目[5]，通过User-Agent和 http_from 字段检测爬虫，目前能够检测到 1,000 种网络爬虫。

在安全运营场景中，对于其余无法识别的爬虫，可以基于HTTP请求的速率、访问量、请求方法、请求文件大小等行为特征，设计算法进行识别。

- 请求总数:请求的数量。
- 会话持续时间:第一个请求和最后一个请求之间经过的总时间。
- 平均时间:两个连续请求之间的平均时间。
- 标准偏差时间:两个连续请求之间时间的标准偏差。
- 重复请求:使用与以前相同的HTTP方法请求已经访问过的页面。
- HTTP请求:四个特性，每个特性包含与以下HTTP响应代码之一相关联的请求的百分比:成功(2xx)、重定向(3xx)、客户机错误(4xx)和服务器错误(5xx)。
- 特定类型请求：特定类型的请求占所有请求数的百分比，这一特征在不同的应用程序中表现不同。

除了上述特征外，这一工作从会话中提取到了一部分语义特征：包括主题总数、独特主题、页面相似度、页面的语义差异等，并使用了四种不同的模型，包括使用RBF的SVM，梯度增强模型，多层感知器和极端梯度增强来测试检测结果。从不同特征集上的实验结果可以看出，**RBF在语义特征上取得了最好的性能**，GB在简单典型特征上取得了最好的性能，GB在典型特征和语义特征的结合上也取得了最好的性能。

- 注意：PathMarker的反爬虫技术，可以通过检测网页或请求之间的关系来检测**分布式爬虫**。在这一方法中，通过向URL添加标记来跟踪访问该URL之前的页面，并识别访问该URL的用户。根据URL访问路径和访问时间的不同模式，使用支持向量机模型来区分恶意网络爬虫和普通用户。实验结果表明，该系统能够成功识别96.74%的爬虫长会话和96.43%的普通用户长会话。PathMarker的体系结构如图1所示，最后使用自动化的公共图灵测试(CAPTCHA)实时地识别爬虫和普通用户。

UA字串的标准格式：浏览器标识 (操作系统标识; 加密等级标识; 浏览器语言) 渲染引擎标识版本信息。但各个浏览器有所不同。

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3100.0 Safari/537.36
```



#### 1-1-1、想法：

> 拦截机制中做判断：
>
> request.getHeaderNames(); // 获取请求头集合
>
> request.getHeader(nextElement);//通过请求头得到请求内容
>
> 进行比对

> 拦截机制有三种：
>
> ###### 1. [过滤器（Filter）](https://www.jianshu.com/p/3960fd97a294)能拿到http请求，但是拿不到处理请求方法的信息。
>
> ###### 2. [拦截器（Interceptor）](https://www.jianshu.com/p/43e937436386)既能拿到http请求信息，也能拿到处理请求方法的信息，但是拿不到方法的参数信息。
>
> ###### 3. [切片（Aspect）](https://www.jianshu.com/p/38930293748d)能拿到方法的参数信息，但是拿不到http请求信息。

#### 1-1-2、拦截器机制：

![image-20221013165517789](https://gitee.com/ncg2237/picture/raw/master/202210131655324.png)

```JAVA
/**
 * 限制User-Agent字段
 */
@Slf4j
@Component
public class dbFilterConfig implements HandlerInterceptor{
    //controller 调用之前被调用
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object handler) throws Exception {
        return true;
    }

    //controller 调用之后被调用，如果有异常则不调用 （可限制请求频率）
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
    }

    //controller 调用之后被调用，有没有异常都会被调用,Exception 参数里放着异常信息
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}	
```

但是只写这个处理拦截器还不行，还需要进一步的配置，请看下面一段代码：

```JAVA
/**
 * 引入第三方过滤器 将其放入spring容器
 */
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter{

//    注入 拦截器
    @Autowired
    private dbFilterConfig db;
    //添加拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //往拦截器注册器里添加拦截器
        registry.addInterceptor(db);
    }

```



### 1-2、**限制IP**

恶意爬虫的请求频率往往比正常流量高，找出这些IP并限制其访问，可以有效降低恶意爬虫造成的危害。

缺点：

容易误伤正常用户，攻击者可以通过搭建IP池的方法，来解决这个问题。



获取用户真实IP地址：不使用request.getRemoteAddr();的原因是有可能用户使用了代理软件方式避免真实IP地址；
可是，如果通过了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP值，究竟哪个才是真正的用户端的真实IP呢？

答案是取X-Forwarded-For中第一个非unknown的有效IP字符串。

工具类：

```JAVA
/**
 * 获取用户访问 ip
 */
public class IpUtil{
//    X-Forwarded-For
    public static String getIpAddress(HttpServletRequest request){
        String header = request.getHeader("X-Forwarded-For");
        String trim = header.trim();
        if (trim == null || trim.length() == 0 || "unknown".equalsIgnoreCase(trim)) {
            trim = request.getHeader("Proxy-Client-IP");
        }
        if (trim == null || trim.length() == 0 || "unknown".equalsIgnoreCase(trim)) {
            trim = request.getHeader("WL-Proxy-Client-IP");
        }
        if (trim == null || trim.length() == 0 || "unknown".equalsIgnoreCase(trim)) {
            trim = request.getHeader("HTTP_CLIENT_IP");
        }
        if (trim == null || trim.length() == 0 || "unknown".equalsIgnoreCase(trim)) {
            trim = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (trim == null || trim.length() == 0 || "unknown".equalsIgnoreCase(trim)) {
            trim = request.getRemoteAddr();
        }
        if (trim != null && trim.contains(",")){
            String[] split = trim.split(",");
            trim = split[0];
        }
        return trim;
    }
}
```

过滤器：

```JAVA
// 设置计数时间
        //服务器使用session把用户的信息临时保存在了服务器上，用户离开网站后session会被销毁。
        //可是session有一个缺陷：如果web服务器做了负载均衡，那么下一个操作请求到了另一台服务器的时候session会丢失。
        if ( userIpInfo == null || userIpInfo.getIpAddress() == null){
            userIpInfo.setTimes(1);
            userIpInfo.setLoginTime(new Date());
            httpServletRequest.getSession().setAttribute("userIpInfo",userIpInfo);
        }else{
            //判断时间是否在一定限制内
            Date nowDate = new Date();
            Date loginTime = userIpInfo.getLoginTime();
            long dicTime = nowDate.getTime() - loginTime.getTime();  // 得到毫秒数
            long time = dicTime / (1000 * 60 * 1); // 一分钟
            if (time > 5){
                userIpInfo.setLoginTime(nowDate);
                userIpInfo.setTimes(1);
                httpServletRequest.getSession().setAttribute("userIpInfo",userIpInfo);
            }
            if (time <= 1 ){
                userIpInfo.setTimes(userIpInfo.getTimes()+1);
                // 一分钟访问3次
                if (userIpInfo.getTimes() >=3) {
                    return false;
                }
                return true;
            }
        }
        return true;
```



### 1-**3、添加验证码**

在登录页等页面，添加验证码

缺点：如今的爬虫技术，已能解决验证码的问题

![image-20221014170731838](C:\Users\26463\AppData\Roaming\Typora\typora-user-images\image-20221014170731838.png)

示例展示：

![image-20221014170810174](https://gitee.com/ncg2237/picture/raw/master/202210141708240.png)



### 1-**4.使用爬虫管理产品**

**蔚可云 **提供了**BotGuard爬虫管理产品**，通过交互验证、大数据分析、合法性验证等策略，帮助企业实时检测、管理和阻断恶意爬虫。

------

### **==最终结果==**：

```java
/**
 * 根据ip地址限制分页查询次数
 * 目前限制：2秒5次
 * 需改动：urls
 */
@Slf4j
@Component
public class DbFilterConfig implements HandlerInterceptor {
    //controller 调用之前被调用
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object handler) throws Exception {
        String[] urls = {
                // 写入防止被爬的路径，以用户查询为例
                "/api/usrdb/userdb"
        };
        // 取出url
        String url = httpServletRequest.getRequestURL().toString();
        // 遍历
        boolean res = Arrays.asList(urls).contains(url);
        if (!res){
            return true;
        }
        String ipAddress = IpUtil.getIpAddress(httpServletRequest);
        RedisTemplate redisTemplate = new RedisTemplate();
        // 判断是否过期
        if (redisTemplate.getExpire(ipAddress)==null) {
            redisTemplate.expire(ipAddress, 2, TimeUnit.SECONDS);
            // 初始化访问次数
            redisTemplate.opsForValue().set("times",1);
            return  true;
        }
        // 没有过期，增加访问次数
        redisTemplate.opsForValue().set("times",(int) redisTemplate.opsForValue().get("times")+1);
        //判断其访问次数是否高达5次  2秒访问5次
        if ((int) redisTemplate.opsForValue().get("times")>=5){
            return false;
        }
        return true;
    }

    //controller 调用之后被调用，如果有异常则不调用 （可限制请求频率）
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    //controller 调用之后被调用，有没有异常都会被调用,Exception 参数里放着异常信息
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

### ![image-20221014201952007](https://gitee.com/ncg2237/picture/raw/master/202210142019154.png)

## 2、用户脱敏

数据库安全技术主要包括：数据库漏扫、数据库加密、数据库防火墙、数据脱敏、数据库安全审计系统。

### 2-1、类型

#### 2-1-1. 静态脱敏

- 将数据抽取出生产环境脱敏后分发至测试、开发等场景。
- 脱敏后数据与生产环境相隔离，满足业务需求的同时保障生产数据库的安全。

#### 2-1-2. 动态脱敏

- 在查询语句执行过程中，根据生效条件是否满足，实现实时的脱敏处理。
- 一般用在生产环境，访问敏感数据时实时进行脱敏，因为有时在不同情况下对于同一敏感数据的读取，需要做不同级别的脱敏处理。
- 需要注意的是，在抹去数据中的敏感内容同时，也需要保持原有的数据特征、业务规则和数据关联性，保证我们在开发、测试以及数据分析类业务不会受到脱敏的影响，使脱敏前后的数据一致性和有效性。

### 2-2、方案

#### 2-2-1. 替换

- 随机值替换，字母变为随机字母，数字变为随机数字，文字随机替换文字的方式来改变敏感数据
- 优点：在一定程度上保留原有数据的格式，往往这种方法用户不易察觉的。如统一将用户名中的“张”替换为A，这种方法更像“障眼法”，对内部人员可以完全保持信息完整性，但**易破解**。

#### 2-2-2. 无效化

- 通过对字段数据值进行 截断、加密、隐藏 等方式让敏感数据脱敏。
- 方法：采用特殊字符（*等）代替真值
- 缺点：用户无法得知原数据的格式，如果想要获取完整信息，要让用户授权查询。如遮盖手机号的中间四位、身份证号中间部分等。

#### 2-2-3. 置乱

- 对敏感数据列的值进行重新随机分布，混淆原有值和其他字段的联系。
- 这种方法不影响原有数据的统计特性，如最大/ 最小/ 方差等均与原数据无异。

#### 2-2-4. 均值

- 平均值方案经常用在统计场景，针对数值型数据，我们先计算它们的均值，然后使脱敏后的值在均值附近随机分布，从而保持数据的总和不变。通常用于产品成本表、工资表等场合。

#### 2-2-5. 对称加密

- 通过给原始数据增加salt值来实现数据的可逆，多用于密码存储等场景。对称加密是一种特殊的可逆脱敏方法，通过**加密密钥**和**算法**对敏感数据进行加密，密文格式与原始数据在逻辑规则上一致，通过密钥解密可以恢复原始数据。
- 要注意的就是密钥的安全性。

#### 2-2-6. 偏移

- 这种方式通过随机移位改变数字数据，偏移取整在保持了数据的安全性的同时保证了范围的大致真实性，比之前几种方案更接近真实数据，在大数据分析场景中意义比较大。

![image-20221014203847071](https://gitee.com/ncg2237/picture/raw/master/202210142038155.png)