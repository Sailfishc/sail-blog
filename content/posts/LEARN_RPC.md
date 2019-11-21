---
title: 浅析RPC
date: 2017-04-29 14:37:00
description: "浅析RPC"
categories: [RPC]
tags: [Java, RPC, SOA]
---

> 目前很多应用应该都不是one in all模式了，避免不了发生远程调用，和同事聊了下RPC之后，发现大家对RPC的概念还是比较模糊的，虽然一直在用，但是不太明白其含义，在知乎上搜了下RPC HTTP这俩个关键词，发现还是有很多误解的。
- 问题一：既然有http 请求，为什么还要用rpc调用？
- 问题二：请问rpc协议和http协议的关系和区别？

> 之后就萌生了写一篇关于RPC的文章。

## 一、RPC的基本概念
RPC，即 Remote Procedure Call（远程过程调用），说得通俗一点就是：调用远程计算机上的服务，就像调用本地服务一样。
RPC的实现包含了两部分，一部分是客户端，一部分是服务端，服务的调用方发送RPC请求到服务提供方，服务提供方根据参数执行方法，响应客户端，一次RPC请求结束。
这篇文章解释的不错：[通俗的语言解释什么是 RPC 框架](https://www.zhihu.com/question/25536695)

![image.png](http://upload-images.jianshu.io/upload_images/734456-56600e1178de48f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

RPC 可基于 HTTP 或 TCP 协议，Web Service 就是基于 HTTP 协议的 RPC，它具有良好的跨平台性，但其性能却不如基于 TCP 协议的 RPC。会两方面会直接影响 RPC 的性能，一是传输方式，二是序列化。

众所周知，TCP 是传输层协议，HTTP 是应用层协议，而传输层较应用层更加底层，在数据传输方面，越底层越快，因此，在一般情况下，TCP 一定比 HTTP 快。就序列化而言，Java 提供了默认的序列化方式，但在高并发的情况下，这种方式将会带来一些性能上的瓶颈，于是市面上出现了一系列优秀的序列化框架，比如：Protobuf、Kryo、Hessian、Jackson 等，它们可以取代 Java 默认的序列化，从而提供更高效的性能。

针对对象序列化，有各种方式的性能对比，[Github地址](https://github.com/eishay/jvm-serializers/wiki):

![image.png](http://upload-images.jianshu.io/upload_images/734456-ff9c09c4b7ae2916.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过对比可知：
- Google的Protostuff性能最好
- JSON/XML性能比较差

但是JSON/XML方式在互联网领域应用比较广泛，第三方的解析包也比较容易使用，所以在效率要求不是很高的情况下是一种不错的选择。

dubbo作为一种服务治理框架，RPC作为其中的内部通信方式，使用也是非常简单：
```
@Component
public class CityDubboConsumerService {

    @Reference(version = "1.0.0")
    CityDubboService cityDubboService;

    public void printCity() {
        String cityName="xx";
        City city = cityDubboService.findCityByName(cityName);
        System.out.println(city.toString());
    }
}
```
- 在不理解RPC概念的情况下，会认为RPC就只有这种应用，其实开发中经常使用的HTTPClient调用也是属于RPC的一种方式。

## 二、RPC的使用
### 1、基于TCP的远程调用

- 服务消费者

```
public class Consumer {


	public static void main(String[] args) throws UnknownHostException, IOException, SecurityException, NoSuchMethodException, ClassNotFoundException{

		//接口名称
		String interfacename= SayHelloService.class.getName();

		//需要远程执行的方法
		Method method = SayHelloService.class.getMethod("sayHello", java.lang.String.class);

		//需要传递到远端的参数
		Object[] arguments = {"hello"};

		Socket socket = new Socket("127.0.0.1", 1234);

		//将方法名称和参数传递到远端
		ObjectOutputStream output = new ObjectOutputStream(socket.getOutputStream());
		output.writeUTF(interfacename); //接口名称
		output.writeUTF(method.getName());  //方法名称
		output.writeObject(method.getParameterTypes());
		output.writeObject(arguments);

		//从远端读取方法执行结果
		ObjectInputStream input = new ObjectInputStream(socket.getInputStream());
		Object result = input.readObject();

		//使用代理对象来处理，直接返回string类型

		System.out.println(result);
	}
}
```

- 服务提供者

```
public class Provider {

	//所有的服务
	private static Map<String,Object> services = new HashMap<String,Object>();

	static{
		services.put(SayHelloService.class.getName(), new SayHelloServiceImpl());
	}

	public static void main(String[] args) throws IOException, ClassNotFoundException, SecurityException, NoSuchMethodException, IllegalArgumentException, IllegalAccessException, InvocationTargetException{

		ServerSocket server = new ServerSocket(1234);
		while(true) {
			Socket socket = server.accept();

			//读取服务信息
			ObjectInputStream input = new ObjectInputStream(socket.getInputStream());
			String interfacename = input.readUTF(); //接口名称
			String methodName = input.readUTF();  //方法名称
			Class<?>[] parameterTypes = (Class<?>[])input.readObject();  //参数类型
			Object[] arguments = (Object[])input.readObject();  //参数对象

			//执行调用
			Class serviceinterfaceclass = Class.forName(interfacename);//得到接口的class
			Object service = services.get(interfacename);//取得服务实现的对象
			Method method = serviceinterfaceclass.getMethod(methodName, parameterTypes);//获得要调用的方法
			Object result = method.invoke(service, arguments);

			ObjectOutputStream output = new ObjectOutputStream(socket.getOutputStream());
			output.writeObject(result);
		}
	}
}
```

- 接口

```
public interface SayHelloService {

	/**
	 * 问好的接口
	 * @param helloArg 参数
	 * @return
	 */
	public String sayHello(String helloArg);
}
```

- 实现类

```
public class SayHelloServiceImpl implements SayHelloService {

	@Override
	public String sayHello(String helloArg) {

		if(helloArg.equals("hello")){
			return "hello";
		}else{
			return "bye bye";
		}

	}

}
```

### 2、基于HTTP的远程调用

- 基础服务接口

```
public interface BaseService {

	public Object execute(Map<String,Object> args);
}
```

- JSON结果集

```
public class JsonResult {

	//结果状态码
	private int resultCode;
	//状态码解释消息
	private String message;
	//结果
	private Object result;

	public int getResultCode() {
		return resultCode;
	}
	public void setResultCode(int resultCode) {
		this.resultCode = resultCode;
	}
	public String getMessage() {
		return message;
	}
	public void setMessage(String message) {
		this.message = message;
	}
	public Object getResult() {
		return result;
	}
	public void setResult(Object result) {
		this.result = result;
	}
}
```

- JSON帮助类

```
public class JsonUtil {

	private static final ObjectMapper mapper = new ObjectMapper();

	public static Object jsonToObject(String json, Class cls) {

		try{
			//允许json串里面的key value不带双引号
			mapper.configure(org.codehaus.jackson.JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);

			// 允许制定的object中的属性没有json串中某个key
			mapper.configure(org.codehaus.jackson.map.DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES, false);

			return mapper.readValue(json, cls);

		}catch(Exception e){}

		return null;

	}

	public static String getJson(Object object)  {

		try{
			String json = null;

			StringWriter sw = new StringWriter();
			JsonGenerator gen = new JsonFactory().createJsonGenerator(sw);
			mapper.writeValue(gen, object);
			gen.close();
			json = sw.toString();
			return json;

		}catch(Exception e){}

		return null;
	}
}
```

```
public class SayHelloService implements BaseService{

	public Object execute(Map<String, Object> args) {
		//request.getParameterMap() 取出来为array,此处需要注意
		String[] helloArg = (String[]) args.get("arg1");

		if("hello".equals(helloArg[0])){
			return "hello";
		}else{
			return "bye bye";
		}
	}

}
```

- 服务消费者

```
public class ServiceConsumer extends HttpServlet{

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {

		this.doPost(req, resp);
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {

		//参数
		String service = "com.http.sayhello";
		String format = "json";
		String arg1 = "hello";


		String url = "http://localhost:8080//testhttprpc/provider.do?"+"service=" + service + "&format=" + format + "&arg1=" + arg1;

		//组装请求
		HttpClient httpClient = new DefaultHttpClient();
		HttpGet httpGet = new HttpGet(url);

		//接收响应
		HttpResponse response = httpClient.execute(httpGet);

		HttpEntity entity = response.getEntity();
		byte[] bytes = EntityUtils.toByteArray(entity);
		String jsonresult = new String(bytes, "utf8");

		JsonResult result = (JsonResult)JsonUtil.jsonToObject(jsonresult, JsonResult.class);

		resp.getWriter().write(result.getResult().toString());

	}
}
```

- 服务提供者

```
public class ServiceProvider  extends HttpServlet{

	private Map<String,BaseService> serviceMap ;


	@Override
	public void init() throws ServletException {
		//服务map初始化
		serviceMap = new HashMap<String,BaseService>();
		serviceMap.put("com.http.sayhello", new SayHelloService());
	}

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {

		this.doPost(req, resp);
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {

		//基本参数
		String servicename = req.getParameter("service");
		String format = req.getParameter("format");

		Map parameters =  req.getParameterMap();

		BaseService service = serviceMap.get(servicename);
		Object result = service.execute(parameters);

		//生成json结果集
		JsonResult jsonResult = new JsonResult();
		jsonResult.setResult(result);
		jsonResult.setMessage("success");
		jsonResult.setResultCode(200);

		String json = JsonUtil.getJson(jsonResult);
		resp.getWriter().write(json);
	}
}
```

### 3、URL风格
- RPC风格的URL
- RESTFUL风格的URL

**RPC风格的URL**
`http://hostname/provider.do?service=com.http.sayhello&format=json&timest amp=2017-04-07-13-22-09&arg1=arg1&arg2=arg2`
- hostname表示服务提供方的主机名
- service表示远程调用的服务接口名称
-  format表示返回参数的格式
- timestamp表示客户端请求的时间戳
- arg1和 arg2表示服务所需要的参数
- 备注：淘宝开放平台的API以这种形式的URL提供


**RESTFUL风格的URL**
```
POST http://hostname/people 创建name为zhangsan的people记录
GET http://hostname/people/zhangsan 返回name为zhangsan的people记录
PUT http://hostname/people/zhangsan 提交name为zhangsan的people记录更新 
DELETE http://hostname/people/zhangsan 删除name为zhangsan的people记录
```
## 三、总结
> - 本文内容部分摘自《大型分布式网站架构》
