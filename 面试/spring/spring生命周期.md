singletonObjects 容器 ====》 单例缓存池 ....

```
singletonsCurrentlyInCreation
```

```
 Mark the specified bean as already created (or about to be created).
	 * <p>This allows the bean factory to optimize its caching for repeated
	 * creation of the specified bean.
	 * @param beanName the name of the bean
alreadyCreated
```

```
	/** Map between dependent bean names: bean name to Set of dependent bean names. */

dependentBeanMap
```

```
	/** Map between depending bean names: bean name to Set of bean names for the bean's dependencies. */

dependenciesForBeanMap
```





第一个map是singletonObjects------------ 单例池  spring容器

第二个map是 singletonFactories 工厂

第三个是earlySingletonObjects [三级缓存](https://so.csdn.net/so/search?q=三级缓存&spm=1001.2101.3001.7020)





/** Cache of singleton objects: bean name to bean instance. */
	/** singletonObjects  一级缓存，可以理解为spring单利池 存放完整的bean*/
	
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
	
	/** Cache of singleton factories: bean name to ObjectFactory. */
	/** 这个二级缓存 是一个bean工厂，可以生产bean（循环依赖aop--通过cglib代理产生一个新的类，问题等可以在这里实现），
	如果二级缓存直接是一个bean 的话，很多需要对bean 的加工都不能实现，这里对aop的实现是通过一个后置处理器AnnotationAwareAspectJAutoProxyCreator */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
	
	/** Cache of early singleton objects: bean name to bean instance. */
	
	/** 三级缓存spring 为什么要三级缓存呢，
	this.earlySingletonObjects.put(beanName, singletonObject);
	this.singletonFactories.remove(beanName);
	从二级缓存singletonFactories remove，然后放入三级，因为有多个相同的依赖注入的话，
	防止singletonFactories 的重复创建，可以直接在earlySingletonObjects 中获取，提高性能。就好比你每次看病找老中医看了一个药方子，
	以后有这个病了就看药方子就行了，不用再重复去医院排队缴费，找老中医说你的症状，直接拿着药方子去买药吃就行了 */
	
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
————————————————
版权声明：本文为CSDN博主「这些不会的」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_25004825/article/details/104154746

一级缓存singletonObjects是线程安全的ConcurrentHashMap。

二级缓存是earlySingletonObjects，主要存放半成品的单例bean。

三级缓存singletonFactories核心是解决aop循环依赖。