---
title: Springboot+Mybatis基本姿势
date: 2021-03-23 14:02:13
tags: 
- 后端
- Java
- 入门系列
category: 后端
---

# Springboot + Mybatis 基本姿势



最近做了个CS5224 Cloud Computing的课程项目，由于前端打算做个Android App，后端为了统一也就选用了Java开发。

项目团队的地址在 https://github.com/nus-cs5224-team/；

极速入门了一波 Springboot，Mybatis，复习了一波 SQL，感觉还是开发比较快乐；



## 基本框架目录结构

```
main                                             
  ├─java                                          
  │  └─edu                                        
  │      └─nus                                    
  │          └─campus                             
  │              │  CampusApplication.java
  │              ├─config                         
  │              │      SwaggerConfig.java        
  │              ├─controller                     
  │              │      BuildingController.java   
  │              │      ...                      
  │              ├─mappers                        
  │              │      BuildingMapper.java       
  │              │      ...                           
  │              └─model                          
  │                      Building.java            
  │                      ...                                    
  └─resources                                     
      │  application.properties                        
      ├─mybatis                                   
      │  │  mybatis-config.xml                    
      │  └─mappers                                
      │          BuildingMapper.xml               
      │          ...                 
      └─sql                                       
              fake_data.sql                       
              initial.sql                         
```



从上到下说，

`CampusApplication.java` 是项目文件的主入口；

`config`文件夹里放的是`Swagger`的配置，不用`Swagger`生成文档的话就不需要这个文件夹了；

`controller`是路由，通过Spring提供的注解，接受`http`请求，进行数据库查询和一些逻辑操作，然后返回结果；

`mappers`是mybatis的dao层，这里写的都是`interface`类型，mybatis会自动从`resources/mybatis/mappers/*.xml`描述的SQL里自动生成方法填充这个类；

`model`是我们的实体类，比如 `User`，比如`Building`，一般和数据库里的table对应，可以从数据库table的一行内容生成一个JavaBean实例；



然后到了 `resources`，主要就是 `mybatis/mappers`里描述的SQL，`Mybatis`可以自动生成一些代码；

然后放了几个SQL脚本，用来生成table，以及写一些假数据填充；



> 我一般的主要困惑就是项目结构如何组织，因为具体的操作方式在官方文档里通常讲的都比较详细，所以把这一节放在了第一部分，首先要搞清楚什么东西该写在哪里~



## Springboot

### API 接口

用`@RestController`注解，非常方便可以映射到接口上，函数返回的结果可以自动转换成Json；

```Java
@RestController
@RequestMapping("/api/v1/bus")
@Api(tags = "Bus API")
public class BusController {
    @ApiOperation("Get a bus by name")
    @GetMapping("/{busName}")
    public Bus getBus(@PathVariable("busName") String busName) {
        // CRUD from db, get the bus
        return ...
    }
    ...
}
```

大概就是这样，上边的那个 `@Api` 注解是Swagger的一部分，可以用来自动生成API文档；

其他的`@PostMapping`之类的，直接看文档就好了，一找一大把；

这其实就是主要用到的Springboot的功能了；



### Swagger(springfox)

可以自动生成文档，非常好用；

```java
@RestController
@RequestMapping("/api/v1/bus")
@Api(tags = "Bus API")
public class BusController {
    @Autowired
    private BusMapper busMapper;

    @ApiOperation("Get a bus by name")
    @GetMapping("/{busName}")
    public Bus getBus(@PathVariable("busName") String busName) {
        // CRUD from db, get the bus
        if (busName == null) return null;
        return busMapper.findByName(busName);
    }
    ...
}
```

其中的 `@Api` 注解，生成效果就是将以下的所有api归为一个类别显示在文档里；

其中的 `@ApiOperation("Get a bus by name")`注解，就是表示具体某个api的作用；

swagger可以自动提取每个函数的参数，并将其转换成http请求的语言，比如上边的`@PathVariable`，或者用`@RequestParam(, require=true|false)`，也可以体现到最终的文档里；



## Mybatis

Mybatis本质上就是个用xml文档来写sql的工具，可以将xml文档翻译成一个Java类，这个类提供了xml里定义的增删改查等等等等功能；

Mybatis强大的地方在于他提供了一系列类似于`resultMap`的元素，可以将sql的结果按照一定规则映射到`resultMap`，然后映射到一个Java对象上；



> `resultMap`的问题在于，每个map都要对应一段java代码定义的实体类，会导致代码部分比较臃肿，所以对于我的小项目，我宁可在函数里跑两次SQL查询，而不是为每类查询都定义一个`resultMap`；这个主要还是看需求的，如果需求上去了，能一个SQL跑完的是一定不运行2次SQL的，毕竟性能第一嘛；



#### xml 实例

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="edu.nus.campus.mappers.StopMapper">
    <select id="findById" resultType="edu.nus.campus.model.Stop">
        SELECT * FROM `stop` WHERE `id`=#{id};
    </select>

    <select id="findRunningThroughBus" parameterType="edu.nus.campus.model.Stop" resultType="edu.nus.campus.model.Bus">
        SELECT * FROM `bus`
        WHERE `id` in (SELECT `bus_id` FROM `running_through`
                       WHERE `stop_id`=#{id});
    </select>
</mapper>
```

第一句的namespace，表示这个Mapper会实现到一个叫`StopMapper`的类，其中类的方法就是xml里元素的id定义的名字；

可以看到可以用`<select>`元素表示一个`select`语句，返回结果可以定义为一个`JavaBean`类，会自动生成这样的一个实体对象。

具体的操作还有`<insert>`等等，`resultType`可以替换成`resultMap`，然后使用`associate`等属性提供连接的查询，会更优雅。



#### 对应的 Mapper interface

```jav
public interface StopMapper {
    Stop findById(int id);
    List<Bus> findRunningThroughBus(Stop stop);
}
```

可以看到返回值的类型，直接就是Java类型了；



### Config

配置在`application.properties`文件里；

```
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mappers/*.xml
```

表示，使用`/resource/mybatis/mybatis-config.xml`代表的mybatis配置，并映射所有位于`/resource/mybatis/mappers/*.xml`的mapper。

这里的配置有很多，可以选择缓存的时效等等，我的项目没有特别的需求，也就没有调整这里的内容；

