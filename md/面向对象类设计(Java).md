# 面向对象类设计(Java)
## 对象概念及约束原则
- 封装
- 继承
- 多态
##### 内在联系
 - 封装（静态）+继承（动态）实现多态
## 表现形式
- 普通类
- 抽象类
- 接口
## 体现方式
- 封装-普通类
- 继承-抽象类、接口类
- 实现：多态
## 类设计原则
##### 单一职责原则
- 注意职责扩散，前期的职责后期分化为多个子职责
##### 开-闭原则
- 对未来可能变动的点抽象，实现增加功能而不是改变功能
##### 里氏原则
- 开-闭原则的一种体现
##### 接口分离原则
- 防止过度细化导致的接口数量爆炸
##### 依赖倒置原则
- 多态和面向接口编程的重要体现点，用继承时注意遵从里氏原则
##### 迪米特原则
- 高内聚低耦合的重要体现，类只与直接朋友（成员变量、入参、返参）耦合，局部变量出现非直接朋友时尝试一下直接朋友关联
##### 组合/聚合复用原则
- ‘has-a’和‘is-a’的区分体现点，利于责任委派和减少对父类的依赖，注意责任划分过度容易产生很多个类
#### 以上原则是针对某个侧重方向而不是按正交维度描述的，有交集的地方。
## 一句话描述
#### 专注于要做什么而不是怎么做；尽量组合而不是继承
## 示例
在spring的BeanFactory实现里，为了让工厂可配置，分离了接口ConfigurableBeanFactory，此接口继承接口HierarchicalBeanFactory，后者目的是可访问父工厂和本地bean，HierarchicalBeanFactory继承的根工厂接口BeanFactory，这个接口里只涉及到了bean的相关基本操作；在对工厂中bean的相关信息获取上，单独分离了接口ListableBeanFactory，此接口直接继承BeanFactory，体现了接口分离原则
```sh
org.springframework.beans.factory.BeanFactory
org.springframework.beans.factory.HierarchicalBeanFactory
org.springframework.beans.factory.config.ConfigurableBeanFactory
org.springframework.beans.factory.config.ListableBeanFactory
```
在Sring框架中，依赖注入是实现依赖倒置原则的一种方法
公司的Artery框架中ArteryUtil获取的getBean()方法，内部调用的ArterySpringUtil的getBean()方法，后者通过实现Spring的ApplicationContextAware接口持有一个ApplicationContext实例成员，真正获取bean是调用的context的getBean()方法，其实这已经有两层代理了，在此util包中，各个util划分开的，比如ArteryBrowserUtil、ArteryCacheUtil、ArteryDateUtil等等，体现了单一职责原则
```sh
com.thunisoft.artery.util.ArterySpringUtil
com.thunisoft.artery.util.ArteryBrowserUtil
com.thunisoft.artery.util.ArteryCacheUtil
com.thunisoft.artery.util.ArteryDateUtil
```