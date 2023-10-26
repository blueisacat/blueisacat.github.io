### parentName

bean定义对象的父类定义对象名称

### beanClassName

bean对象的实际class类

### scope

bean对象是否为单例

### lazyInit

是否懒加载

### dependsOn

设置依赖的bean对象，被依赖的bean对象总是会比当前bean对象先创建

### autowireCandidate

设置是否可以自动注入。只对@Autowired注解有效，配置文件中可以通过property显示注入

### primary

配置bean为主要候选bean。当同一个接口的多个实现类或者一个类多次注入到spring容器时，通过该属性来配置某个bean为主候选bean，通过类型来注入时，默认为使用主候选bean注入

### factoryBeanName

设置创建bean的工厂名称

### factoryMethodName

设置创建bean的工厂中，创建bean的具体方法

### initMethodName

设置创建bean时，默认初始化的方法

### destroyMethodName

设置销毁bean时调用的方法名称。注意需要调用context的close()方法才会调用

### role

设置bean的分类。APPLICATION:用户INFRASTRUCTURE:完全内部使用，与用户无关SUPPORT:某些复杂配置的一部分

### description

对bean对象的描述

### ConstructorArgumentValues

记录构造函数注入属性，通过bean的水属性constructor-arg来注入

### MutablePropertyValues

普通属性集合