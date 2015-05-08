---
layout: post
category: "python"
title: "阿里云[3]－ ODPS自定义UDF解析json字符串"
tags: ["大数据技术分享"]
---

#### 1.吐槽阿里ODPS：
- 带队用阿里的SLS+ODPS折腾数据，本想着从另一套hadoop自建集群照搬流程过来，可惜这条技术路坑比较多
- SLS收录日志功能只能收取客户端前五分钟的数据，如果出故障就再也折腾不进去了，我技术挫，重指时间列，改时间，改系统时间..... 问阿里技术曰：就是只能收前五钟的，原理不可说。
- SLS数据归档问题，日志数据中，假如只有两列，一列时间，一列json，归档到ODPS中，你会发现数据归为了一列，并且json化，也就意味着原来的json又被json化了一遍，会变成json字符串，加斜线了，问曰：设计问题，会改正，可自己写udf解之。
- 仅仅这样开始了苦逼的udf解析json之路。其实后还有坑更大




#### 2.神我拿什么写UDF：
翻文档，有两种方式：java和python，心想python吧，团队其他人也会折腾，省得以后麻烦，开始找文档，耕遍文档库都没有，心想用“odps udf python”来google之，搜之无一字，无力问度娘。算了吧，拿java来折腾吧，想偷懒也不成了。然后下eclipse,写代码，打jar，代码写了十来分钟，怕代码有问题，用正则试了一把没问题，大有搞头的udf开始了,代码如下：

```
package playcrab.com.aliyun.odps.udf;

import java.util.Map;

import com.aliyun.odps.udf.UDF;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;

public final class custom_get_json_object extends UDF {

	public static String json2object(String jsonStr, String key) {
		try {
			Map map1 = JSON.parseObject(jsonStr);
			return JSON.toJSONString(map1.get(key.toString()));

		} catch (Exception e) {

			JSONArray jarr = JSONArray.parseArray(jsonStr);// JSON.parseArray(jsonStr);
			return JSON.toJSONString(jarr.get(Integer.parseInt(key)));
		}
	}

	public static String json2array(String jsonStr, String index) {

		JSONArray jarr = JSONArray.parseArray(jsonStr);// JSON.parseArray(jsonStr);
		return JSON.toJSONString(jarr.get(Integer.parseInt(index)));

	}

	public  String evaluate(String jsonStr, String index) {

		if (jsonStr == null) {
			return null;
		}

		jsonStr.replaceAll("\\\\\"", "\"");
		jsonStr.replace("\"{\"", "{\"");
		jsonStr.replace("}\"}", "}}");

		try {
			String[] keys = index.split("\\.");

			for (int i = 1; i < keys.length; i++) {
				
				jsonStr = json2object(jsonStr, keys[i]);
				
			}
			return jsonStr;

		} catch (Exception e) {
			
			String[] keys = index.split("\\.");

			for (int i = 1; i < keys.length; i++) {
				jsonStr = json2array(jsonStr, keys[i]);
			}
			return jsonStr;
		
		}

	}

	public static void main(String[] args) {

		//String jsonStr = "{\"a\":{\"b\":{\"c\":[\"fff\",\"ddddd\"]}}}";
		//System.out.println(evaluate(jsonStr, "$.a.b.c.1"));
		
		//String jsonStr ="[{'age':22,'sex':'男','userName':'xiaoliang'},{'age':23,'sex':'男','userName':'xiaoliang'}]";
		//System.out.println(evaluate(jsonStr, "$.0.age"));
		
	    // select
		// get_json_object(regexp_replace(regexp_replace(regexp_replace(sls_extract_others,'\\\\\"','"'),'"{"','{"'),'}"}','}}'),'$.content.content.action')
		// from z limit 1;

	}

}

```
- 经过如下几步满怀期望
- drop resource custom_get_json_object.jar;
- create resource jar custom_get_json_object.jar;

- create function custom_get_json_object playcrab.com.aliyun.odps.udf.custom_get_json_object custom_get_json_object.jar
- 执行后，select custom_get_json_object(sls_extract_others,'$.content.content.zhi') from z limit 1;
- 好吧报错，说是找不到第三方fastjson
- 问曰：不支持第三方包，必须用odps自己的包，好吧看了一眼果然有一json－1.0.jar，心想大爷的我怎么知道你有哪些方法，只能把包全加到lib中，import试，真佩服自己，又过了一遍jar，看见有gson，终于有希望了，这个是常用的包哈，快速扫了一遍方法，改吧，改吧，改的一点都不能依赖阿里之外的第三方包，爷！fastjson不是阿里的么........

```
package playcrab.com.aliyun.odps.udf;

import java.util.Map;

import com.aliyun.odps.udf.UDF;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.reflect.TypeToken;
import com.google.gson.internal.LinkedTreeMap;

public class custom_get_json_object extends UDF {

	/**
	 *
	 * 函数名称: parseData 函数描述: 将json字符串转换为map
	 * 
	 * @param data
	 * @return
	 */
	private static Map<String, String> parseData(String data) {

		GsonBuilder gb = new GsonBuilder();
		Gson g = gb.create();
		Map<String, String> map = g.fromJson(data,
				new TypeToken<Map<String, String>>() {
				}.getType());
		return map;
	}

	public static String json2object(String jsonStr, String key, int is_last) {
		try {
			// JSON转map后get指定key
			return (String) parseData(jsonStr).get(key);

		} catch (Exception e) {
			//System.out.println(e.toString());
			// 本身就是json串，变为map 指定key
			Gson gson = new Gson();
			Map depts = gson.fromJson(jsonStr, Map.class);
			if (is_last == 0) {
				return gson.toJson(depts.get(key));
			} else {
				return depts.get(key).toString();
			}
		}
	}

	// TODO define parameters and return type, e.g., public Long evaluate(Long
	// a, Long b)
	public String evaluate(String jsonStr, String index) {
		//String index;
		//index = "$.content.game";
		String[] keys = index.split("\\.");

		for (int i = 1; i < keys.length; i++) {
			//System.out.println(keys.length);
			//System.out.println(i);
			if (i < keys.length - 1) {
				jsonStr = json2object(jsonStr, keys[i], 0);
			} else {
				jsonStr = json2object(jsonStr, keys[i], 1);
			}

		}
		return jsonStr.toString();

	}

}

```
打包，上传，使用，没问题了，好吧，刚才漏了一点，上传使用过程中，因为刚才我上传过一个jar并且创建函数了，后面的就不能用相同的函数名了，不知道能不能删除原来的。好吧，终于能使用了，阿门！


>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
