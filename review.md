#### Spring:

    IOC(控制反转):
        在控制反转出来之前，组件之间的协调关系都是由程序内部代码控制的，这种方式会造成组件之间的互相耦合，IOC就是将组件间的关系从程序内部提到外部容器来管理;
        spring来负责控制对象的生命周期和对象之间的关系

    DI(依赖注入): 
        就是将服务注入到使用到它的地方，对象只提供普通的方法让容器去决定依赖关系;
        spring中DI的实现方式 setter 构造器
    
    bean生命周期:
        @Bean: 手动指定init方法和destory方法,生成bean顺序 无参构造方法-> init -> destory

        InitializingBean: afterPropertiesSet()等同于init
        DisposableBean: destroy()等同于destroy

        @PostConstruct:等同于init
        @PreDestroy:等同于destroy

        BeanPostProcessor(bean后置通知处理器):
            postProcessorBeforeInitialization:组件的初始化方法调用之前执行
            postProcessorAfterInitialization:组件的初始化方法调用之后执行
```java
        public class MyBeanPostProcessor implements BeanPostProcessor {
            @Override
            public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                System.out.println(beanName + " 初始化之前调用");
                return bean;
            }

            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                System.out.println(beanName + " 初始化之后调用");
                return bean;
            }
        }
```
        InstantiationAwareBeanPostProcessor: BeanPostProcessor的子类
            Initialization:初始化
                在spring bean生命周期中创建bean后，对其属性进行赋值，后置处理等操作
            Instantiation:实例化    
                实例化指的是创建bean过程
            该接口新增三个方法:
                postProcessBeforeInstantiation:组件的实例化方法调用之前执行
                postProcessAfterInstantiation:组件的实例化方法调用之后执行
