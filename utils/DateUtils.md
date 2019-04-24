# 日期

## 字符串和日期相互转换

```java
package com.univer.crawl.util;

import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;

import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.time.DateUtils;

import lombok.Generated;

@Generated
public class DateUtil {
	
	private static final DateTimeFormatter DATE_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
	
	//date转字符串
	public static String format(Date date) {
		if(null == date) {
			return null;
		}
		return DATE_FORMATTER.format(date.toInstant().atZone(ZoneId.of("Asia/Shanghai")).toLocalDateTime());
	}
	//字符串转date
	public static Date parse(String date) {
		if(StringUtils.isBlank(date)) {
			return null;
		}
		LocalDateTime localDateTime = LocalDateTime.parse(date, DATE_FORMATTER);
		ZoneId zoneId = ZoneId.of("Asia/Shanghai");
        ZonedDateTime zdt = localDateTime.atZone(zoneId);
        return Date.from(zdt.toInstant());
    }
	
	
	//得到当天的0时
	public static Date getDayZero(Date date) {
		if(null == date) {
			return null;
		}
	
		Date dayZero = DateUtils.setSeconds(DateUtils.setMinutes(DateUtils.setHours(date, 0), 0), 0);
		return dayZero;
	}
	//得到当天的23:59:59
	public static Date getDayTwelve(Date date) {
		if(null == date) {
			return null;
		}
	
		Date dayTwelve = DateUtils.setSeconds(DateUtils.setMinutes(DateUtils.setHours(date, 23), 59), 59);
		return dayTwelve;
	}
	
	 /**
     * @author wumingsheng
     * 计算两个时间点之间的天数
     */
    public static long getBetweenDay(Date start, Date end) {
    	
    	LocalDate startLocalDate = LocalDate.ofInstant(start.toInstant(),  ZoneId.of("Asia/Shanghai"));
    	LocalDate endLocalDate = LocalDate.ofInstant(end.toInstant(),  ZoneId.of("Asia/Shanghai"));
        return endLocalDate.toEpochDay() - startLocalDate.toEpochDay();
    }

}

```


