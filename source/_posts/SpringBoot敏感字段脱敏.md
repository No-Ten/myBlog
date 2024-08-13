---
title: SpringBoot敏感字段脱敏
date: 2024-8-13 21:55:27
category: 字段脱敏
comment: 'valine'
tags:
  - 字段脱敏
---



# SpringBoot敏感字段脱敏

## 简介

由于某些场景不可以直接展示用户相关信息，如姓名、手机号码、地址等，故需要采用脱敏后的数据进行展示。

## 解决方案

1. 接口直接对数据进行用 * 号脱敏
2. 采用自定义注解方式，在数据序列化的时候做脱敏处理。

第一种比较麻烦，只是适用少量接口，而且不好维护，这里采用第二种。

## 代码

### 创建工程

在pom.xml导入以下依赖，或者确保项目中已经有了Jackson依赖包。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
    <!--如果不是SpringBoot项目，或者项目没有jackson依赖的，则需要下面的依赖-->
    <!--jackson依赖-->
    <!--        <dependency>-->
    <!--            <groupId>com.fasterxml.jackson.core</groupId>-->
    <!--            <artifactId>jackson-core</artifactId>-->
    <!--            <version>2.9.8</version>-->
    <!--        </dependency>-->
    <!--        <dependency>-->
    <!--            <groupId>com.fasterxml.jackson.core</groupId>-->
    <!--            <artifactId>jackson-annotations</artifactId>-->
    <!--            <version>2.9.8</version>-->
    <!--        </dependency>-->
    <!--        <dependency>-->
    <!--            <groupId>com.fasterxml.jackson.core</groupId>-->
    <!--            <artifactId>jackson-databind</artifactId>-->
    <!--            <version>2.9.8</version>-->
    <!--        </dependency>-->

</dependencies>
```



### 编写枚举类

满足不同场景的需求

```java
public enum SensitiveEnum {

    /**
     * 自定义，这个需要指定脱敏的范围
     */
    CUSTOM,
    /**
     * 中文名
     */
    CHINESE_NAME,

    /**
     * 身份证号
     */
    ID_CARD,

    /**
     * 座机号
     */
    FIXED_PHONE,

    /**
     * 手机号
     */
    MOBILE_PHONE,

    /**
     * 地址
     */
    ADDRESS,

    /**
     * 电子邮件
     */
    EMAIL,

    /**
     * 银行卡
     */
    BANK_CARD,

    /**
     * 公司开户银行联号
     */
    CNAPS_CODE
}
```



### 自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonSerialize(using = SensitiveSerialize.class)
@Target({ElementType.FIELD})
public @interface SensitiveWrapped {

    /**
     * 脱敏类型，自定义类型需要指定其他参数
     */
    SensitiveEnum type();

    /**
     * 前置不需要打码的长度
     */
    int prefixNoMaskLen() default 1;

    /**
     * 后置不需要打码的长度
     */
    int suffixNoMaskLen() default 1;

    /**
     * 用什么打码
     */
    String symbol() default "*";
}
```



### 自定义脱敏序列化类

