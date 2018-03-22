---
title: retrofit和动态代理
date: 2017-02-07 09:45:11
tags: 
- Android
categories: notes
password: 123456
---

简述动态代理在Retrofit的应用

> 动态代理

代理类在运行前不存在，运行时由程序动态生成的代理方式称为动态代理

<!--more-->

#### 关于Retrofit

Square公司的OkHttp简直是完美的一个网络请求库，而在其上又封装了一层的Retrofit库，使其调用Restful Api更方便

#### 简述Retrofit调用流程

因为本篇只是简述动态代理在Retrofit的使用，所以不过多总结Retrofit的详细使用，在上家公司的项目都网络层，用的retrofit，现在这家公司的项目因为刚开始是我独立开发，所以网络层框架我也是用的Retrofit，功能强大，详细使用，以后再总结

熟悉Retrofit使用的都对下面几个步骤比较熟悉了：

- 定义一个ApiService接口，通过注解可以标记请求方法，请求参数，以及添加的头信息
- 然后创建Retrofit对象，通过建造者模式设置BaseUrl等一些参数，通过Retrofit对象create一个你定义的接口对象
- 拿到接口对象调用具体的方法完成请求

具体的代码大概如下：

```java
public interface ApiService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .build();
ApiService service = retrofit.create(ApiService.class);
```

```java
Call<List<Repo>> repos = service.listRepos("fengan");
```

#### ApiService是如何产生的

因为接口是不可以直接new出来的，那么ApiService是如何产生的呢？

ApiService service = retrofit.create(ApiService.class);方法内部到底做了什么？

没错，就是动态代理

为了更好的理解动态代理，下面过一下简易版的Retrofit，

```j a v
public interface Callback<T> {

    void onSuccess(Object t);

    void onFailed(Exception e);

}
```

```java
    /**
     * 约定最后一个参数是callback
     */
public interface GithubService {

    @GET("users/{user}/repos")

    void list<Repos>(@Path("user") String user,Callback<List<Repo>> callback);
}
```

用到了两个注解，一个是方法注解，一个是参数注解

```java
@Retention(RetentionPolicy.RUNTIME)

@Target({ElementType.METHOD})

public @interface GET {

    String value() default "";

}
```

```java
@Retention(RetentionPolicy.RUNTIME)

@Target(ElementType.PARAMETER)

public @interface Path {

    String value();

}
```

Repo实体类是使用GsonFormat根据json自动生成的。现实使用中，我们在构建Retrofit过程传入GsonFactory

Retrofit这个类应该是一个builder模式，里面可以设置baseUrl，姑且忽略其他所有参数。还有一个create方法

```java
public class Retrofit {

    private String baseUrl;
    private Retrofit(Builder builder) {
        this.baseUrl = builder.baseUrl;
    }

    public <T> T create(Class<T> clazz) {
        return null
    }

    static class Builder {
        private String baseUrl;
        Builder baseUrl(String host) {
            this.baseUrl = host;
            return this;
        }

        Retrofit build() {
            return new Retrofit(this);
        }
    }
}
```

最关键的就是create这个方法了，

```java
public <T> T create(Class<T> clazz) {
        /**
         * 缓存中去
         */
        Object o = serviceMap.get(clazz);
        /**
         * 取不到则取构造代理对象
         */
        if (o == null) {
            o = (T) Proxy.newProxyInstance(Retrofit.class.getClassLoader(), new Class[]{clazz}, new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    final Callback<?> callback = (Callback<?>) args[args.length - 1];


                    final GET get = method.getAnnotation(GET.class);
                    if (get != null) {
                        /**
                         * 获得GET注解的值
                         */
                        String getValue = get.value();

                        System.out.println(getValue);

                        /**
                         * 获得所有参数上的注解
                         */
                        Annotation[][] methodParameterAnnotationArrays = method.getParameterAnnotations();

                        if (methodParameterAnnotationArrays != null) {
                            int count = methodParameterAnnotationArrays.length;
                            for (int i = 0; i < count; i++) {
                                /**
                                 * 获得单个参数上的注解
                                 */
                                Annotation[] methodParameterAnnotations = methodParameterAnnotationArrays[i];

                                if (methodParameterAnnotations != null) {
                                    for (Annotation methodParameterAnnotation : methodParameterAnnotations) {

                                        /**
                                         * 如果是Path注解
                                         */
                                        if (methodParameterAnnotation instanceof Path) {

                                            /**
                                             * 取得path注解上的值
                                             */
                                            Path path = (Path) methodParameterAnnotation;
                                            String pathValue = path.value();
                                            System.out.println(pathValue);

                                            /**
                                             * 这是对应的参数的值
                                             */
                                            System.out.println(args[i]);


                                            Request.Builder builder = new Request.Builder();


                                            /**
                                             * 使用path注解替换get注解中的值为参数值
                                             */
                                            String result = getValue.replaceAll("\\{" + pathValue + "\\}", (String) args[i]);

                                            System.out.println(result);

                                            /**
                                             * 开始构造请求
                                             */
                                            Request request = builder.get()
                                                    .url(baseUrl + "/" + result)
                                                    .build();

                                            okHttpClient.newCall(request).enqueue(new okhttp3.Callback() {
                                                @Override
                                                public void onFailure(Call call, IOException e) {
                                                    /**
                                                     * 失败则回调失败的方法
                                                     */
                                                    callback.onFailed(e);
                                                }

                                                @Override
                                                public void onResponse(Call call, Response response) throws IOException {
                                                    if (response.isSuccessful()) {
                                                        /**
                                                         * 请求成功
                                                         */
                                                        String body = response.body().string();

                                                        /**
                                                         * 使用fastjson进行zhuan转换
                                                         */
                                                        Type type = callback.getClass().getGenericInterfaces()[0];

                                                        Object o1 = JSON.parse(body);

                                                        /**
                                                         * 回调成功
                                                         */
                                                        callback.onSuccess(o1);
                                                    }
                                                }
                                            });

                                        }
                                    }
                                }

                            }
                        }
                    }


                    return null;
                }
            });
            /**
             * 扔到缓存中
             */
            serviceMap.put(clazz, o);
        }
        return (T) o;
    }
```

