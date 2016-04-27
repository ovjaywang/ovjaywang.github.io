---
title: 利用Gson转换json数据和java对象
categories:
  - 代码狗
  - 学习log
tags:
  - java
id: 189
date: 2013-05-17 10:03:39
---

一直在使用json-lib包
每次得到数据都得转换半天 除了实体类外还要写一个影射类 非常的麻烦
利用Gson包就可以直接导入整个实体类以转换 不过需要的是json的数据名称与实体类的名称要一致
首先是实体类
```java
public class User {
	private String userName;
	private String psw;
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public String getPsw() {
		return psw;
	}
	public void setPsw(String psw) {
		this.psw = psw;
	}
	public User(String userName, String psw) {
		super();
		this.userName = userName;
		this.psw = psw;
	}
	@Override
	public String toString() {
		return &quot;userName= &quot; + userName + &quot; psw= &quot; + psw;
	}

}
```
利用Gson输出Json数据
----原版的利用影射类
```java
public class MyServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;
	public MyServlet() {
		super();
	}
	public void destroy() {
		super.destroy(); 
	}

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		this.doPost(request, response);
	}

	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		response.setContentType(&quot;text/html;charset=UTF-8&quot;);
		PrintWriter out = response.getWriter();
		JSONArray array = new JSONArray();
		 JdbcTemplate template=new JdbcTemplate();
		 List&lt;User&gt;  list=null;		 
		 try {
			 list=(List&lt;User&gt;)template.findAll(&quot;select * from User&quot;, new UserMapping());
//数据库已连接好的情况下...User是表 UserMapping是实体类
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		 	for (int i = 0; i &lt; list.size(); i++) 
		 		{
				 JSONObject temp = new JSONObject()   
		         .element( &quot;Name&quot;,list.get(i).getUserName() )  
		         .element( &quot;password&quot;, list.get(i).getPsw() );		   
				 array.add(temp);	}
			out.print(array.toString());
			System.out.println(array.toString());
		} 
		}
```
现在利用Gson就可以了

```java
                Gson gson = new Gson();
		String msgJson = gson.toJson(list);
```
利用Gson转换json数据
转换单一字符串 如：
[{"userName":"userName","pwd":123}]
User user= gson.fromJson(str, User.class);
转换字符串组， 如：
```java
List<User> users = gson.fromJson(str, new TypeToken<List<User>>(){}.getType());
for(int i = 0; i < users.size() ; i++)
{
     User user = users.get(i);
     System.out.println(user.toString());//这里想做什么都可以了 记住必须是实体类的名字和json中传送的名字相同
}
```