```java
@AllArgsConstructor
@NoArgsConstructor
public class SensitiveSerialize extends JsonSerializer<String> implements ContextualSerializer {
    /**
     * 脱敏类型
     */
    private SensitiveEnum type;

    private int prefixNoMaskLen;
    private int suffixNoMaskLen;
    private String symbol;


    @Override
    public void serialize(String s, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        // 序列化方法
        switch (this.type) {
            case CHINESE_NAME: {
                jsonGenerator.writeString(SensitiveInfoUtils.chineseName(s));
                break;
            }
            case ID_CARD: {
                jsonGenerator.writeString(SensitiveInfoUtils.idCardNum(s));
                break;
            }
            case FIXED_PHONE: {
                jsonGenerator.writeString(SensitiveInfoUtils.fixedPhone(s));
                break;
            }
            case MOBILE_PHONE: {
                jsonGenerator.writeString(SensitiveInfoUtils.mobilePhone(s));
                break;
            }
            case ADDRESS: {
                jsonGenerator.writeString(SensitiveInfoUtils.address(s, 4));
                break;
            }
            case EMAIL: {
                jsonGenerator.writeString(SensitiveInfoUtils.email(s));
                break;
            }
            case BANK_CARD: {
                jsonGenerator.writeString(SensitiveInfoUtils.bankCard(s));
                break;
            }
            case CNAPS_CODE: {
                jsonGenerator.writeString(SensitiveInfoUtils.cnapsCode(s));
                break;
            }
            case CUSTOM: {
                jsonGenerator.writeString(SensitiveInfoUtils.custom(s, prefixNoMaskLen, suffixNoMaskLen, symbol));
                break;
            }
            default:
                throw new IllegalArgumentException("Unknown type enum " + this.type);
        }
    }

    @Override
    public JsonSerializer<?> createContextual(SerializerProvider serializerProvider, BeanProperty beanProperty) throws JsonMappingException {
        // 上下文序列
        // 为空直接跳过
        if (beanProperty != null) {
            // 非 String 类直接跳过
            if (Objects.equals(beanProperty.getType().getRawClass(), String.class)) {
                SensitiveWrapped sensitiveWrapped = beanProperty.getAnnotation(SensitiveWrapped.class);
                if (sensitiveWrapped == null) {
                    sensitiveWrapped = beanProperty.getContextAnnotation(SensitiveWrapped.class);
                }
                if (sensitiveWrapped != null) {
                    // 如果能得到注解，就将注解的 value 传入 SensitiveSerialize
                    return new SensitiveSerialize(sensitiveWrapped.type(), sensitiveWrapped.prefixNoMaskLen(),
                            sensitiveWrapped.suffixNoMaskLen(), sensitiveWrapped.symbol());
                }
            }
            return serializerProvider.findValueSerializer(beanProperty.getType(), beanProperty);
        }
        return serializerProvider.findNullValueSerializer(beanProperty);
    }

//    public SensitiveSerialize() {}
//
//    public SensitiveSerialize(final SensitiveEnum type,int prefixNoMaskLen,int suffixNoMaskLen,String symbol) {
//        this.type = type;
//    }
}
```



### 脱敏工具类

这里有两个工具类，一个是采用apache.commons，一个是采用String.replaceAll

