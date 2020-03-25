---
title: 趟坑之路-[SpringBoot整合Redis做缓存]
date: 2020-03-14 14:00
categories: FrameWork
tags: [SpringBoot,Redis]
urlname: Springboot-Integrate-Redis
---



redis是一个高性能的非关系型内存数据库，通常被用作数据缓存，此文记录了SpringBoot2.x整合Redis做缓存服务器的常规步骤及排错。

----

<!--more-->

本文使用到的环境：
{% raw %}
<table>
    <tr>
    	<td>OS</td>
        <td>win10</td>
    </tr>
    <tr>
    	<td>JDK</td>
        <td>1.8</td>
    </tr>
    <tr>
    	<td>SpringBoot版本</td>
        <td>2.2.5.RELEASE</td>
    </tr>
    <tr>
    	<td>redis版本</td>
        <td>2.x以上</td>
    </tr>
</table>
{% endraw %}
下面开始整合redis。

### 1. 整合Redis步骤

#### 1.1 引入依赖

在`pom.xml`文件中引入redis依赖

```xml
<!-- redis的支持-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

> 1.5版本默认采用的客户端是jedis, 2.x版本默认是lettuce
>
> - jedis：多线程使用同一连接时是线程不安全的，需使用到连接池为每个jedis分配一个连接。
> - lettuce：当多线程使用同一连接时，是线程安全的，底层基于Netty的 。

#### 1.2 配置文件中配置redis

在项目的`application.properties`配置文件中配置redis，配置如下

```properties
########redis配置##############
# Redis数据库索引(默认为0)
spring.redis.database=0
#服务器地址
spring.redis.host=127.0.0.1
#连接端口号
spring.redis.port=6379
#连接密码，默认空
spring.redis.password=
#连接池最大连接数(负值表示无限制)
spring.redis.jedis.pool.max-active=10
#连接池最大阻塞等待时间(负值表示无限制)
spring.redis.jedis.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
#连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
#连接超时时间(毫秒)
spring.redis.timeout=5000
# 若开启redis方式的session存储，type值应为redis
spring.session.store-type=redis
spring.session.timeout=10
server.servlet.session.timeout=10
# 显示hibernate的sql语句,方便查看是否成功缓存
spring.jpa.show-sql=true
```

#### 1.3 创建一个Redis缓存配置类

```java
@Configuration
@EnableCaching
public class RedisConfig {
    /*
        SpringBoot2.x的ReidsCacheManager自定义配置方法
        缓存之后,key会多一个前缀,默认是将CacheName作为key的前缀
     */
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        //解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        // 配置序列化（可解决Redis中非中文的问题，但会导致反序列化异常，原因是PageImpl没有空参构造器导致的，如果只是做缓存
        // 可以选择不自定义value的序列化规则，因为jackson2的序列化器反序列化时需要序列化类有空参构造，如果没有就会报错
        // 且不自定义value的序列化规则，不影响其反序列化出来的内容，只是redis中显示非中文，或者使用JDK默认的序列化器）
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                // 缓存有效期60s
                .entryTtl(Duration.ofSeconds(60))
//                // 设置key序列化
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
//                 // 设置value序列化
.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                //不缓存null值
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(config)
                .build();
        System.out.println("ReidsCacheManager自定义配置方法被调用了...");
        return cacheManager;
    }
}
```

> 如果开启设置value序列化，会和我下面自定义的Page类冲突，故如果使用到我自定义的Page类，需要将value序列化关闭。

#### 1.4 在适当的地方加入缓存

- User类

```java
@Entity // 表示这是个实体类
@Table(name = "user_") // 表示该类对应的表名是user
@SuppressWarnings("all") // 压制所有警告
@JsonIgnoreProperties({"handler","hibernateLazyInitianlizer"})
public class User implements Serializable{ //注意需实现序列化接口
    private static final long serialVersionUID = -6373343588870458520L;
    @Id // 表明主键
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 表明自增长方式
    @Column(name = "id") // 表名对应的数据库字段名
    private int id;
    @Column(name = "name")
    private String name;

