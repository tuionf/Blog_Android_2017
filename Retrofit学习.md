### Android   Retrofit入门学习使用

## 是什么？

一个基于 OkHttp 的 RESTful API 请求工具



Retrofit 在使用时其实就充当了一个**适配器**（Adapter）的角色，主要是将一个 Java 接口翻译成一个 HTTP 请求对象，然后用 OkHttp 去发送这个请求



**核心思想**：动态代理—通俗来讲，就是你要执行某个操作的前后需要增加一些操作，比如查看用户个人信息前需要判断用户是否登录，用户访问数据库后想清除用户的访问记录等操作





Retrofit 主要定义了 4 个接口：

- Callback<T>：请求数据的返回；
- Converter<F, T>：对返回数据进行解析，一般用 GSON ；
- Call<T>：发送请求，Retrofit 默认的实现是 OkHttpCall<T>，也可以依需自定义 Call<T>；
- CallAdapter<T>：将 Call 对象转换成其他对象，如转换成支持 RxJava 的 Observable对象



![](https://ws1.sinaimg.cn/large/534fc2d6ly1fiwbc9hp91j20g10b1ad3.jpg)



## 怎么用？



示例讲解

### 1. 引入相关包以及权限 



1. **在 Gradle加入Retrofit库的依赖**

   *build.gradle*

```java 
//    compile 'com.squareup.okhttp3:okhttp:3.8.1'   可以不引入，应该retrofit2 会把okhttp引进来
compile 'com.squareup.retrofit2:retrofit:2.0.0-beta3'  
compile 'com.squareup.retrofit2:converter-gson:2.3.0'   // 后面需要用到 
```



2. **加入权限** 

*AndroidManifest.xml*

```xml
<uses-permission android:name="android.permission.INTERNET" />
```



**2. 创建接收服务器数据的类**



*Translation.java*

```
/**
 * Created by tuion on 2017/8/26
 *// URL模板
http://fy.iciba.com/ajax.php

// URL实例
http://fy.iciba.com/ajax.php?a=fy&f=auto&t=auto&w=hello%20world

// 参数说明：
// a：固定值 fy
// f：原文内容类型，日语取 ja，中文取 zh，英语取 en，韩语取 ko，德语取 de，西班牙语取 es，法语取 fr，自动则取 auto
// t：译文内容类型，日语取 ja，中文取 zh，英语取 en，韩语取 ko，德语取 de，西班牙语取 es，法语取 fr，自动则取 auto
// w：查询内容
 
 */

public class Translation {

    private String status;
    private content content;
    private static class content{
        private String from;
        private String to;
        private String vendor;
        private String out;
        private int errNo;
    }

    public void show() {
        System.out.println(status);
        System.out.println(content.from);
        System.out.println(content.to);
        System.out.println(content.vendor);
        System.out.println(content.out);
        System.out.println(content.errNo);
    }

}
```





3. 创建**描述网络请求**的**接口**——采用 **注解 **描述



*GetRequestInterface.java*

```
/**
 * Created by tuion on 2017/8/26.
 */

//采用 注解 描述 网络请求参数
public interface GetRequestInterface {

    // 注解里传入 网络请求 的部分URL地址
    // Retrofit把网络请求的URL分成了两部分：一部分放在Retrofit对象里，另一部分放在网络请求接口里
    // 如果接口里的url是一个完整的网址，那么放在Retrofit对象里的URL可以忽略
    // getCall()是接受网络请求数据的方法

    @GET("ajax.php?a=fy&f=auto&t=auto&w=你好")
    Call<Translation> getCall();
}
```



4. 创建Retrofit对象——创建 网络请求接口 的实例——发送网络请求



```
package com.example.tuionf.retrofitlearn;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.trans).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                request();
            }
        });

    }

    // 使用Retrofit封装的方法
    private void request() {

        //步骤4:创建Retrofit对象
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fy.iciba.com/") //http://fy.iciba.com/
                .addConverterFactory(GsonConverterFactory.create())
                .build();

        // 步骤5:创建 网络请求接口 的实例
        GetRequestInterface request = retrofit.create(GetRequestInterface.class);
        Call<Translation> call = request.getCall();

        //步骤6:发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
                Log.e(TAG, "onResponse: "+response.body() );

                Translation translation = response.body();
                translation.show();
            }

            @Override
            public void onFailure(Call<Translation> call, Throwable t) {

            }
        });
    }
}
```



## 详细说明 

#### 注解说明 



![](https://ws1.sinaimg.cn/large/534fc2d6gy1fiwpnrl19aj20hs0f4q4m.jpg)

a. @GET、@POST、@PUT、@DELETE、@HEAD——以上方法分别对应 HTTP中的网络请求方式



```java
//采用 注解 描述 网络请求参数
public interface GetRequestInterface {
    @GET("ajax.php?a=fy&f=auto&t=auto&w=你好")
    Call<Translation> getCall();
}
```



@GET——作用：采用 Get方式发送网络请求

getCall——接收请求网络数据的方法



### 数据解析器（Converter）——作用：是对于Call中T的转换

- Retrofit支持多种数据解析方式
- 使用时需要在Gradle添加依赖



![](https://ws1.sinaimg.cn/large/534fc2d6ly1fiwpua9w1qj20hs08oq6j.jpg)



- 示例中加入了Gson数据转换器 
- 这样在网络请求的时候回调结果中可以很方便的得到想要的数据类型 

```java
            Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("http://fy.iciba.com/") //http://fy.iciba.com/
            .addConverterFactory(GsonConverterFactory.create())   // 此处加入Gson数据转换器
            .build();
```


```java
call.enqueue(new Callback<Translation>() {
    @Override
    public void onResponse(Call<Translation> call, Response<Translation> response) {
        Log.e(TAG, "onResponse: "+response.body() );
		//加入Gson数据转换器之后，回调中直接就是需要的对象类型 
        Translation translation = response.body();
        translation.show();
    }

    @Override
    public void onFailure(Call<Translation> call, Throwable t) {

    }
});
```



###  Retrofit提供的CallAdapter



Converter是对于Call中T的转换，而**CallAdapter则可以对Call转换**，这样的话Call中的Call也是可以被替换的，而返回值的类型就决定你后续的处理程序逻辑，同样Retrofit提供了多个CallAdapter



![](https://ws1.sinaimg.cn/large/534fc2d6ly1fiwqdheuktj20hf04cglt.jpg)



参考：[这是一份很详细的 Retrofit 2.0 使用教程（含实例讲解）](https://mp.weixin.qq.com/s?__biz=MzIwMzYwMTk1NA==&mid=2247485739&idx=1&sn=751318824da6b00507c3023207476f84&chksm=96cda866a1ba2170d2ac32d93147a4302ab3e083e96d55966d7e7f545564e02463d4c5083a92&scene=0#rd)