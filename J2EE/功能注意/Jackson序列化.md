# Jackson序列化



## 一、Jackson特点

- 容易使用 - jackson API 提供了一个高层次外观，以简化常用的用例。
- 无需创建映射 - API提供了默认的映射大部分对象序列化。
- 性能高 - 快速，低内存占用，适合大型对象图表或系统。
- 干净的 JSON - jackson 创建一个干净和紧凑的 JSON 结果，这是让人很容易阅读。
- 不依赖 - 库不需要任何其他的库，除了 JDK。
- 开源代码 - jackson 是开源的，可以免费使用。





## 二、Jackson注解

​	Jackson的注解可以快速建立 Java 类与 JSON 之间的关系。

#### @JsonProperty

​	@JsonProperty 注解指定一个属性用于 JSON 映射，默认情况下映射的 JSON 属性与注解的属性名称相同，不

​	过可以使用该注解的 value 值修改 JSON 属性名，该注解还有一个 `index` 属性指定生成 JSON 属性的顺序，

​	如果有必要的话。

#### @JsonIgnore

​	@JsonIgnore 注解用于排除某个属性，这样该属性就不会被 Jackson 序列化和反序列化。

##### @JsonIgnoreProperties

​	@JsonIgnoreProperties 注解是类注解。在序列化为 JSON 的时候，@JsonIgnoreProperties({"prop1", 

​		"prop2"}) 会忽略 pro1 和 pro2 两个属性。在从 JSON 反序列化为 Java 类的时候，	

​	@JsonIgnoreProperties(ignoreUnknown=true) 会忽略所有没有 Getter 和 Setter 的属性。该注解在 Java 类

​		和 JSON 不完全匹配的时候很有用。

#### @JsonIgnoreType

​	@JsonIgnoreType 也是类注解，会排除所有指定类型的属性。

####@JsonPropertyOrder

​	@JsonPropertyOrder 和 @JsonProperty 的 index 属性类似，指定属性序列化时的顺序。

####@JsonRootName

​	@JsonRootName 注解用于指定 JSON 根属性的名称。





## 三、例子



```java
/**
 * 
 *	序列化\反序列化类 ObjectMapper:
 *		- readValue()
 *		- writeValueAsString() 序列化为字符串形式
 */
public class JsonTester {
    public static void main(String[] args) {
        // 创建 ObjectMapper 对象
        ObjectMapper mapper = new ObjectMapper();
        String jsonString = "{\"name\":\"Mahesh\", \"age\":21}";

        try {
            // 反序列化 JSON 到对象
            Student student = mapper.readValue(jsonString, Student.class);
            System.out.println(student);

            // 序列化对象到 JSON
            String json = mapper.writeValueAsString(student);
            System.out.println(json);
        } catch (JsonParseException e) {
            e.printStackTrace();
        } catch (JsonMappingException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

class Student {
    private String name;
    private int age;

    public Student() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String toString() {
        return "Student [ name: " + name + ", age: " + age + " ]";
    }
}
```