然后我们就可以根据Retrofit那样进行调用了

```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.github.com")
        .build();

GithubService githubService = retrofit.create(GithubService.class);

githubService.listRepos("lizhangqu", new Callback<List<Repo>>() {
    @Override
    public void onSuccess(Object t) {
        System.out.println(t);
    }
    @Override
    public void onFailed(Exception e) {
    }
});
```

#### retrofit动态代理原理

原理就是先拿到最后一个参数，也就是回调，再拿到方法上的注解，获得具体的值，然后拿到除了回调之外的其他参数，获得参数上的注解，然后根据注解取得对应的值，还有原来的参数值，将方法上的注解的值中进行替换。使用OkHttp构造请求，请求完成后根据将结果解析为回调中的类型。整个过程如下

拦截到方法、参数，再根据我们在方法上的注解，去拼接为一个正常的Okhttp请求，然后执行。

#### java中的动态代理

在java的动态代理机制中，有两个重要的类或接口，一个是 InvocationHandler(Interface)、另一个则是 Proxy(Class)，这一个类和接口是实现我们动态代理所必须用到的

每一个动态代理类都必须要实现InvocationHandler这个接口，并且每个代理类的实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。我们来看看InvocationHandler这个接口的唯一一个方法 invoke 方法：

```java
Object invoke(Object proxy, Method method, Object[] args) throws Throwable

proxy:　　指代我们所代理的那个真实对象
method:　　指代的是我们所要调用真实对象的某个方法的Method对象
args:　　指代的是调用真实对象某个方法时接受的参数
```

Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler h)  throws IllegalArgumentException
```

这个方法的作用就是得到一个动态的代理对象，其接收三个参数，我们来看看这三个参数所代表的含义：

```java

public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException

loader:　　一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载

interfaces:　　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了

h:　　一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上
```

使用如下：

- 定义被代理对象

```java
public interface Subject
{
    public void rent();
    
    public void hello(String str);
}
```

```java
public class RealSubject implements Subject
{
    @Override
    public void rent()
    {
        System.out.println("I want to rent my house");
    }
    
    @Override
    public void hello(String str)
    {
        System.out.println("hello: " + str);
    }
}
```

- 定义动态代理类，任何动态代理类都必须实现InvotionHandler这个接口，重写invoke方法

```java
public class DynamicProxy implements InvocationHandler
{
    //　这个就是我们要代理的真实对象
    private Object subject;
    
    //    构造方法，给我们要代理的真实对象赋初值
    public DynamicProxy(Object subject)
    {
        this.subject = subject;
    }
    
    @Override
    public Object invoke(Object object, Method method, Object[] args)
            throws Throwable
    {
        //　　在代理真实对象前我们可以添加一些自己的操作
        System.out.println("before rent house");
        
        System.out.println("Method:" + method);
        
        //    当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
        method.invoke(subject, args);
        
        //　　在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("after rent house");
        
        return null;
    }

}
```

- 使用

```Java
public class Client
{
    public static void main(String[] args)
    {
        //    我们要代理的真实对象
        Subject realSubject = new RealSubject();

        //    我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
        InvocationHandler handler = new DynamicProxy(realSubject);

        /*
         * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
         * 第一个参数 handler.getClass().getClassLoader() ，我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
         * 第二个参数realSubject.getClass().getInterfaces()，我们这里为代理对象提供的接口是真实对象所实行的接口，表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
         * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 InvocationHandler 这个对象上
         */
        Subject subject = (Subject)Proxy.newProxyInstance(handler.getClass().getClassLoader(), realSubject
                .getClass().getInterfaces(), handler);
        
        System.out.println(subject.getClass().getName());
        subject.rent();
        subject.hello("world");
    }
}
```