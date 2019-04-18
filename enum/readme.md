# 枚举


```java

import lombok.AllArgsConstructor;
import lombok.Generated;
import lombok.Getter;

@AllArgsConstructor
@Getter
@Generated
public enum DataTypeEnum {
	/** 视频 */
	VIDEO("video", "视频"),
	/** 图片 */
	IMAGE("image", "图片"),
	/** 文本 */
	TXT("txt", "文本");

	private String dataType;

	private String desc;

	public static String getDesc(String dataType) {
		for (DataTypeEnum en : DataTypeEnum.values()) {
			if (en.dataType.equals(dataType)) {
				return en.desc;
			}
		}
		return null;
	}

}

```


