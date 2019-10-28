---
title: AsyncHttpClient如何进行Session的保存
date: 2019-05-11 12:25:24
toc: true
categories: 移动开发
---

在之前我从来没有通过AsyncHttp这个请求框架去保存Session，但是今天不得不用呀！接口设计有限，我也只好用专门给浏览器用的接口了，这次我把AsyncHttpClient封装成了一个单例，通过这样的方式来保存Session，暂时先这么用吧，等到找到其他解决方案了再更新此文！

## 一、封装

先封装一个专门处理Cookie的工具类：

```java
public class CookieUtils {

    private static List<Cookie> cookies;

    /* 返回cookies列表 */
    public static List<Cookie> getCookies() {
        return cookies != null ? cookies : new ArrayList<>();
    }

    /* 设置cookies列表 */
    public static void setCookies(List<Cookie> cookies) {
        CookieUtils.cookies = cookies;
    }

    /* 存储cookie */
    public static void saveCookie(AsyncHttpClient client, Context context) {
        PersistentCookieStore cookieStore = new PersistentCookieStore(context);
        client.setCookieStore(cookieStore);
    }

    /* 得到cookie */
    public static List<Cookie> getCookie(Context context) {
        PersistentCookieStore cookieStore = new PersistentCookieStore(context);
        List<Cookie> cookies = cookieStore.getCookies();
        return cookies;
    }

    /* 清除cookie */
    public static void clearCookie(Context context) {
        PersistentCookieStore cookieStore = new PersistentCookieStore(context);
        cookieStore.clear();
    }
}
```

再封装一下AsyncHttpClient，这样所有的AsyncHttpClient用的都是同一个对象了！

```java
public class FinalAsyncHttpClient {
    private AsyncHttpClient client;
    /* 构造方法 */
    public FinalAsyncHttpClient() {
        client = new AsyncHttpClient();//实例化client
        client.setTimeout(5);//设置5秒超时
        // 获取cookie列表
        if (CookieUtils.getCookies() != null) {
            BasicCookieStore bcs = new BasicCookieStore();
            bcs.addCookies(CookieUtils.getCookies().toArray(
                    new Cookie[CookieUtils.getCookies().size()]));//得到cookie列表
            client.setCookieStore(bcs);//给client加载cookie
        }
    }

    /* 得到client对象方法 */
    public AsyncHttpClient getAsyncHttpClient() {
        return this.client;
    }
}
```

## 二、使用

获取对象并且设置保存Cookie

```java
AsyncHttpClient asyncHttpClient = new FinalAsyncHttpClient().getAsyncHttpClient();
CookieUtils.saveCookie(asyncHttpClient,LoginAty.this);
```

登录成功

```java
asyncHttpClient.post(AppConfig.loginAddress, params, new AsyncHttpResponseHandler() {
  @Override
  public void onSuccess(int statusCode, Header[] headers, byte[] responseBody) {
    	//....
			XToast.success(getContext(), "登录成功").show();
			CookieUtils.setCookies(CookieUtils.getCookie(LoginAty.this));
    	//....
  }
```

## 三、拓展

但是这样也有问题，每次开启App的时候cookie就消失了，我们把cookie通过序列化的方式存起来就好了，如果发现cookie过期，现在有更好的解决方式，直接在App开启的时候就更新一次cookie就好了！