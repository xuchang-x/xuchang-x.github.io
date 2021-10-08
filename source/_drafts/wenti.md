-  xmlns:crane="http://code.dianping.com/schema/crane"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://code.dianping.com/schema/crane http://code.dianping.com/schema/crane/crane-1.0.xsd"
          这些命名空间
- 用lsof -i:端口号命令行，以80为例的，查看端口是否被占用
- java8 reduce 和各种函数
```java
List<Integer> boardIdList = boardMap.values().stream()
                .filter(list -> null != list && CollectionUtil.isNotEmpty(list))
                .flatMap(Collection::stream)
                .map(Board::getBoardId)
                .collect(Collectors.toList());
```
list里不会有null么，还是map.values()会去掉null，看总提示让去掉null != list这段代码

-- interface和enum不用static（变灰）

微服务10要素
微服务的10要素：API、服务调用、服务发现；日志、链路追踪；容灾性、监控、扩缩容；发布管道；鉴权。