## 配置环境变量

​	

```chinese
java环境变量配置：新建系统环境变量：JAVA_HOME，变量值：  Jdk路径 如D：\jdk
新建系统环境变量：CLASSPATH 变量值：;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%lib\tools.jar
在Path变量中：添加： %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
运行cmd ，分别输入java,和javac检测是否能运行。java -version检测版本

tomcat环境变量配置：新建系统环境变量：CATALINE_BASE 变量值： tomcat安装路线 如D:\spache-tomcat-7.085
新建系统环境变量：CATALINE_HOME 变量值：同CATALINE_BASE ，tomcat安装路线
在Path变量中：添加：%CATALINE_HOME%\lib;%CATALINE_HOME%\bin

Mysql环境变量基本同上。
python环境变量基本同上。目的都是让系统更好更快的找到所需软件运行的环境。

```

## 数组、String、List、Set之间的相互转换问题

## 数组和String

## String转换为char[]

**String转换为char[]的一种方法：**

```java
public static void stringToChar(String str){
		char[] chars1 = str.toCharArray();			//通过toCharArray方法
	}
```

## 	**toCharArray()的源码:**

```java
 public char[] toCharArray() {
        // Cannot use Arrays.copyOf because of class initialization order issues
        char result[] = new char[value.length]; //构造返回的char[]变量
        System.arraycopy(value, 0, result, 0, value.length); 
        //通过arraycopy将String的value值拷贝到result上，返回result
        return result;
    }

```

## char[]转换为String

**char[]数组转换为String的两种现有方法：**  

```java
public static void charToString(){

		char[] chars = {'1','2','3','4','5','6'};
		String str1 = new String(chars);			//通过String构造函数
		String str2 = String.valueOf(chars);		//通过String类型转换，实际也是new String(char[])
		
}
 public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length); 
        //深拷贝一个新的char[]数组给String的value(对于基本类型而言都是深拷贝)
  }
   public static String valueOf(char data[]) {
        return new String(data); //实际上还是String的构造函数
   }

```

## 数组和List

------

## 包装类数组转换为List

```java
public static void arrayToList(){
		
		Character[] chars = {'1','2','3','4','5','6','7','8','9'};
		
		//方式一
		List<Character> list1 = Arrays.asList(chars); //通过Arrays的工具类
		//方式二
		List<Character> list2 = new ArrayList<>();   //通过Collections工具类
		Collections.addAll(list2,chars);

}
//Arrays工具类的源码：

   public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);   //返回Arrays类内部的静态内部类ArrayList实例对象
    }
1
2
3
//Collections.addAll工具类的源码：

   public static <T> boolean addAll(Collection<? super T> c, T... elements) {
        boolean result = false;
        for (T element : elements) //通过遍历数组
            result |= c.add(element); //每次都把数组插入到集合c中
        return result;
    }

```

## List转换为包装类数组

**List转换为包装类数组的一种方式：**

```java
public static void listToArray(){
		List<Character> list = new ArrayList<>();
		list.add('1');
		list.add('2');
		Character[] chars =  list.toArray(new Character[list.size()]);
		//注意toArray()的参数
}
//关于源码的实现，就要具体看是那种List了，从List接口中，我们也能知道一些信息

 <T> T[] toArray(T[] a); //参数是什么类型就返回什么类型的数组
```

**注意：**
我们这里特地强调了是List和包装类数组之间的转换。因为集合只支持对包装类进行操作。
所以你如过要进行基本类型数组和List之间的转换，比如char[]转换为List,要先转换成它的包装类型数组Character[]

## String和List

------

## 分隔符String转换为List

**String转换为List的两种方式:**

```java
	//原理就是首先将String转换成String[]数组，再通过工具类转换为List
	public static void stringToList (String str){
	
		String str1 = "123456";
		//方式一
		List<String>list1  = Arrays.asList(str.split("")); //str.split()返回一个String[]数组
		//方式二
		List<String>list2 = new ArrayList<>();
		Collections.addAll(list2, str.split(""));
	}

```

所以有**两个步骤**是

- 首先要将String转换为包装类型（如Character[]）或String[]数组
- 再把包装类型数组转换成List.

## List转换为分隔符String

**List转换为String的一种方式：**

```
public static void listToString(){
		List<String> list = Arrays.asList("1","2","3","4");
		String str = String.join("", list); //"1234"
}

```

## List和Set

------

## List转换为Set

```java
public static void listToSet(){
		 List<String> list = new ArrayList<>();
		 Set<String> set = new HashSet<>(list);   //直接构造函数即可
}

```

## Set转换为List

```java
 public static void setToList(){
		 Set<String> set = new HashSet<>();
		 List<String> list = new ArrayList<>(set); //直接构造函数即可
}

```

## String和Set

------

## String转换为Set

```java
	public static void stringToSet(){
		String str = "12345";
		String[] strs= str.split("");
		
		//方式一
		List<String> list1 = Arrays.asList(strs);
		Set<String> set1 = new HashSet<>(list1);

		//方式二
		Set<String> set2 = new HashSet<>();
		Collections.addAll(set2, strs);
	}
/**
**
方式一有三个步骤：

String转换为String[]数组 或包装类型数组（如Character[]）
将数组转换为List,可以使用Arrays或Collections工具类
将list转换为Set
方式二有两个步骤

String转换为String[]数组 或包装类型数组（如Character[]）
使用Collections工具类将数组转换为Set
*
**/
```



------------------------------------------------
## Set转换为String

```java
	public static void setToString(){
		Set<String> set = new HashSet<>();
		String.join("", set);
	}

```

## 数组和Set

------

## 数组转换为Set

```java
public static void arrayToSet(){
		Character[] chars = {'1','2','3','4'};

		//方式一
		List<Character> list = Arrays.asList(chars);
		Set<Character> set12 = new HashSet<>(list);

		//方式二
		Set<Character> set1 = new HashSet<>();
		Collections.addAll(set1, chars);
}
```

两种方式，同样是受到不同工具类的影响
第一种方式的**两个步骤：**

- 数组通过Arrays或Collections工具类转换为List
- 再把list转换为set

第二种方式的**一个步骤:**

- 通过Collections将数组转换为Set

## Set转换为数组

```java
	public static void setToArray(){
		Set<Character> set = new HashSet<>();
		Character[] chars = set.toArray(new Character[set.size()]);
	}

```

## List的分组操作

```
1，跟据某个属性分组OfficeId

 Map<String, List<IncomeSumPojo>> collect = list.stream().collect(Collectors.groupingBy(IncomeSumPojo::getOfficeId));

2，根据某个属性分组OfficeId，汇总某个属性Money

Map<String, Double> collect = list.stream().collect(Collectors.groupingBy(IncomeSumPojo::getOfficeId,Collectors.summingDouble(IncomeSumPojo::getMoney)));

3，根据某个属性添加条件过滤数据，

list = list.stream().filter(u -> !u.getAmount().equals("0.00")).collect(Collectors.toList());

4，判断一组对象里面有没有属性值是某个值

List<Menu> menuList = UserUtils.getMenuList();
boolean add = menuList.stream().anyMatch(m -> "plan:ctPlan:add".equals(m.getPermission()));

5，取出一组对象的某个属性组成一个新集合

List<String> tableNames=list.stream().map(User::getMessage).collect(Collectors.toList());

6，list去重复

list = list.stream().distinct().collect(Collectors.toList());

```