```java
import org.apache.commons.lang3.StringUtils;
public class SensitiveInfoUtils {

    /**
     * [中文姓名] 只显示第一个汉字，其他隐藏为2个星号<例子：李**>
     */
    public static String chineseName(final String fullName) {
        if (StringUtils.isBlank(fullName)) {
            return "";
        }
        final String name = StringUtils.left(fullName, 1);
        return StringUtils.rightPad(name, StringUtils.length(fullName), "*");
//        以下是优化内容
//        if (StringUtils.length(fullName) < 3){
//         return StringUtils.rightPad(name, StringUtils.length(fullName), "*");
//        }
//        return StringUtils.rightPad(name, StringUtils.length(fullName) - 1, "*").concat(StringUtils.right(fullName, 1));
    }

    /**
     * [中文姓名] 只显示第一个汉字，其他隐藏为2个星号<例子：李**>
     */
    public static String chineseName(final String familyName, final String givenName) {
        if (StringUtils.isBlank(familyName) || StringUtils.isBlank(givenName)) {
            return "";
        }
        return chineseName(familyName + givenName);
    }

    /**
     * [身份证号] 显示最后四位，其他隐藏。共计18位或者15位。<例子：420**********5762>
     */
    public static String idCardNum(final String id) {
        if (StringUtils.isBlank(id)) {
            return "";
        }

        return StringUtils.left(id, 3).concat(StringUtils
                .removeStart(StringUtils.leftPad(StringUtils.right(id, 4), StringUtils.length(id), "*"),
                        "***"));
    }

    /**
     * [固定电话] 后四位，其他隐藏<例子：****1234>
     */
    public static String fixedPhone(final String num) {
        if (StringUtils.isBlank(num)) {
            return "";
        }
        return StringUtils.leftPad(StringUtils.right(num, 4), StringUtils.length(num), "*");
    }

    /**
     * [手机号码] 前三位，后四位，其他隐藏<例子:138******1234>
     */
    public static String mobilePhone(final String num) {
        if (StringUtils.isBlank(num)) {
            return "";
        }
        return StringUtils.left(num, 3).concat(StringUtils
                .removeStart(StringUtils.leftPad(StringUtils.right(num, 4), StringUtils.length(num), "*"),
                        "***"));

    }

    /**
     * [地址] 只显示到地区，不显示详细地址；我们要对个人信息增强保护<例子：北京市海淀区****>
     *
     * @param sensitiveSize 敏感信息长度
     */
    public static String address(final String address, final int sensitiveSize) {
        if (StringUtils.isBlank(address)) {
            return "";
        }
        final int length = StringUtils.length(address);
        return StringUtils.rightPad(StringUtils.left(address, length - sensitiveSize), length, "*");
    }

    /**
     * [电子邮箱] 邮箱前缀仅显示第一个字母，前缀其他隐藏，用星号代替，@及后面的地址显示<例子:g**@163.com>
     */
    public static String email(final String email) {
        if (StringUtils.isBlank(email)) {
            return "";
        }
        final int index = StringUtils.indexOf(email, "@");
        if (index <= 1) {
            return email;
        } else {
            return StringUtils.rightPad(StringUtils.left(email, 1), index, "*")
                    .concat(StringUtils.mid(email, index, StringUtils.length(email)));
        }
    }

    /**
     * [银行卡号] 前六位，后四位，其他用星号隐藏每位1个星号<例子:6222600**********1234>
     */
    public static String bankCard(final String cardNum) {
        if (StringUtils.isBlank(cardNum)) {
            return "";
        }
        return StringUtils.left(cardNum, 6).concat(StringUtils.removeStart(
                StringUtils.leftPad(StringUtils.right(cardNum, 4), StringUtils.length(cardNum), "*"),
                "******"));
    }

    /**
     * [公司开户银行联号] 公司开户银行联行号,显示前两位，其他用星号隐藏，每位1个星号<例子:12********>
     */
    public static String cnapsCode(final String code) {
        if (StringUtils.isBlank(code)) {
            return "";
        }
        return StringUtils.rightPad(StringUtils.left(code, 2), StringUtils.length(code), "*");
    }

    /**
     * 自定义类型脱敏函数
     * @param source 原始字符串
     * @param prefixNoMaskLen 前置不脱敏的长度
     * @param suffixNoMaskLen 后置不脱敏的长度
     * @param symbol 用什么符号脱敏
     * @return 脱敏后的字符串
     */
    public static String custom(String source, int prefixNoMaskLen, int suffixNoMaskLen, String symbol) {
        if (StringUtils.isBlank(source)){
            return "";
        }
        StringBuilder sb = new StringBuilder();

        int n = source.length();

        // 既处理前置，也处理后置和脱敏
//        for (int i = 0 ; i < n; i++){
//            // 前置不脱敏的字符长度
//            if (i < prefixNoMaskLen) {
//                sb.append(source.charAt(i));
//                continue;
//            }
//
//            // 后置不脱敏的字符长度
//            if (i > (n - suffixNoMaskLen - 1)){
//                sb.append(source.charAt(i));
//                continue;
//            }
//
//            // 需要脱敏字符
//            sb.append(symbol);
//        }

        for (int i = 0; i < n; i++) {
            // 优化，不用那么多continue，只处理头脱敏的即可，既  prefix < i < suffix
            // 只处理要脱敏的，其他的不处理
            if (prefixNoMaskLen <= i && i <= (n - suffixNoMaskLen - 1)){
                sb.append(symbol);
                continue;
            }
            sb.append(source.charAt(i));
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        String custom = custom("12345", -1, -1, "@");
        System.out.println(custom);

        System.out.println(chineseName("张三"));
        System.out.println(chineseName("张李四"));
        System.out.println(chineseName("张李四啊留"));
    }
}
```



