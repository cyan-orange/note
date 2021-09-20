# 定义枚举
使用说明

1. 使用 enum 定义的枚举类默认继承了 java.lang.Enum类，因此不能再继承其他类

1. 枚举类的构造器只能使用 private 权限修饰符

1. 枚举类的所有实例必须在枚举类中显式列出(, 分隔 ; 结尾)。列出的实例系统会自动添加 public static final 修饰

1. 必须在枚举类的第一行声明枚举类对象
```java
public enum MessageCodeEnum {
    SUCCESS(1, "成功"),
    ERROR(0, "失败");

    private final int code;
    private final String message;

    MessageCodeEnum(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}

```


# Enum类的主要方法


1. values()方法：返回枚举类型的对象数组。该方法可以很方便地遍历所有的枚举值。
1. valueOf(String str)：可以把一个字符串转为对应的枚举类对象。要求字符串必须是枚举类对象的“名字”。如不是，会有运行时异常：`IllegalArgumentException`。
1. toString()：返回当前枚举类对象常量的名称
```java
MessageCodeEnum[] values = MessageCodeEnum.values();
//values()
System.out.println(Arrays.asList(values)); //[SUCCESS, ERROR]

//toString()
String s = MessageCodeEnum.SUCCESS.toString();
System.out.println(s); //SUCCESS
```
