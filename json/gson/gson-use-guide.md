# gson使用指南

## 1. 依赖包

```xml
<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
``` 

## 2. gson的基本用法

Gson提供了**`fromJson()`** 和**`toJson()`** 两个直接用于解析和生成的方法

### 2.1 基本数据类型的解析与生成

```java
Gson gson = new Gson();
Integer fromJson = gson.fromJson("100", Integer.class);// 100
Boolean fromJson2 = gson.fromJson("true", Boolean.class);// true
String fromJson3 = gson.fromJson("100", String.class);// 100
log.info("{},{},{}", fromJson, fromJson2, fromJson3);
String jsonNumber = gson.toJson(100);       // 100
String jsonBoolean = gson.toJson(false);    // false
String jsonString = gson.toJson("String"); //"String"

log.info("{},{},{}",jsonNumber, jsonBoolean, jsonString);
```

### 2.2 POJO类的生成与解析


```java
import java.util.Date;
import java.util.List;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class User {
	
	
	private String username;
	
	private Integer age;
	
	private Date birthday;
	
	private Boolean ifMarried;
	
	private String[] addr;
	
	private List<Integer> scores;
	

}
```
生成json
```java

import java.util.Arrays;
import java.util.Date;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import com.example.gsondemo.po.User;
import com.google.gson.Gson;

import lombok.extern.slf4j.Slf4j;

@RestController
@Slf4j
public class UserController {

	@GetMapping("/")
	public Object name() {
		Gson gson = new Gson();

		User user = new User("woms", 32, new Date(), true, new String[] { "bj", "sx" }, Arrays.asList(34, 45));

		String json = gson.toJson(user);

		log.info(json);
		return json;

	}

	

}

```

```bash
$ curl localhost:8080 -s | jq
{
  "username": "woms",
  "age": 32,
  "birthday": "May 5, 2019, 12:13:54 PM",
  "ifMarried": true,
  "addr": [
    "bj",
    "sx"
  ],
  "scores": [
    34,
    45
  ]
}

$ curl localhost:8080 -s | jq -R
"{\"username\":\"woms\",\"age\":32,\"birthday\":\"May 5, 2019, 12:16:48 PM\",\"ifMarried\":true,\"addr\":[\"bj\",\"sx\"],\"scores\":[34,45]}"


```


解析json

```java
Gson gson = new Gson();

		User user = gson.fromJson("{\"username\":\"woms\",\"age\":32,\"birthday\":\"May 5, 2019, "
				+ "12:16:48 PM\",\"ifMarried\":true,\"addr\":[\"bj\",\"sx\"],\"scores\":[34,45]}", User.class);
```


## 3. 属性重命名 `@SerializedName` 注解的使用

```java

import java.util.Date;
import java.util.List;

import com.google.gson.annotations.SerializedName;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class User {
	
	
	private String username;
	
	private Integer age;
	
	private Date birthday;
	@SerializedName("if_married")
	private Boolean ifMarried;
	
	private String[] addr;
	
	private List<Integer> scores;
	

}
```

```bash
$ curl localhost:8080 -s|jq 
{
  "username": "woms",
  "age": 32,
  "birthday": "May 5, 2019, 12:54:23 PM",
  "if_married": true,
  "addr": [
    "bj",
    "sx"
  ],
  "scores": [
    34,
    45
  ]
}

```
### 3.1 为POJO字段提供备选属性名



`@SerializedName`注解提供了两个属性，上面用到了其中一个，别外还有一个属性alternate，接收一个String数组。



> 注：alternate需要2.4版本


```java
@SerializedName(value = "emailAddress", alternate = {"email", "email_address"})
public String emailAddress;
```


当上面的三个属性(email_address、email、emailAddress)都中出现任意一个时均可以得到正确的结果。



注：当多种情况同时出时，以最后一个出现的值为准。


```java
Gson gson = new Gson();

String json = "{\"name\":\"怪盗kidou\",\"age\":24,\"emailAddress\":\"ikidou_1@example.com\",\"email\":\"ikidou_2@example.com\",\"email_address\":\"ikidou_3@example.com\"}";

User user = gson.fromJson(json, User.class);

System.out.println(user.emailAddress); // ikidou_3@example.com

```


## 4. Gson中使用泛型

JSON中的集合有两种：数组、list

如果是数组的话，没问题，因为数组本身就是一种类型，如果是list的话会有问题，因为设计到了泛型

```java
@Data
public class Response<T> {
	
	private Integer code;
	
	private String message;
	
	private T data;

}

```

```java
Gson gson = new Gson();

Response<List<User>> response = new Response<>();
List<User> list = new ArrayList<>();
list.add(new User("woms", 32, new Date(), true, new String[] { "bj", "sx" }, Arrays.asList(34, 45)));
response.setCode(200);
response.setMessage("success");
response.setData(list);

return gson.toJson(response);
```

```bash
$ curl localhost:8080 -s|jq 
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "username": "woms",
      "age": 32,
      "birthday": "May 5, 2019, 1:04:53 PM",
      "if_married": true,
      "addr": [
        "bj",
        "sx"
      ],
      "scores": [
        34,
        45
      ]
    }
  ]
}


```

