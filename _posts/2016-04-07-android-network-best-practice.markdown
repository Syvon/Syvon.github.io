---
layout:     post
title:      "网络编程最佳实践"
subtitle:   ""
date:       2016-04-07
author:     "WunWun"
header-img: "img/android-network-best-practice.png"
tags:
    - Android
    - 学习笔记
---

## 使用HTTP协议访问网络

通常情况下我们都应该将这些通用的网络操作提取到一个公共的类里，并提供一个静态方法，当想要发起网络请求的时候只需简单地调用一下这个方法即可。

    public class HttpUtil {
      public static void sendHttpRequest(final String address) {  
            HttpURLConnection connection = null;
            try {
              URL url = new URL(address);
              connection = (HttpURLConnection) url.openConnection();
              connection.setRequestMethod("GET");
              connection.setConnectTimeout(8000);
              connection.setReadTimeout(8000);
              connection.setDoInput(true);
              connection.setDoOutput(true);
              InputStream in = connection.getInputStream();
              BufferedReader reader = new       BufferedReader(newInputStreamReader(in));
              StringBuilder response = new StringBuilder();
              String line;
              while ((line = reader.readLine()) != null) {
                response.append(line);
              }
              if (listener != null) {
                // 回调onFinish()方法
                listener.onFinish(response.toString());
              }
            } catch (Exception e) {
              if (listener != null) {
                //回调onError()方法
                listener.onError(e);
              }
            } finally {
              if (connection != null) {
                connection.disconnect();
              }
            }
          }
    }

以后每当需要发起一条 HTTP 请求的时候就可以这样写：

    String address = "http://www.baidu.com";  
    String response = HttpUtil.sendHttpRequest(address);

在获取到服务器响应的数据后我们就可以对它进行解析和处理了。但是需要注意，网络请求通常都是属于耗时操作，而 sendHttpRequest()方法的内部并没有开启线程，这样就有可能导致在调用 sendHttpRequest()方法的时候使得主线程被阻塞住。

你可能会说，很简单嘛，在 sendHttpRequest()方法内部开启一个线程不就解决这个问题了吗？其实不是像你想象中的那么容易，因为如果我们在 sendHttpRequest()方法中开启了一个线程来发起 HTTP 请求，那么**服务器响应的数据是无法进行返回的**，所有的耗时逻辑都是在子线程里进行的， sendHttpRequest()方法会在服务器还来得及响应的时候就执行结束了，当然也就无法返回响应的数据了。

那么遇到这种情况应该怎么办呢？其实解决方法并不难，只需要使用 Java 的回调机制就可以了，下面就让我们来学习一下回调机制到底是如何使用的。

定义接口

    public interface HttpCallbackListener {
      void onFinish(String response);
      void onError(Exception e);
    }

实现接口

    public class HttpUtil {
      public static void sendHttpRequest(final String address, final HttpCallbackListener listener) {
        new Thread(new Runnable() {
          @Override
          public void run() {
            HttpURLConnection connection = null;
            try {
              URL url = new URL(address);
              connection = (HttpURLConnection) url.openConnection();
              connection.setRequestMethod("GET");
              connection.setConnectTimeout(8000);
              connection.setReadTimeout(8000);
              connection.setDoInput(true);
              connection.setDoOutput(true);
              InputStream in = connection.getInputStream();
              BufferedReader reader = new       BufferedReader(newInputStreamReader(in));
              StringBuilder response = new StringBuilder();
              String line;
              while ((line = reader.readLine()) != null) {
                response.append(line);
              }
              if (listener != null) {
                // 回调onFinish()方法
                listener.onFinish(response.toString());
              }
            } catch (Exception e) {
              if (listener != null) {
                //回调onError()方法
                listener.onError(e);
              }
            } finally {
              if (connection != null) {
                connection.disconnect();
              }
            }
          }
        }).start();
      }
    }

我们首先给 sendHttpRequest()方法添加了一个 HttpCallbackListener 参数，并在方法的内部开启了一个子线程，然后在子线程里去执行具体的网络操作。注意**子线程中是无法通过return 语句来返回数据的**，因此这里我们将服务器响应的数据传入了 HttpCallbackListener 的onFinish()方法中，如果出现了异常就将异常原因传入到 onError()方法中。

另外需要注意的是， onFinish()方法和 onError()方法最终还是在子线程中运行的，因此我们不可以在这里执行任何的 UI 操作，如果需要根据返回的结果来更新 UI，则仍然要使用异步消息处理机制。