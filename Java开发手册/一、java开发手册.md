#  一、java开发手册

## 1.1、NullPointerException最佳实践

### 1.1.1、 **equals() 和 equalsIgnoreCase()** 的使用

```java
Object unknownObject = null;

// 方式一：抛异常
if (unknownObject.equals( "knownObject" )){
    System.err.println( "This may result in NullPointerException if unknownObject is null" );
}
// 方式二：不抛异常
if ( "knownObject" .equals(unknownObject)){
     System.err.println( "better coding avoided NullPointerException" );
}
```

### 1.1.2、**valueOf() 和 toString()** 的使用

```java
Object unknownObject = null;

// 方式一：抛异常
System.out.println(unknownObject.toString());
// 方式二：不抛异常
System.out.println(String.valueOf(unknownObject))
```

### 1.1.3、**使用null安全的方法和库** 

