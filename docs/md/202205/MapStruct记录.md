# MapStruct记录

#### 导包问题

如果自动生成的实现类需要导包，就要在类上的Mapper注解上加上import， 例如：

```java
@Mapper(imports = {JsonUtil.class, Map.class, Integer.class})
public interface XxxConverter {
  
  @Mappings({@Mapping(target = "Xxxx",expression = "java(JsonUtil.objectToJson(data.getYyy()))")})
  DO toDO(PO po);
}
```

JsonUtil 是自己写的工具类，或者从其他地方引入的工具类，在这里要imports进去，不然报错；