解析json使用`new TypeToken<Response<List<User>>>(){}.getType()`

```java
	Gson gson = new Gson();

	Response<List<User>> fromJson = gson.fromJson("{\"code\":200,\"message\":\"success\",\"data\":[{\"username\":\"woms\",\"age\":32,\"birthday\":\"May 5, 2019, 1:07:02 PM\",\"if_married\":true,\"addr\":[\"bj\",\"sx\"],\"scores\":[34,45]}]}"
		, new TypeToken<Response<List<User>>>(){}.getType());
```



## 5. 使用GsonBuilder导出null值、格式化输出、日期时间

一般情况下Gson类提供的 API已经能满足大部分的使用场景，但我们需要更多更特殊、更强大的功能时，这时候就引入一个新的类 GsonBuilder。



GsonBuilder从名上也能知道是用于构建Gson实例的一个类，要想改变Gson默认的设置必须使用该类配置Gson。



### 5.1 默认null字段是不显示的

```java
Gson gson = new Gson();

Response<List<User>> response = new Response<>();
List<User> list = new ArrayList<>();
list.add(new User("woms", null, new Date(), true, new String[] { "bj", "sx" }, Arrays.asList(34, 45)));
response.setCode(200);
response.setMessage("success");
response.setData(list);

return gson.toJson(response);
```

age字段我们设置了null, 生成的json是没有age字段的

```bash
$ curl localhost:8080 -s|jq
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "username": "woms",
      "birthday": "May 5, 2019, 1:37:46 PM",
      "if_married": true,
      "addr": [
        "bj",
        "sx"
      ],
      "scores": [
        34,
        45
      ]
    }
  ]
}

```


如果我们这样生成一个gson

```java
GsonBuilder gsonBuilder = new GsonBuilder();
Gson gson = gsonBuilder.serializeNulls().create();

Response<List<User>> response = new Response<>();
List<User> list = new ArrayList<>();
list.add(new User("woms", null, new Date(), true, new String[] { "bj", "sx" }, Arrays.asList(34, 45)));
response.setCode(200);
response.setMessage("success");
response.setData(list);

return gson.toJson(response);
```

```json
$ curl localhost:8080 -s|jq
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "username": "woms",
      "age": null,  //这里是可以看到这个key的
      "birthday": "May 5, 2019, 1:42:00 PM",
      "if_married": true,
      "addr": [
        "bj",
        "sx"
      ],
      "scores": [
        34,
        45
      ]
    }
  ]
}

```


### 5.2 格式化日期

```java

GsonBuilder gsonBuilder = new GsonBuilder();
		Gson gson = gsonBuilder.serializeNulls().setDateFormat("yyyy-MM-dd").create();

```


```json
$ curl localhost:8080 -s|jq
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "username": "woms",
      "age": null,
      "birthday": "2019-05-05",
      "if_married": true,
      "addr": [
        "bj",
        "sx"
      ],
      "scores": [
        34,
        45
      ]
    }
  ]
}

```


## 6. 字段过滤的几种方法


### 6.1 基于@Expose注解


```java
GsonBuilder gsonBuilder = new GsonBuilder();
Gson gson = gsonBuilder.serializeNulls()
	.excludeFieldsWithoutExposeAnnotation()//启用expose注解  只有被注解的字段才会有效
	.setDateFormat("yyyy-MM-dd")
	.create();
```

```java
@Expose //
@Expose(deserialize = true,serialize = true) //序列化和反序列化都都生效
@Expose(deserialize = true,serialize = false) //反序列化时生效
@Expose(deserialize = false,serialize = true) //序列化时生效
@Expose(deserialize = false,serialize = false) // 和不写一样
```

### 6.2 基于版本



Gson在对基于版本的字段导出提供了两个注解 @Since 和 @Until,和GsonBuilder.setVersion(Double)配合使用。@Since 和 @Until都接收一个Double值。

```java
import com.google.gson.annotations.Since;
import com.google.gson.annotations.Until;

import lombok.Data;

@Data
public class Response<T> {
	
	
	@Until(3.2)
	private Integer code;
	@Since(3.2)
	private String message;
	
	
	private T data;

}
```

```java
GsonBuilder gsonBuilder = new GsonBuilder();
		Gson gson = gsonBuilder.serializeNulls()
				.setVersion(3.2)//设置版本号
				.setDateFormat("yyyy-MM-dd").create();
```


### 6.3 基于访问修饰符

```java
@Data
public class Response<T> {
	

	public Integer code;

	private String message;
	
	
	private T data;

}

```

```java
import java.lang.reflect.Modifier;

GsonBuilder gsonBuilder = new GsonBuilder();
Gson gson = gsonBuilder.serializeNulls()
		.excludeFieldsWithModifiers(Modifier.PUBLIC)
		.setDateFormat("yyyy-MM-dd").create();
```


## 7. 禁用特殊字符转移

```java
Gson gson = new GsonBuilder().disableHtmlEscaping().create();
```


