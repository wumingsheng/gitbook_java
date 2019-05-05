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



























