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











