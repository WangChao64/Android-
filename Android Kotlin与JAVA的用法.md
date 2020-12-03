# Android Kotlin与JAVA的用法
打印日志
java

System.out.print("Amit Shekhar");
System.out.println("Amit Shekhar");
kotlin

print("Amit Shekhar")
println("Amit Shekhar")
常量与变量
java

String mName = "Java  string name";
final String mName = "Java  string name";
kotlin

var mName = "kotlin string name"
val mName = "kotlin string name"
null声明
java

String mName;
mName = null;
kotlin

var mName : String?
mName = null
空判断
java

if (str != null) {
    int length = str.length();
}
kotlin

str?.let {
    val length = str.length
}
// 更简单的写法
val length = str?.length
// 为null赋予默认值
val length = str?.length?:0
字符串拼接
java

String firstName = "Jiang";
String lastName = "super";
String message = "My name is: " + firstName + " " + lastName;
kotlin

val firstName = "Jiang"
val lastName = "super"
val message = "My name is: $firstName $lastName"
换行
java

String text = "First Line\n" +
          "Second Line\n" +
          "Third Line";
kotlin

val text = """
        |First Line
        |Second Line
        |Third Line
        """.trimMargin()
三元表达式(三目运算符)
java

String text = x > 5 ? "x > 5" : "x <= 5";
kotlin

val text = if (x > 5)
          "x > 5"
       else "x <= 5"
操作符
java

final int andResult  = a & b;
final int orResult   = a | b;
final int xorResult  = a ^ b;
final int rightShift = a >> 2;
final int leftShift  = a << 2;
final int unsignedRightShift = a >>> 2;
kotlin

val andResult  = a and b
val orResult   = a or b
val xorResult  = a xor b
val rightShift = a shr 2
val leftShift  = a shl 2
val unsignedRightShift = a ushr 2
类型判断和转换 (声明式)
java 

if (object instanceof Car) {
}
Car car = (Car) object;
kotlin

if (object is Car) {
}
var car = object as Car
类型判断和转换 (隐式)
java

if (object instanceof Car) {
   Car car = (Car) object;
}
kotlin

if (object is Car) {
   var car = object // 智能转换
}
多重条件
java

if (score >= 0 && score <= 300) { }
kotlin

if (object is Car) {
   var car = object // 智能转换
}
更灵活的case语句
java

int score = 8// 定义分数;
String grade;
switch (score) {
    case 10:
    case 9:
        grade = "Excellent";
        break;
    case 8:
    case 7:
    case 6:
        grade = "Good";
        break;
    case 5:
    case 4:
        grade = "OK";
        break;
    case 3:
    case 2:
    case 1:
        grade = "Fail";
        break;
    default:
        grade = "Fail";
}
 
kotlin

var score = 8// 定义分数
var grade = when (score) {
    9, 10 -> "Excellent"
    in 6..8 -> "Good"
    4, 5 -> "OK"
    in 1..3 -> "Fail"
    else -> "Fail"
}
for循环
java

for (int i = 1; i <= 10 ; i++) { }
 
for (int i = 1; i < 10 ; i++) { }
 
for (int i = 10; i >= 0 ; i--) { }
 
for (int i = 1; i <= 10 ; i+=2) { }
 
for (int i = 10; i >= 0 ; i-=2) { }
 
for (String item : collection) { }
 
for (Map.Entry<String, String> entry: map.entrySet()) { }
 
kotlin

for (i in 1..10) { }
 
for (i in 1 until 10) { }
 
for (i in 10 downTo 0) { }
 
for (i in 1..10 step 2) { }
 
for (i in 10 downTo 1 step 2) { }
 
for (item in collection) { }
 
for ((key, value) in map) { }
更方便的集合操作
java

final List<Integer> listOfNumber = Arrays.asList(1, 2, 3, 4);
 
final Map<Integer, String> keyValue = new HashMap<Integer, String>();
map.put(1, "Amit");
map.put(2, "Ali");
map.put(3, "Mindorks");
 
// Java 9
final List<Integer> listOfNumber = List.of(1, 2, 3, 4);
 
final Map<Integer, String> keyValue = Map.of(1, "Amit",
                               2, "Ali",
                              3, "Mindorks");
 
kotlin

val listOfNumber = listOf(1, 2, 3, 4)
val keyValue = mapOf(1 to "Amit",
             2 to "Ali",
             3 to "Mindorks")
遍历
java

//Java
 
// Java 7 以及更低
for (Car car : cars) {
  System.out.println(car.speed);
}
 
// Java 8+
cars.forEach(car -> System.out.println(car.speed));
 
// Java 7 以及更低
for (Car car : cars) {
  if (car.speed > 100) {
    System.out.println(car.speed);
  }
}
 
// Java 8+
cars.stream().filter(car -> car.speed > 100).forEach(car -> System.out.println(car.speed));
 
 
 kotlin

cars.forEach {
    println(it.speed)
}
 
cars.filter { it.speed > 100 }
      .forEach { println(it.speed)}
方法定义
java

void doSomething() {
   // todo here
}
 
void doSomething(int... numbers) {
   // todo here
}
kotlin

fun doSomething() {
   // logic here
}
 
fun doSomething(vararg numbers: Int) {
   // logic here
}
带返回值的方法
java

int getScore() {
   // logic here
   return score;
}
kotlin

fun getScore(): Int {
   // logic here
   return score
}
 
// 单表达式函数
fun getScore(): Int = score
无结束符号
java

int getScore(int value) {
    // logic here
    return 2 * value;
}
kotlin

fun getScore(value: Int): Int {
   // logic here
   return 2 * value
}
 
// 单表达式函数
fun getScore(value: Int): Int = 2 * value
constructor 构造器
java

public class Utils {
 
    private Utils() { 
      // 私有构造方法
    }
    
    public static int getScore(int value) {
        return 2 * value;
    }
    
}
 
kotlin

class Utils private constructor() {
    companion object {
        fun getScore(value: Int): Int {
            return 2 * value
        }
        
    }
}
 
// 第二个例子
object Utils {
    fun getScore(value: Int): Int {
        return 2 * value
    }
}
Get Set 构造器
java

public class Developer {
 
    private String name;
    private int age;
 
    public Developer(String name, int age) {
        this.name = name;
        this.age = age;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
 
        Developer developer = (Developer) o;
 
        if (age != developer.age) return false;
        return name != null ? name.equals(developer.name) : developer.name == null;
 
    }
 
    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }
 
    @Override
    public String toString() {
        return "Developer{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
 
kotlin

data class Developer(val name: String, val age: Int)
原型扩展(拓展函数)
java

public class Utils {
 
    private Utils() { 
      // 私有构造方法
    }
    
    public static int triple(int value) {
        return 3 * value;
    }
    
}
 
int result = Utils.triple(3);
 
kotlin

fun Int.triple(): Int {
  return this * 3
}
 
var result = 3.triple()
java

public enum Direction {
        NORTH(1),
        SOUTH(2),
        WEST(3),
        EAST(4);
 
        int direction;
 
        Direction(int direction) {
            this.direction = direction;
        }
 
        public int getDirection() {
            return direction;
        }
    }
kotlin

enum class Direction constructor(direction: Int) {
    NORTH(1),
    SOUTH(2),
    WEST(3),
    EAST(4);
 
    var direction: Int = 0
        private set
 
    init {
        this.direction = direction
    }
}
 
 
