# springboot 时间格式化问题

## 全局配置

```yml
 # 时间默认转时间戳
  jackson:
    serialization:
      write-dates-as-timestamps: true
      
      
      #或者
    jackson
       date-format：yyyy-MM-dd HH:mm:ss
       time-zone：GMT+8
```

## 局部配置

```
@JsonFormat(pattern = "yyyy-MM-dd", timezone="GMT+8")
private Date releaseDate;
#自定义日期序列化与反序列化，如下所示：
/**
 * 日期序列化
 */
public class DateJsonSerializer extends JsonSerializer<Date> {
    @Override
    public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws
            IOException {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        jsonGenerator.writeString(dateFormat.format(date));
    }
}

/**
 * 日期反序列化
 */
public class DateJsonDeserializer extends JsonDeserializer<Date> {

    @Override
    public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
        try {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
            return dateFormat.parse(jsonParser.getText());
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}

/**
 * 使用方式
 */
@JsonSerialize(using = DateJsonSerializer.class)
@JsonDeserialize(using = DateJsonDeserializer.class)
private Date releaseDate;



```

## mysql **如何避免出现时区问题**

```
首先保证系统时区准确。

jdbc连接串中指定时区，并与数据库时区一致。

time_zone参数建议设置为'+8:00'，不使用容易误解的CST。

各环境数据库实例时区参数保持相同。
```



