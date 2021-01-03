# 一、编程规约
## （一）命名规约
* 8【强制】POJO类中布尔类型的变量，都不要加is,否则部分框架解析会引起序列化错误.  
```
反例：定义为基本数据类型boolean isSuccess;的属性，它的方法也是isSuccess(),RPC框架在反向解析的时候，“以为”对应的属性名称是success,
导致属性获取不到，进而抛出异常。
```
* 9【强制】包名统一使用小写，点分隔符之间有且仅有一个自然语意的英文单词。包名统一使用单数形式，但是类名如果有复数含义，类名可以使用复数形式。
* 15【参考】各层命名规约：  
```
A)service/DAO层方法命名规约
    1)获取单个对象的方法用get做前缀
    2)获取多个对象的方法用list做前缀
    3)获取统计值的方法用count做前缀
    4)插入的方法用save(推荐)或insert做前缀
    5)删除的方法用remove(推荐)或delete做前缀
    6)修改的方法用update做前缀
B)领域模型命名规约
    1)数据对象：xxxDO,xxx为表名
    2)数据传输对象：xxxDTO,xxx为业务领域相关的名称
    3)展示对象：xxxVO，xxx一般为网页名称
    4)POJO是DO/VO/DTO/BO的统称，禁止命名成xxxPOJO
```
## （二）常量定义
* 1【强制】不允许出现任何魔法值（即未经定义的常量）直接出现在代码中
* 2【强制】long或者Long初始赋值时，必须使用大写的L，不能使用小写的l，小写容易跟数字1混淆，造成误解。
* 3【推荐】不要使用一个常量类维护所有的常量，应该按照常量功能进行归类，分开维护。如：缓存相关的常量放在类：CacheConsts下，系统配置相关的常量放在类：ConfigConsts下。
* 4【推荐】常量的复用层次有五层：跨应用共享常量、应用内共享常量、子工程内共享常量、包内共享常量、类内共享常量。  
```
1)跨应用共享常量：放置在二方库中，通常是client.jar中的constant目录下
2)应用内共享常量：放置在一方库的modules中的constant目录下。
3)子工程内部共享常量：即当前子工程的constant目录
4)包内共享常量：即当前包下的constant目录
5)类内共享常量：直接在类的内部private static final定义
```
* 5【推荐】如果变量值仅在一个范围内变化用Enum类，如果还带有名称之外的延伸属性，必须使用Enum类，下面正例中的数字就是延伸信息，表示星期几。  
```
正例：
public Enum{
    MONDAY(1),TUESDAY(2),WEDNESDAY(3),THURSDAY(4),FRIDAY(5),SATURDAY(6),SUNDAY(7);
}
```
## （四）OOP规约
* 8【强制】关于基本数据类型与包装数据类型的使用标准如下：  
```
1)所有POJO类属性必须使用包装数据类型
2)RPC方法的返回值和参数都必须使用包装类型数据
3)所有的局部变量【推荐】使用基本数据类型
说明：POJO类属性没有初值是提醒使用者在需要使用时，必须显式地进行赋值，任何NPE问题，或者入库检查，都由使用者来保证。
正例：数据库的查询结果可能为null,因为自动拆箱，用基本数据类型接收有NPE风险。
反例：比如显示成交总额跌涨情况，即正负x%，x为基本数据类型，调用的RPC服务，调用不成功时，返回的是默认值，页面显示0%，
这是不合理的，应该显示为中划线-。所以包装数据类型的null值，能够表示额外的信息，比如：远程调用失败，异常退出等。
```
## （五）集合处理
* 1【强制】关于hashCode和equals()的处理，遵循如下规则：
```
1)只要重写equals(),就必须重写hashCode();
2)因为Set存储的是不重复的对象，依据hashCode()和equals()进行判断，所以Set存储的对象必须重写这两个方法。
3)如果自定义对象作为Map的键，那么必须重写hashCode()和equals()。
```
* 2【强制】ArrayList的subList结果不可强转为ArrayList,否则会抛出异常。
```
说明：subList()返回的是ArrayList的内部类SubList，并不是ArrayList，而是ArrayList上的一个视图，对于subList子列表的所有操作最终会反映到原来的ArrayList上。
```
* 3【强制】在subList场景中，高度注意对源集合个数的修改，会导致子列表的遍历、增加、删除均产生ConcurrentModificationException。
* 5【强制】使用工具类Arrays.asList()把数组转换为集合时，不能使用其修改集合相关的方法，它的add/remove/clear方法会抛出UnsupportedOperationException。
```
说明：asList()返回的是一个Arrays内部类，并没有实现集合的修改方法。Arrays.asList()体现的是适配器模式，只是转换接口，后台数据仍然是数组。
String[] str = new String[]{"a","b"};
List list = Arrays.toList(str);
第一种情况：list.add("c");运行时异常
第二种情况：str[0] = "c";那么list.get(0)也会随着修改。
```
* 6【强制】泛型通配符<? extends T>来接收返回的数据，此写法的泛型集合不能使用add方法。
```
说明：苹果装箱后返回一个<? extends Fruit>对象，此对象就不能往里添加任何水果，包括苹果。
```
* 7【强制】不要再foreach循环里面进行元素的remove/add操作。remove元素请使用Iterator方式，如果是并发操作，需要对Iterator对象加锁。
```
反例：
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
for(String temp : a){
    if("1".equals(temp)){
        a.remove(temp);
    }
}
正例：
Iterator<String> it = a.iterator();
while(it.hasNext()){
    String temp = it.next();
    if(删除的条件){
        it.remove();
    }
}
```
* 10【推荐】使用entrySet<K,V>遍历Map类集合KV，而不是keySet方式进行遍历。
```
说明：keySet其实是遍历了2次，一次是为了转为Iterator对象，另一次是从hashMap中取出key所对应的value。而entrySet只是遍历了一次就把key/value都放到了entrySet中，效率更高。如果是JDK8.0,使用Map.foreach()方法。
```
## （六）并发处理
* 1【强制】获取单例对象必须保证线程安全，其中的方法也要保证线程安全。
* 3【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
```
说明:使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者过度切换等问题。
```
* 4【强制】线程池不允许使用Executors去创建，而是通过ThreadPoolExecutors的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
```
说明：Executors返回的线程池对象弊端如下：
1)FixedThreadPool和SingleThreadPool:允许的请求队列长度为Integer.MAX_VALUE,可能会堆积大量请求，从而导致OOM；
2)CachedThreadPool和ScheduledThreadPool:允许创建的线程数量为Integer.MAX_VALUE,可能会创建大量线程，从而导致OOM。
```
* 5【强制】SimpleDateFormat是线程不安全的类，一般不要定义为static变量，如果定义为static变量，则需要加锁，或者使用DateUtils工具类。
```
正例：注意线程安全，使用DateUtils。亦推荐如下处理：
private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
    @Override
    protected DateFormat initialValue(){
        return new SimpleDateFormat("yyyy-MM-dd");
    }
}
如果是jdk8.0的应用，可以使用Instant代替Date,LocalDateTime代替Calendar,DateTimeFormatter代替SimpleDateFormater。
```
*
