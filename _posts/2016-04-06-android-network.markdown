---
layout:     post
title:      "Android中的网络技术"
subtitle:   "Http，XML，JSON"
date:       2016-04-06
author:     "WunWun"
header-img: "img/android-network.jpg"
tags:
    - Android
    - 学习笔记
    - Http
    - XML
    - JSON
---

## 使用HTTP协议访问网络

> HTTP 协议的工作原理特别的简单，就是客户端向服务器发出一条HTTP 请求，服务器收到请求之后会返回一些数据给客户端，然后客户端再对这些数据进行解析和处理就可以了。

在 Android 上发送 HTTP 请求的方式一般有两种， HttpURLConnection 和 HttpClient.

#### 使用 HttpURLConnection

##### GET方法

    URL url = new URL("http://www.baidu.com");
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("GET");
    connection.setConnectTimeout(8000);
    connection.setReadTimeout(8000);    

    InputStream in = connection.getInputStream();
    connection.disconnect();

在实际工作中，可能会为HttpURLConnection单独开启一个线程。

    new Thread(new Runnable(){
        @Override
        public void run() {
            HttpURLConnection connection = null;
            try {
                URL url = new URL("http://www.baidu.com");
                connection = (HttpURLConnection) url.openConnection();
                connection.setRequestMethod("GET");
                connection.setConnectTimeout(8000);
                connection.setReadTimeout(8000);
                InputStream in = connection.getInputStream();
                // 下面对获取到的输入流进行读取
                BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                StringBuilder response = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    response.append(line);
                }
                Message message = new Message();
                message.what = SHOW_RESPONSE;
                // 将服务器返回的结果存放到Message中
                message.obj = response.toString();
                handler.sendMessage(message);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (connection != null) {
                    connection.disconnect();
                }
            }
        }
    }).start();

##### POST方法

如果是想要提交数据给服务器，只需要将 HTTP 请求的方法改成 POST，并在获取输入流之前把要提交的数据写出即可。注意每条数据都要以键值对的形式存在，数据与数据之间用&符号隔开，比如说我们想要向服务器提交用户名和密码，就可以这样写：

    connection.setRequestMethod("POST");
    DataOutputStream out = new DataOutputStream(connection.getOutputStream());
    out.writeBytes("username=admin&password=123456");

#### 使用 HttpClient

HttpClient 是 Apache 提供的 HTTP 网络访问接口，因此无法创建它的实例，通常情况下都会创建一个 DefaultHttpClient 的实例。

##### 发起一个GET请求

    HttpClient httpClient = new DefaultHttpClient();
    HttpGet httpGet = new HttpGet("http://www.baidu.com");
    HttpResponse httpResponse = httpClient.execute(httpGet);
    if (httpResponse.getStatusLine().getStatusCode() == 200) {
        // 请求和响应都成功了
        HttpEntity entity = httpResponse.getEntity();
        String response = EntityUtils.toString(entity,"utf-8");
    }

##### 发起一个POST请求

    HttpClient httpClient = new DefaultHttpClient();
    HttpPost httpPost = new HttpPost("http://www.baidu.com");   

    List<NameValuePair> params = new ArrayList<NameValuePair>();
    params.add(new BasicNameValuePair("username", "admin"));
    params.add(new BasicNameValuePair("password", "123456"));
    UrlEncodedFormEntity entity = new UrlEncodedFormEntity(params, "utf-8");
    httpPost.setEntity(entity); 

    HttpResponse httpResponse = httpClient.execute(httpGet);
    if (httpResponse.getStatusLine().getStatusCode() == 200) {
        // 请求和响应都成功了
        HttpEntity entity = httpResponse.getEntity();
        String response = EntityUtils.toString(entity,"utf-8");
    }




执行 execute()方法之后会返回一个 **HttpResponse 对象，服务器所返回的所有信息就会包含在这里面**。通常情况下我们都会先取出服务器返回的状态码，如果等于 200 就说明请求和响应都成功了，如下所示：

    if (httpResponse.getStatusLine().getStatusCode() == 200) {
        // 请求和响应都成功了
    }

接下来在这个 if 判断的内部取出服务返回的具体内容，可以调用 getEntity()方法获取到一个 **HttpEntity 实例**，然后再用 EntityUtils.toString()这个静态方法将 HttpEntity 转换成字符串即可，如下所示：

    HttpEntity entity = httpResponse.getEntity();
    String response = EntityUtils.toString(entity);

注意如果服务器返回的数据是带有中文的，直接调用 EntityUtils.toString()方法进行转换会有乱码的情况出现，这个时候只需要在转换的时候将字符集指定成 utf-8 就可以了，如下所示：

    String response = EntityUtils.toString(entity, "utf-8");