脱敏工具类2

```java
import org.apache.commons.lang3.StringUtils;
public class PrivacyUtils {

    /**
     * 隐藏电话号码
     *
     * @param phone
     * @return
     */
    public static String hidePhone(String phone) {
        if (StringUtils.isEmpty(phone)) {
            return null;
        }
        return phone.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
    }

    /**
     * 隐藏邮件
     *
     * @param email
     * @return
     */
    public static String hideEmail(String email) {
        if (StringUtils.isEmpty(email)) {
            return null;
        }
        return email.replaceAll("(\\w?)(\\w+)(\\w)(@\\w+(.[a-z]+)?)", "$1****$3$4");
    }

    /**
     * 隐藏身份证号码
     *
     * @param idCard
     * @return
     */
    public static String hideIdCard(String idCard) {
        if (StringUtils.isEmpty(idCard)) {
            return null;
        }
        return idCard.replaceAll("(\\d{4})\\d{10}(\\d{4})", "$1****$2");
    }

    public static void main(String[] args) {
        String s = hidePhone("12334542345");
        System.out.println(s);
        String s1 = hideEmail("12345677@qq.com");
        System.out.println(s1);

        String email = SensitiveInfoUtils.email("2345677@qq.com");
        System.out.println(email);
        // 441111200001011233
        String s2 = hideIdCard("441111200001011233");
        System.out.println(s2);

        String s3 = SensitiveInfoUtils.idCardNum("441111200001011233");
        System.out.println(s3);

    }
}
```



### 测试实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserEntity {

    /**
     * 用户ID
     */
    private Long userId;

    /**
     * 用户姓名
     */
    private String name;

    /**
     * 手机号
     */
    @SensitiveWrapped(type = SensitiveEnum.MOBILE_PHONE)
    private String mobile;

    /**
     * 身份证号码
     */
    @SensitiveWrapped(type = SensitiveEnum.ID_CARD)
    private String idCard;

    /**
     * 年龄
     */
    private String sex;

    /**
     * 性别
     */
    private int age;

}
```



### 测试类



```java
public class SensitiveTest {
    public static void main(String[] args) throws JsonProcessingException {
        UserEntity userEntity = new UserEntity();
        userEntity.setUserId(1L);
        userEntity.setName("张三");
        userEntity.setMobile("18000000001");
        userEntity.setIdCard("420117200001011000008888");
        userEntity.setAge(20);
        userEntity.setSex("男");

        ObjectMapper mapper = new ObjectMapper();
        System.out.println(mapper.writeValueAsString(userEntity));
    }
}
```

结果

```json
{"userId":1,"name":"张三","mobile":"180****0001","idCard":"420*****************8888","sex":"男","age":20}
```



用过SpringBoot启动，访问localhost:8080/hello

```java
@RestController
public class TestController {
    @RequestMapping("/hello")
    public UserEntity hello() {
        UserEntity userEntity = new UserEntity();
        userEntity.setUserId(1L);
        userEntity.setName("张三");
        userEntity.setMobile("18000000001");
        userEntity.setIdCard("420117200001011000008888");
        userEntity.setAge(20);
        userEntity.setSex("男");
        return userEntity;
    }
}
```

结果如下

![image-20240813215359691](SpringBoot%E6%95%8F%E6%84%9F%E5%AD%97%E6%AE%B5%E8%84%B1%E6%95%8F/image-20240813215359691.png)



仓库地址：

https://github.com/No-Ten/springboot-field-desensitization



参考：

https://mp.weixin.qq.com/s/xjnwBzNizMQO6S5WFOaf5A

https://blog.csdn.net/qq_36737803/article/details/122366043