```java
        @Component
        public class MyBeanInstantiationPostProcessor implements InstantiationAwareBeanPostProcessor {
            @Override
            public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
                if ("demoApplication".equals(beanName)) {
                    System.out.println("post process before " + beanName + " instantiation");
                }
                return null;
            }

            @Override
            public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
                if ("demoApplication".equals(beanName)) {
                    System.out.println("post process after " + beanName + " instantiation");
                }
                return true;
            }

            @Override
            public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
                if ("demoApplication".equals(beanName)) {
                    System.out.println("post process " + beanName + " properties");
                }
                return pvs;
            }
        }
```
        createBean -> doCreateBean -> populateBean -> initializeBean 
        [springbean生命周期]: (https://mrbird.cc/Spring-Bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.html)


        BeanFactoryPostProcessor:
            BeanFactory标准初始化之后，所有的Bean定义已经被加载，但Bean的实例还没被创建（不包括BeanFactoryPostProcessor类型）。该方法通常用于修改bean的定义，Bean的属性值等，甚至可以在此快速初始化Bean
            postProcessBeanFactory:
```java
    @Component
    public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
        public MyBeanFactoryPostProcessor(){
            System.out.println("实例化 MyBeanFactoryPostProcessor bean");
        }
        @Override
        public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
            int count = beanFactory.getBeanDefinitionCount();

            System.out.println("bean count：" + count);
        }

        @Component
        static class TestBean{
            public TestBean(){
                System.out.println("实例化TestBean");
            }

        }
    }

```

        BeanDefinitionRegistryPostProcessor:
            继承自BeanFactoryPostProcessor,新增了postProcessBeanDefinitionRegistry
                所有的Bean定义即将被加载，但Bean的实例还没被创建时。也就是说，BeanDefinitionRegistryPostProcessor的**postProcessBeanDefinitionRegistry**方法执行时机先于BeanFactoryPostProcessor的**postProcessBeanFactory**方法。这个方法通常用于给IOC容器添加额外的组件。
```java
    @Component
    public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
        @Override
        public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
            int count = registry.getBeanDefinitionCount();

            System.out.println("bean count11111 :" + count);
            RootBeanDefinition rbd = new RootBeanDefinition(Object.class);
            registry.registerBeanDefinition("testRbd", rbd);

        }

        @Override
        public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
            System.out.println("after bean count111111 :" + beanFactory.getBeanDefinitionCount());

        }
    }
```

    AOP:底层为动态代理,在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式,相应设计模式为代理模式
        四种通知方法:
            1.前置通知@Before
            2.后置通知@After
            3.返回通知@AfterReturning 
            4.异常通知@AfterThrowing
            执行顺序
            正常情况：@Before —-> 目标方法 —-> @AfterReturning —-> @After
            异常情况：@Before —-> 目标方法 —-> @AfterThrowing —-> @After
    [aop源码分析]: (https://mrbird.cc/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Spring-AOP%E5%8E%9F%E7%90%86.html)
```java
        @Override
        public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
            if (!IN_NATIVE_IMAGE &&
                    (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config))) {
                Class<?> targetClass = config.getTargetClass();
                if (targetClass == null) {
                    throw new AopConfigException("TargetSource cannot determine target class: " +
                            "Either an interface or a target is required for proxy creation.");
                }
                if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
                    return new JdkDynamicAopProxy(config);
                }
                return new ObjenesisCglibAopProxy(config);
            }
            else {
                return new JdkDynamicAopProxy(config);
            }
        }
```

    spring循环依赖: spring使用提前曝光机制，利用三级缓存解决循环依赖问题
        创建bean过程:
            doGetBean方法中先通过getSingleton(String beanName)方法从三级缓存中获取Bean实例，如果不为空则进行后续处理；
            如果为空，则通过getSingleton(String beanName, ObjectFactory<?> singletonFactory)方法创建Bean实例并进行后续处理。
        总结:
            上面的例子都是基于属性注入的情况，假如存在构造器注入情况下的循环依赖，Spring将没办法解决。这是因为对象的提前曝光时机发生在对象实例化之后，而构造器注入时机为对象实例化时，所以此时还未进行提前曝光操作，循环依赖也就没办法解决了，比如下面这种情况：
```java
    @SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

@Component
class BeanA {

    private BeanB beanB;

    public BeanA(BeanB beanB) {
        this.beanB = beanB;
    }

    public BeanB getBeanB() {
        return beanB;
    }

    public void setBeanB(BeanB beanB) {
        this.beanB = beanB;
    }
}

@Component
class BeanB {

    private BeanA beanA;

    public BeanB(BeanA beanA) {
        this.beanA = beanA;
    }

    public BeanA getBeanA() {
        return beanA;
    }

    public void setBeanA(BeanA beanA) {
        this.beanA = beanA;
    }
}

```
        
```java
    public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

        /** Cache of singleton objects: bean name to bean instance. 
        * 一级缓存，key为Bean名称，value为Bean实例。这里的Bean实例指的是已经完全创建好的，即已经经历实例化->属性填充->初始化以及各种后置处理过程的Bean，可直接使 用。
        */
        private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

        /** Cache of singleton factories: bean name to ObjectFactory. 
        * 三级缓存，key为Bean名称，value为Bean工厂。在Bean实例化后，属性填充之前，如果允许提前曝光，Spring会把该Bean转换成Bean工厂并加入到三级缓存。在需要引用提前曝光对象时再通过工厂对象的getObject()方法获取。
        */
        private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

        /** Cache of early singleton objects: bean name to bean instance. 
        * 二级缓存，key为Bean名称，value为Bean实例。这里的Bean实例指的是仅完成实例化的Bean，还未进行属性填充等后续操作。用于提前曝光，供别的Bean引用，解决循环依赖。
        */
        private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);

        ......

    }
    // 从3级缓存获取bean实例
    @Nullable
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        // 从一级缓存中获取目标Bean实例
        Object singletonObject = this.singletonObjects.get(beanName);
        // 如果从一级缓存中没有获取到，并且该Bean处于正在创建中的状态时
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            // 从二级缓存获取目标Bean实例
            singletonObject = this.earlySingletonObjects.get(beanName);
            // 如果没有获取到，并且允许提前曝光的话
            if (singletonObject == null && allowEarlyReference) {
                synchronized (this.singletonObjects) {
                    // 在锁内重新从一级缓存中往下查找
                    singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        singletonObject = this.earlySingletonObjects.get(beanName);
                        if (singletonObject == null) {
                            // 从三级缓存中取出目标Bean工厂对象
                            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                            if (singletonFactory != null) {
                                // 工厂对象不为空，则通过调用getObject方法实例化Bean实例
                                singletonObject = singletonFactory.getObject();
                                // 放到二级缓存中
                                this.earlySingletonObjects.put(beanName, singletonObject);
                                // 删除对应的三级缓存
                                this.singletonFactories.remove(beanName);
                            }
                        }
                    }
                }
            }
        }
        return singletonObject;
    }

    // 如果通过三级缓存的查找都没有找到目标Bean实例，则通过getSingleton(String beanName, ObjectFactory<?> singletonFactory)方法创建：
    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        synchronized (this.singletonObjects) {
            // 从一级缓存获取
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // 为空则继续
                ......
                // 方法内会将当前Bean名称添加到正在创建Bean的集合（singletonsCurrentlyInCreation）中
                beforeSingletonCreation(beanName);
                boolean newSingleton = false;
                ......
                try {
                    // 通过函数式接口创建Bean实例，该实例已经经历实例化->属性填充->初始化以及各种后置处理过程，可直接使用
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                }
                catch (IllegalStateException ex) {
                   ......
                }
                finally {
                    ......
                }
                if (newSingleton) {
                    // 添加到缓存中
                    addSingleton(beanName, singletonObject);
                }
            }
            return singletonObject;
        }
    }

    protected void addSingleton(String beanName, Object singletonObject) {
        synchronized (this.singletonObjects) {
            // 添加到一级缓存
            this.singletonObjects.put(beanName, singletonObject);
            // 删除对应的二三级缓存
            this.singletonFactories.remove(beanName);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }

```

    事务:
        传播行为:
            PROPAGATION_MANDATORY: 表示该方法必须在事务中进行，如果当前事务不存在，则会 抛出一个异常
            PROPAGATION_NESTED:	如果当前已存在一个事务，那么该方法在嵌套事务中运行。 嵌套事务可以独立于当前事务进行单独地提交或回滚。如 果当前事务不存在，那么行为和 PROPAGATION_REQUIRED一样
            PROPAGATION_NEVER: 表示当前方法不运行在事务上下文中。如果当前正有一个 事务在运行，则会抛出异常
            PROPAGATION_NOT_SUPPORTED: 表示该方法不应该运行在事务上下文中，如果存在当前事 务，在该方法运行期间，当前事务会被挂起
            PROPAGATION_REQUIRED: 表示当前方法必须运行在事务中，如果事务不存在，则启 动一个新的事务
            PROPAGATION_REQUIRED_NEW: 表示当前方法必须运行在它自己的事务中。一个新的事务 将启动。如果存在当前事务，当前事务会被挂起
            PROPAGATION_SUPPORTS: 表示当前方法不需要事务上下文，但是如果存在当前事务 的话，那么该方法会在这个事务中运行  
        
        隔离级别:
            脏读（Dirty reads）: 发生在一个事务读取了另一个事务改写后但未提交的数据，如果改写被回滚了，那么第一个事务获取的数据就是“脏”的。
            不可重复读（Nonrepeatable read）: 一个事务执行两次以上相同查询得到不同的数据。这通常是另外一个事务在此期间更新了数据。
            幻读（Phantom read）: 一个事务读取了几行数据，另一个事务插入了几条数据，当第一个事务再次读取时发现多了几条原本没有的数据。

            隔离级别	            含义
            ISOLATION_DEFAULT :	使用后端数据库默认的隔离级别
            ISOLATION_READ_UNCOMMITTED	: 允许读取尚未提交的数据表更。可能导致脏读，不可重复 读，幻读
            ISOLATION_READ_COMMITTED   :	允许读取并发事务已经提交的数据，可以阻止脏读，但不 可重复读，幻读仍可能发生
            ISOLATION_REPEATABLE_READ  :	对同一字段的多次读取结果一致，除非数据是本事务自己 修改的。可以阻止脏读，不可重复读，但仍可能发生幻读
            REPEATABLE_SERIALIZABLE :	完全服从事务的ACID原则，避免脏读，不可重复读，幻读



#### 算法

##### 二叉树
    前中后序遍历:
```java
    // 前序遍历 method1
    public List<Integer> preOrder(TreeNode root){
        List<Integer> result = new ArrayList<>();
        if (root == null){
            return ressult;
        }

        result.add(root.value);
        result.addAll(preOrder(root.left));
        result.addAll(preOrder(root.right));
    }

    // method2
    List<Integer> result = new ArrayList<>();

    public List<Integer> preOrder(TreeNode root){
        recursion(root);
        return result;
    }

    void recursion(TreeNode root){
        if (root == null){
            return;
        }

        result.add(root.value);
        recursion(root.left);
        recursion(root.right);
    }






    // ****************************************************************
    // 中序遍历 method1
    public List<Integer> preOrder(TreeNode root){
        List<Integer> result = new ArrayList<>();
        if (root == null){
            return ressult;
        }

        result.add(root.value);
        result.addAll(preOrder(root.left));
        result.addAll(preOrder(root.right));
    }

    // method2
    List<Integer> result = new ArrayList<>();

    public List<Integer> middleOrder(TreeNode root){
        recursion(root);
        return result;
    }

    void recursion(TreeNode root){
        if (root == null){
            return;
        }

        recursion(root.left);
        result.add(root.value);

        recursion(root.right);
    }






    // ****************************************************************
    // 后序遍历 method1
    public List<Integer> postOrder(TreeNode root){
        List<Integer> result = new ArrayList<>();
        if (root == null){
            return ressult;
        }

        result.addAll(postOrder(root.left));
        result.addAll(postOrder(root.right));
        result.add(root.value);

    }

    // method2
    List<Integer> result = new ArrayList<>();

    public List<Integer> postOrder(TreeNode root){
        recursion(root);
        return result;
    }

    void recursion(TreeNode root){
        if (root == null){
            return;
        }

        result.add(root.value);
        recursion(root.left);
        recursion(root.right);
    }


    // 层级遍历
    void levelTraverse(TreeNode root){
        if (root == null){
            return ;
        }

        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);

        while (!q.isEmpty()){
            int size = q.size();

            for (int i = 0; i < size ; i ++){
                TreeNode t = q.poll();
                if (t.left != null){
                    q.offer(t.left);
                }
                if (t.right != null){
                    q.offer(t.right);
                }
                
            }
        }
    }
```