在实际工作中，可能会为HttpClient单独开启一个线程。

    new Thread(new Runnable(){
        @Override
        public void run() {
            try {
                HttpClient httpClient = new DefaultHttpClient();
                HttpGet httpGet = new HttpGet("http://www.baidu.com");
                HttpResponse httpResponse = httpClient.execute(httpGet);
                if (httpResponse.getStatusLine().getStatusCode() == 200) {
                    // 请求和响应都成功了
                    HttpEntity entity = httpResponse.getEntity();   

                    String response = EntityUtils.toString(entity,
                    "utf-8");
                    Message message = new Message();
                    message.what = SHOW_RESPONSE;
                    // 将服务器返回的结果存放到Message中
                    message.obj = response.toString();
                    handler.sendMessage(message);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (connection != null) {
                    connection.disconnect();
                }
            }
        }
    }).start();


## 解析XML格式数据

假设要获取如下格式的XML数据

    <apps>
        <app>
            <id>1</id>
            <name>Google Maps</name>
            <version>1.0</version>
        </app>
        <app>
            <id>2</id>
            <name>Chrome</name>
            <version>2.1</version>
        </app>
        <app>
            <id>3</id>
            <name>Google Play</name>
            <version>2.3</version>
        </app>
    </apps>

#### Pull解析方式

    XmlPullParser parser = XmlPullParserFactory.newInstance().newPullParser();  
    parser.setInput(fileInputStream, “utf-8”);//设置数据源编码  
    int eventCode = parser.getEventType();//获取事件类型  
    while(eventCode != XmlPullParser.END_DOCUMENT)  {     
        switch (eventCode){     
            case XmlPullParser.START_DOCUMENT: //开始读取XML文档    
            //实例化集合类    
            break;     
        case XmlPullParser.START_TAG://开始读取某个标签       
            if("person".equals(parser.getName())) {     
            //通过getName判断读到哪个标签，然后通过nextText()获取文本节点值，或通过getAttributeValue(i)获取属性节点值  
            }     
            break;  
        case XmlPullParser.END_TAG://读完一个Person，可以将其添加到集合类中  
            break;  
        }  
        parser.next();  
    } 

测试

    private void parseXMLWithPull(String xmlData) {
      try {
        XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
        XmlPullParser xmlPullParser = factory.newPullParser();
        xmlPullParser.setInput(new StringReader(xmlData));
        int eventType = xmlPullParser.getEventType();
        String id = "";
        String name = "";
        String version = "";
        while (eventType != XmlPullParser.END_DOCUMENT) {
          String nodeName = xmlPullParser.getName();
           switch (eventType) {
              // 开始解析某个结点
            case XmlPullParser.START_TAG: {
              if ("id".equals(nodeName)) {
                id = xmlPullParser.nextText();
              } else if ("name".equals(nodeName)) {
                name = xmlPullParser.nextText();
              } else if ("version".equals(nodeName)) {
                version = xmlPullParser.nextText();
              }
              break;
             }
            // 完成解析某个结点
            case XmlPullParser.END_TAG: {
              if ("app".equals(nodeName)) {
                Log.d("MainActivity", "id is " + id);
                Log.d("MainActivity", "name is " + name);
                Log.d("MainActivity", "version is " + version);
              }
              break;
            }
            default:
              break;
          }
          eventType = xmlPullParser.next();
        }
       } catch (Exception e) {
        e.printStackTrace();
       }
    }


#### SAX解析方式

Sax采用事件驱动的方式解析文档。在Sax的解析过程中，读取到文档开头、结尾，元素的开头和结尾都会触发一些回调方法，你可以在这些回调方法中进行相应事件处理这四个方法是：startDocument() 、 endDocument()、 startElement()、 endElement此外，光读取到节点处是不够的，我们还需要characters()方法来仔细处理元素内包含的内容将这些回调方法集合起来，便形成了一个类，这个类也就是我们需要的触发器一般从Main方法中读取文档，却在触发器中处理文档，这就是所谓的事件驱动解析方法

    public class SaxHandler extends DefaultHandler {

        @Override
        public void startDocument() throws SAXException {

        }
     
        /*arg0是名称空间
          arg1是包含名称空间的标签，如果没有名称空间，则为空
          arg2是不包含名称空间的标签
          arg3很明显是属性的集合 */
        @Override
        public void startElement(String arg0, String arg1, String arg2,
                Attributes arg3) throws SAXException {

        } 

        /* 此方法有三个参数
           arg0是传回来的字符数组，其包含元素内容
           arg1和arg2分别是数组的开始位置和结束位置 */
        @Override
        public void characters(char[] arg0, int arg1, int arg2) throws SAXException {

        }

        /* arg0是名称空间
           arg1是包含名称空间的标签，如果没有名称空间，则为空
           arg2是不包含名称空间的标签 */
        @Override
        public void endElement(String arg0, String arg1, String arg2)
                throws SAXException {

        }

        @Override
        public void endDocument() throws SAXException {

        }
    }

- startDocument()方法会在开始 XML 解析的时候调用，

- startElement()方法会在开始解析某个结点的时候调用， 

- characters()方法会在获取结点中内容的时候调用， 

- endElement()方法会在完成解析某个结点的时候调用， 

- endDocument()方法会在完成整个 XML 解析的时候调用。


其中， startElement()、 characters()和 endElement()这三个方法是有参数的，从 XML 中解析出的数据就会以参数的形式传入到这些方法中。需要注意的是，在获取结点中的内容时， characters()方法可能会被调用多次，一些换行符也被当作内容解析出来，我们需要针对这种情况在代码中做好控制。

测试

    public class ContentHandler extends DefaultHandler {
      private String nodeName;
      private StringBuilder id;
      private StringBuilder name;
      private StringBuilder version;
      @Override
      public void startDocument() throws SAXException {
        id = new StringBuilder();
        name = new StringBuilder();
        version = new StringBuilder();
      }
      @Override
      public void startElement(String uri, String localName, String qName,Attributes attributes) throws SAXException {
        // 记录当前结点名
        nodeName = localName;
      }
      @Override
      public void characters(char[] ch, int start, int length)   throwsSAXException {
        // 根据当前的结点名判断将内容添加到哪一个StringBuilder对象中
        if ("id".equals(nodeName)) {
          id.append(ch, start, length);
        } else if ("name".equals(nodeName)) {
          name.append(ch, start, length);
        } else if ("version".equals(nodeName)) {
          version.append(ch, start, length);
        }
      }
      @Override
      public void endElement(String uri, String localName, String   qName) throwsSAXException {
        if ("app".equals(localName)) {
          Log.d("ContentHandler", "id is " + id.toString().trim());
          Log.d("ContentHandler", "name is " + name.toString().trim());
          Log.d("ContentHandler", "version is " +     version.toString().trim());
        // 最后要将StringBuilder清空掉id.setLength(0);
          name.setLength(0);
          version.setLength(0);
        }
      }
      @Override
      public void endDocument() throws SAXException {
      }
    }   

    private void parseXMLWithSAX(String xmlData) {
      try {
        SAXParserFactory factory = SAXParserFactory.newInstance();
        XMLReader xmlReader =   factory.newSAXParser().getXMLReader();
        ContentHandler handler = new ContentHandler();
        // 将ContentHandler的实例设置到XMLReader中  
        xmlReader.setContentHandler(handler);
        // 开始执行解析
        xmlReader.parse(new InputSource(new   StringReader(xmlData)));
      } catch (Exception e) {
        e.printStackTrace();
      }
    }

## 解析JSON格式数据

JSON相对于xml体积更小,但是语义较差,没xml直观.解析JSON可以使用官方提供的 JSONObject，也可以使用谷歌的开源库 GSON。或者第三方的开源库如 Jackson、 FastJSON .

假设要获取如下格式的JSON数据

#### 使用JSONObject

    private void parseJSONWithJSONObject(String jsonData) {
        try {
            JSONArray jsonArray = new JSONArray(jsonData);
            for (int i = 0; i < jsonArray.length(); i++) {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                String id = jsonObject.getString("id");
                String name = jsonObject.getString("name");
                String version = jsonObject.getString("version");
                Log.d("MainActivity", "id is " + id);
                Log.d("MainActivity", "name is " + name);
                Log.d("MainActivity", "version is " + version);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

#### 使用GSON

    public class App {
      private String id;
      private String name;
      private String version;
      public String getId() {
        return id;
      }
      public void setId(String id) {
        this.id = id;
      }
      public String getName() {
        return name;
      }
      public void setName(String name) {
        this.name = name;
      }
      public String getVersion() {
        return version;
      }
      public void setVersion(String version) {
        this.version = version;
      }
    }   

    private void parseJSONWithGSON(String jsonData) {
      Gson gson = new Gson();
      List<App> appList = gson.fromJson(jsonData, new TypeToken<List<App>>() {}.getType());
      for (App app : appList) {
        Log.d("MainActivity", "id is " + app.getId());
        Log.d("MainActivity", "name is " + app.getName());
        Log.d("MainActivity", "version is " + app.getVersion());
      }
    }

