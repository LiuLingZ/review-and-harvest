#  

​	https://juejin.im/post/5cfa6e465188254ee433bc69

# Mybatis-Plus使用



## 一、过程

### 1、编写Entity

```java
/**
 * @Description 学生信息实体类
 * @Author Sans
 * @CreateTime 2019/5/26 21:41
 */
@Data
@TableName("user_info")//@TableName中的值对应着表名
public class UserInfoEntity {

    /**
     * 主键
     * @TableId中可以决定主键的类型,不写会采取默认值,默认值可以在yml中配置
     * AUTO: 数据库ID自增
     * INPUT: 用户输入ID
     * ID_WORKER: 全局唯一ID，Long类型的主键
     * ID_WORKER_STR: 字符串全局唯一ID
     * UUID: 全局唯一ID，UUID类型的主键
     * NONE: 该类型为未设置主键类型
     */
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
}

```



## 2、编写Dao，需要继承 BaseMapper

```java
/*
	以实体类为类型
*/
public interface UserInfoDao extends BaseMapper<UserInfoEntity> {
}
```



### 3、编写Service(接口),需要继承 IService<>接口

```java
/*
	需要继承IService接口，并将Entity作为类型
*/
public interface UserInfoService extends IService<UserInfoEntity>{}

```



### 4、编写Service接口实现类 ,需要额外继承 ServiceImpl<>类

```java

/**
	ServiceImple<继承了BaseBaseMapper的dao，entity>
*/
@Service
@Transactional
public class UserInfoService extends ServiceImpl<UserInfoDao,UserInfoEntity> implements UserInfoService{
	    
}
```





## 二、各种操作

​	`UserInfoService` 是 继承了IService的接口，假设以此为例 ：

```java
//根据Id获取用户信息 getById
UserInfoEntity userInfoEntity = userInfoService.getById(userId);

//查看所有用户信息 list
List<UserInfoEntity> list = userInfoService.list() ;

/** 
*	分页查询所有数据
*/
IPage<UserInfoEntity> page = new Page<>();
page.setCurrent(5) ; //当前页
page.setSize(10); //每页条数
page = userInfoService.page(page);
return page ;


//新增
userInfoService.save(obj);

//批量新增
List<obj> list
userInfoService.save(list)


// !!!!!!!!!! 自己编写SQL查询时也可以同时做到分页效果。
https://juejin.im/post/5cfa6e465188254ee433bc69   看最下面的例子。



```





















