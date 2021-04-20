```java
public static <T> T json2obj(String jsonStr, Type targetType) {
        try {
            ObjectMapper mapper = new ObjectMapper();
            //忽略多余的字段
            mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
            //忽略空的字段
            mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
            JavaType javaType = TypeFactory.defaultInstance().constructType(targetType);

            return mapper.readValue(jsonStr, javaType);
            // 转成带泛型的
            // return mapper.readValue(jsonStr, new TypeReference<R>() {});
        } catch (IOException e) {
            e.printStackTrace();
            throw new IllegalArgumentException("将JSON转换为对象时发生错误:" + jsonStr, e);
        }
    }
```