---
title: jsoup要点记录
date: 2018-03-16 12:50:17
categories: "Jsoup" 
tags:
	- java爬虫
	- Jsoup
---


### 查找dom元素 

- getElementById: 根据id查询
- getElementsByTag: 根据tag名称查询
- getElementsByClass: 根据样式class名称查询
- getElementsByAttribute: 根据属性名查询
- getElementsByAttributeValue: 根据属性名和属性值查询

实例代码

``` java
	
	import org.apache.http.HttpEntity;
	import org.apache.http.client.methods.CloseableHttpResponse;
	import org.apache.http.client.methods.HttpGet;
	import org.apache.http.impl.client.CloseableHttpClient;
	import org.apache.http.impl.client.HttpClients;
	import org.apache.http.util.EntityUtils;
	import org.jsoup.Jsoup;
	import org.jsoup.nodes.Document;
	import org.jsoup.nodes.Element;
	import org.jsoup.select.Elements;

	public class Demo01 {

		public static void main(String[] args) throws Exception {
		
			/*用heepclient发起请求获取页面*/
			CloseableHttpClient httpClient = HttpClients.createDefault();
			HttpGet httpGet = new HttpGet("https://www.cnblogs.com/");
			CloseableHttpResponse response = httpClient.execute(httpGet);
			HttpEntity entity = response.getEntity();
			String content = EntityUtils.toString(entity, "utf-8");
			response.close();
			Document doc = Jsoup.parse(content); //用jsoup解析
			
			//通过标签名称查询
			Elements elements = doc.getElementsByTag("title");
			Element element = elements.get(0);
			String title = element.text();
			System.out.println("网页标题：" + title);
			
			//获取首个标题标签内容
			Element titleElement = doc.getElementsByTag("title").first(); 
			System.out.println("首个标题： " + titleElement.text());
			
			//通过id查询
			Element idElement = doc.getElementById("site_nav_top"); 
			System.out.println("id查询：" + idElement.text());
			
			//通过class样式获取查询
			Elements itemElements = doc.getElementsByClass("post_item"); 
			System.out.println("**********样式查询**********");
			for(Element e: itemElements) {
				System.out.println(e.text());
				System.out.println("-------");
			}
		
			//属性名称查询
			Elements attrElements = doc.getElementsByAttribute("width");
			System.out.println("**********属性查询**********");
			for(Element e: attrElements) {
				System.out.println(e.toString());
			}
		
			//属性名称和属性值查询
			Elements attrValueElements = doc.getElementsByAttributeValue("target", "_blank");
			System.out.println("**********属性和属性值查询**********");
			for(Element e: attrValueElements) {
				System.out.println(e.toString());
			}
			
		}

	}

```


### 查找dom元素属性值 


实例代码

```java

import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;


public class Demo2 {

	public static void main(String[] args) throws Exception {

		/*用heepclient发起请求获取页面*/
		CloseableHttpClient httpClient = HttpClients.createDefault();
		HttpGet httpGet = new HttpGet("https://www.cnblogs.com/");
		CloseableHttpResponse response = httpClient.execute(httpGet);
		HttpEntity entity = response.getEntity();
		String content = EntityUtils.toString(entity, "utf-8");
		response.close();
		Document doc = Jsoup.parse(content); //用jsoup解析

		
		//获取带有href属性的a标签
		Elements attrElements = doc.select("a[href]");
		System.out.println("**********获取属性值**********");
		for(Element e: attrElements) {
			System.out.println(e.toString());
			System.out.println("-------");
		}
		
		//查找拓展名为gif的img标签
		Elements imgElements = doc.select("img[src$=.gif]");
		System.out.println("**********带有拓展名查询**********");
		for(Element e: imgElements) {
			System.out.println(e.toString());
		}
	}

}


```



### 使用选择器查询

 **Jsoup 支持css，jquery的选择器**

实例代码

``` java

import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;


public class Demo03 {

	public static void main(String[] args) throws Exception {
		
		/*用heepclient发起请求获取页面*/
		CloseableHttpClient httpClient = HttpClients.createDefault();
		HttpGet httpGet = new HttpGet("https://www.cnblogs.com/");
		CloseableHttpResponse response = httpClient.execute(httpGet);
		HttpEntity entity = response.getEntity();
		String content = EntityUtils.toString(entity, "utf-8");
		response.close();
		Document doc = Jsoup.parse(content); //用jsoup解析
		
		//通过选择器查询
		Elements linkElements = doc.select(".headline ul .editor_pick a");
		System.out.println("**********选择器查询**********");
		for(Element e: linkElements) {
			System.out.println(e.toString());
			System.out.println("地址： " + e.attr("href"));
			System.out.println("-------");
		}
		
		Element linkElement = doc.select(".headline ul li").first();
		System.out.println("文本： " + linkElement.text());
		System.out.println("html: " + linkElement.html());
		System.out.println("class属性值： " + linkElement.attr("class"));
			
	}

}

```

