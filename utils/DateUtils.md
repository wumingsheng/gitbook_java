# 日期

## 字符串和日期相互转换

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;


public class DateUtil {
	
	private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
	
	//date转字符串
	public static String format(Date date) {
		return formatter.format(date.toInstant().atZone(ZoneId.of("Asia/Shanghai")).toLocalDateTime());
	}
	//字符串转date
	public static Date parse(String date) {
		LocalDateTime localDateTime = LocalDateTime.parse(date, formatter);
		ZoneId zoneId = ZoneId.of("Asia/Shanghai");
        ZonedDateTime zdt = localDateTime.atZone(zoneId);
        return Date.from(zdt.toInstant());
    }

    //获取当天的00:00:00
	public static Date getDayZero(Date date) {
		if(null == date) {
			return null;
		}
		long time = date.getTime();
		long zero=time/(1000*3600*24)*(1000*3600*24)-TimeZone.getDefault().getRawOffset();//今天零点零分零秒的毫秒数
		return new Date(zero);
	}
	//获取当天的23:59:59
	public static Date getDayTwelve(Date date) {
		if(null == date) {
			return null;
		}
		long time = date.getTime();
		long zero=time/(1000*3600*24)*(1000*3600*24)-TimeZone.getDefault().getRawOffset();//今天零点零分零秒的毫秒数
        long twelve=zero+24*60*60*1000-1;//今天23点59分59秒的毫秒数
        return new Date(twelve);
	}

}
```


