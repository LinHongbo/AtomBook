# Set
## HashSet
**1、特点：**
* 由于使用Hash算法来存储查找集合中的元素，故HashSet拥有较好的存储与查找性能；
* HashSet具有Set的定义，即存储的是无序、不重复的元素，HashSet不能保证元素的顺序，顺序可能也添加进HashSet中的顺序不一致;
* HashSet不是同步的，这是集合的特点，大部分集合都是线程不安全的（**注：可能已经存在线程安全的集合类，目前尚未深入接触与了解**）;
* 集合元素可以是null;

**2、总结:**
    使用HashSet必须重写HashSet的equals()方法和hashCode()方法，原因如下：
* hashCode()相同，equals()不同：通过hashCode()计算出在hash队列中的位置，再根据equals()判断两个对象是否相等。hash的作用是快速存储，如果同一个hash位置有大于1个对象，将会严重影响hash的性能。
* hashCode()不同，equals()相同：通过hashCode()计算出在hash队列中的位置，但equals()相同意味着hash队列中存在重复的相同的对象，这与Set的定义矛盾。
* hashCode的计算方式

实例变量类型 | 计算方式
---|---
boolean | hashCode = (f?0:1);
float | hashCode = Float.floatToIntBits(f);
byte、short、char、int | hashCode = (int)f;
double | long l = Double.doubleToLongBits(f);hashCode = (int)(1^(1>>>32));
long | hashCode = (int)(f^(f>>>32));
引用类型 | hashCode = f.hashCode();
* 为了避免直接相加出现的偶然相等，可以通过为各实例变量的hashCode值乘以任意一个质数后再相加。
* 当程序把可变对象添加到HashSet后，尽量要避免去修改集合中的元素中参与计算hashCode()、equals()的实例变量，否则将会导致HashSet无法正确操作这些元素元素。

## LinkedHashSet
1、是HashSet的子类
2、维护了一个链表，表示插入Set的顺序。
3、性能略低于HashSet、但遍历集合内的元素时比HashSet高。

## TreeSet
1、通过红黑树的策略进行排序，使用自然排序或者定制排序对插入元素进行排序（**只会进行一次排序**）
2、与HashSet的策略不同，HashSet是通过hashCode()决定元素的位置，TreeSet则是以Comparable接口或者Comparator接口的排序结果决定元素的位置。
3、同样的，equals()与Comparable接口或者Comparator接口的排序结果必须保持一致。
* equals()相等，而Comparable接口或者Comparator接口的排序结果不为0：集合中存在2个相等的元素，违反set定义。
* equals()不等，而Comparable接口或者Comparator接口的排序结果为0：元素不能成功进行插入，违反set定义。
4、自然排序是通过集合元素本身实现的Comparable接口去进行排序；而定制排序，则是通过集合中的Comparator回调接口去实现排序。
5、同样地，不应对插入TreeSet中的元素进行修改操作。