    public User() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

- UserDao

```java
public interface UserDao extends JpaRepository<User,Integer> {
}
```

- UserService

```java
public interface UserService {
    public Page4Navigator<User> getList(Pageable pageable);
    public void save(User user);
    public void delete(User user);
    public User get(int id);
}
```

- UserServiceImpl

```java
@Service
// 设置缓存名称
@CacheConfig(cacheNames = "user")
public class UserServiceImpl implements UserService {
    @Autowired UserDao userDao;


    @Override
    // 开启缓存
    @Cacheable(key = "'user'+#p0.offset+'-'+#p0.pageSize ")
    public Page4Navigator<User> getList(Pageable pageable) {
        Page<User> pageFromJPA = userDao.findAll(pageable);
        Page4Navigator<User> page = new Page4Navigator<>(pageFromJPA, 5);
        return page;
    }
    
	@Override
    // 该方法调用后立即清除缓存
    @CacheEvict(allEntries = true)
    public void save(User user) {
        userDao.save(user);
    }

    @Override
    @CacheEvict(allEntries = true)
    public void delete(User user) {
        userDao.delete(user);
    }

    @Override
    // 将查询结果缓存到redis中， user+#p0指定传入的第一个参数+user做为redis的key
    @Cacheable(key = "'user '+ #p0")
    public User get(int id) {
        return userDao.findById(id).get();
    }
}
```

##### Redis注解方式

###### @CacheConfig

定义该类下所有缓存到redis中的父名称。

###### @Cacheable

根据方法的请求参数对其结果进行缓存，根据Key进行判断，如已在缓存中，则不执行方法，直接从redis中查询返回结果。

- `cacherNames`：存储该方法调用结果的缓存名称。
- `key`：缓存的key，可以为空；如果指定要按照SpEL表达式编写，不指定默认按照方法的所有参数进行组合。
- `cacheManager`：指定使用的cacheManager
- `condition`：缓存的条件，可以为空，使用SpEL编写，返回布尔值，只有为true才进行缓存。举例：` @Cacheable(cacheNames=“userTemp1”, key = “‘UserId_’ + #id”,condition = “#id<=110”) `
  - 表示判断id值，如果id值小于等于110，在方法执行成功后，缓存到redis中。

###### @CachePut

根据方法的请求参数对其结果进行缓存，更新Redis中对应键的值，参数同上。

###### @CacheEvict

根据一定条件对缓存进行清空，特有参数如下：

- `allEntries`：是否清空所有缓存内容，默认false；指定为true则方法调用后将立即清空所有缓存。
- ` beforeInvocation `：是否在方法执行前就清空，默认false；指定为true则方法还未执行时就会清空缓存。缺省清空下如果方法抛出异常，则不会清空缓存。

其他详细Redis注解内容可见[文章](https://segmentfault.com/a/1190000017057950)

- 分页工具类Page

```java
public class Page4Navigator<T> implements Serializable{
    private static final long serialVersionUID = -4042369404722051112L;
    Page<T> page4jpa;
    int navigatePages;
    int totalPages;
    int number;
    long totalElements;
    int size;
    int numberOfElements;
    List<T> content;
    boolean isHasContent;
    boolean first;
    boolean last;
    boolean isHasNext;
    boolean isHasPrevious;
    int[] navigatepageNums;

    public Page4Navigator() {
    }

    public Page4Navigator(Page<T> page4jpa, int navigatePages) {
        this.page4jpa = page4jpa;
        this.navigatePages = navigatePages;
        totalPages = page4jpa.getTotalPages();
        number = page4jpa.getNumber();
        totalElements = page4jpa.getTotalElements();
        size = page4jpa.getSize();
        numberOfElements = page4jpa.getNumberOfElements();
        content = page4jpa.getContent();
        isHasContent = page4jpa.hasContent();
        first = page4jpa.isFirst();
        last = page4jpa.isLast();
        isHasNext = page4jpa.hasNext();
        isHasPrevious = page4jpa.hasPrevious();
        calcNavigatepageNums();
    }

    private void calcNavigatepageNums() {
        int navigatepageNums[];
        int totalPages = getTotalPages();
        int num = getNumber();
        //当总页数小于或等于导航页码数时
        if (totalPages <= navigatePages) {
            navigatepageNums = new int[totalPages];
            for (int i = 0; i < totalPages; i++) {
                navigatepageNums[i] = i + 1;
            }
        } else { //当总页数大于导航页码数时
            navigatepageNums = new int[navigatePages];
            int startNum = num - navigatePages / 2;
            int endNum = num + navigatePages / 2;

            if (startNum < 1) {
                startNum = 1;
                //(最前navigatePages页
                for (int i = 0; i < navigatePages; i++) {
                    navigatepageNums[i] = startNum++;
                }
            } else if (endNum > totalPages) {
                endNum = totalPages;
                //最后navigatePages页
                for (int i = navigatePages - 1; i >= 0; i--) {
                    navigatepageNums[i] = endNum--;
                }
            } else {
                //所有中间页
                for (int i = 0; i < navigatePages; i++) {
                    navigatepageNums[i] = startNum++;
                }
            }
        }
        this.navigatepageNums = navigatepageNums;
    }

    //省略get/set方法
    ... ... 
}

```

- 控制器

```java
@RestController
public class UserController_Json {
    @Autowired UserService userService;

    /**
     * 获取全部user对象
     * @return
     * @throws Exception
     */
    @GetMapping("/user")
    public List<User> listUser(@RequestParam(value = "start",defaultValue = "0") int start,
                               @RequestParam(value = "size",defaultValue = "5") int size) throws Exception{
        start = start < 0?0:start;
        new Sort.Order(Sort.Direction.DESC,"id");
        Sort orders = Sort.by("id");
        // 根据start、size和sort创建分页对象
        PageRequest pageable = PageRequest.of(start,size,orders);
        // DAO根据这个分页对象获取结果page,这个page对象里，不仅包含了分页信息，还包含了数据信息,类转换异常？
        Page4Navigator<User> list = userService.getList(pageable);
        return list.getContent();
    }

    /**
     * 获取单个user对象
     * @param id
     * @return
     * @throws Exception
     */
    @GetMapping("/user/{id}")
    @ResponseBody
    public User getUser(@PathVariable("id") int id,
                        HttpServletResponse response) throws Exception{
        User user = userService.get(id);
        System.out.println(user);
        return user;
    }

    /**
     * 添加用户
     * @param user
     * @throws Exception
     */
    @PutMapping("/user")
    public void addUser(@RequestBody User user) throws Exception{
        System.out.println("SpringBoot接受到浏览器以JSON格式提交的数据："+user);
        userService.save(user);
    }
}
```

#### 1.5 启动redis

找到`redis-server.exe`，双击运行。

![redis-server](http://yanxuan.nosdn.127.net/63d63e531aa2b31f79be985307c226e4.png)

#### 1.6 启动项目

启动项目测试接口，http://localhost/user，http://localhost/user/4，查看hql语句输出，可以看到除了第一次访问会查询数据库，后续都没有查询数据库，而是从redis中获取数据。

![test](http://yanxuan.nosdn.127.net/af29a07caeb364eec6d679bec77f91e6.png)

### 2. 踩坑记录

#### 2.1 二次查询报错，PageImpl没有构造方法，导致无法反序列化

其原因在于我自定义的Page类中有使用到`org.springframework.data.domain.Page`类，该类有一个实现类`PageImpl`，此类是没有实现无参构造器的，故从缓存中读取数据反序列化时会报错：` org.springframework.data.redis.serializer.SerializationException: Could not read JSON: Can not construct instance of org.springframework.data.domain.PageImpl ....`

解决这个问题有两个方法：

1. 不设置`RedisConfig`配置类中自定义的value序列化规则，因为使用`jackson2JsonRedisSerializer`设置value的序列化值就会报错，不设置就不会；不设置的弊端就是，Redis数据库中存储的数据是HEX格式的，也就是16进制数据，不方便查阅。
2. 可以使用另一个Page类，替换我原本的page类，暂时没有深入研究这两个类的区别，有兴趣的同学可以研究下。

```java
public class MyPage<T> implements Iterable<T>, Serializable {
    private static final long serialVersionUID = -3720998571176536865L;
    private List<T> content = new ArrayList<>();
    private long totalElements;
    private int pageNumber;
    private int pageSize;
    private boolean first;
    private boolean last;
    private boolean empty;
    private int totalPages;
    private int numberOfElements;

    public MyPage() {
    }

    
    public MyPage(Page<T> page) {
        this.content = page.getContent();
        this.totalElements = page.getTotalElements();
        this.pageNumber = page.getPageable().getPageNumber();
        this.pageSize = page.getPageable().getPageSize();
        this.numberOfElements = page.getNumberOfElements();
    }
    
    // 省略get/set方法
    ... ...
    
   //迭代器
    @Override
    public Iterator<T> iterator() {
        return getContent().iterator();
    }
}    
```



#### 2.2 java.util.LinkedHashMap cannot be cast to XXX

发生在读取redis缓存的时候报错，` java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to XXX `，但是我个人代码中并没有定义和使用到`LinkedHashMap`，查资料后发现原因是` Jackson2JsonRedisSerializer`，该序列化器中默认使用的`ObjectMapper`是直接`new ObjectMapper()`创建的，这样`ObjectMapper`会将redis中的字符串反序列化为`LinkedHashMap`类型，导致后续类型转换错误。

解决方法，配置`objectMapper`的默认类型：

```java
Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer =
    new Jackson2JsonRedisSerializer<>(Object.class);
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
objectMapper.configure(MapperFeature.USE_ANNOTATIONS, false);
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
// 配置如下，解决LinkedHashMap转换异常
objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
```

#### 2.3 redis数据非中文

只需在`redisCacheManager`方法中进行redis的缓存配置，设置value序列化。

```java
RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
    // 缓存有效期60s
    .entryTtl(Duration.ofSeconds(60))
    // 设置key序列化
    .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
    // 设置value序列化
    .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
    //不缓存null值
    .disableCachingNullValues();

```

#### 2.4 相同类转换异常

关闭IDEA的热部署即可，原因和JVM的双亲委派有关，感兴趣的自查。
