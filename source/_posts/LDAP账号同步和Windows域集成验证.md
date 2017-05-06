---
title: LDAP账号同步和Windows域集成验证
date: 2014-03-13 17:31
categories: 
tags: 
---
<h2><span style="font-family: 微软雅黑;"><strong><span style="font-size: 18px;">应用场景</span></strong></span></h2> 
<p><span style="font-family: 微软雅黑; font-size: 14px;"></span></p> 
<div style="font-family: 微软雅黑; font-size: 14px;"> 
 <ul> 
  <li><span style="">应用系统中的账号信息除了本地创建的之外，还要有LDAP中的，并且随时与LDAP中的最新数据一致；</span></li> 
  <li><span style="">公司所有人的电脑都在一个域中管理，员工通过域账号和密码登录他的计算机之后，在登录应用系统之后不再需要输入密码，直接进入系统；</span></li> 
  <li><span style="">如果员工拥有多个不同的域账号和密码，那么他也可以在选择任意一个域账号来登录应用系统（而不仅仅是登录计算机那个域账号和密码）；</span></li> 
 </ul> 
</div> 
<span style="font-family: 微软雅黑;"><strong><span style="font-size: 18px;">目标功能</span></strong></span>
<br>
<p></p> 
<p><span style="font-family: 微软雅黑;"><span style="font-family: 微软雅黑;"></span></span></p> 
<div style="font-family: 微软雅黑;"> 
 <ul style="font-size: 14px;"> 
  <li> <span style="">1，LDAP账号同步<br></span><span style="">把LDAP中的用户数据同步到本地数据中</span> </li> 
  <li> <span style="">2，LDAP用户登录验证<br></span><span style="">根据用户提供的用户名和密码验证用户是否为合法的域用户</span> </li> 
  <li> <span style="">3，Windows域集成验证<br></span><span style="">比如一个信息管理系统，当用户使用域中（域控，ActiveDirectory）的计算机登录信息管理系统的时候，由于该用户在登录计算机的时候已经通过了身份验证，所以不需要再次输入用户名和密码而直接进入信息管理系统。<br></span> </li> 
 </ul> 
 <div style="font-size: 14px;">
  测试环境准备
 </div> 
 <div style="font-size: 14px;">
  服务器IP:
  <span style="font-family: 微软雅黑; font-size: 14px;">192.168.116.128</span> 
 </div> 
 <div style="font-size: 14px;">
  服务器域信息：
 </div> 
 <div style="font-size: 14px;"> 
  <img alt="" style="display: inline-block; font-family: 微软雅黑; font-size: 14px;">
  <br> 
 </div> 
 <div style="font-size: 14px;"> 
  <img src="http://img.blog.csdn.net/20140313171933421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="">
  <br> 
 </div> 
 <div style="font-size: 14px;">
  <br>
 </div> 
 <div style="font-size: 14px;"> 
  <span style="font-family: 微软雅黑; font-size: 14px;">Active Directory 状态：</span>
  <br> 
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><img src="http://img.blog.csdn.net/20140313172140765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaW1odXFpYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt=""><br></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><br></span>
 </div> 
 <div>
  <span style="font-family: 微软雅黑;"><strong><span style="font-size: 18px;"><span style="font-family: 微软雅黑;">功能：</span><span style="font-family: 微软雅黑;">LDAP账号同步</span></span></strong><br></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><br></span></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;">首先，需要获取到LdapContext：</span><br></span></span>
 </div> 
 <div style="font-size: 14px;"> 
  <pre name="code" class="java">	private LdapContext getLdapContext()throws NamingException{
		Hashtable&lt;String,String&gt; hashtable = new Hashtable&lt;String,String&gt;();
		hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
		hashtable.put(Context.PROVIDER_URL, "ldap://192.168.116.128:389");//服务器地址
		hashtable.put(Context.SECURITY_AUTHENTICATION, "simple");
		hashtable.put(Context.SECURITY_PRINCIPAL, "Administrator@abc.com");//用户名
		hashtable.put(Context.SECURITY_CREDENTIALS, "321%cba");//密码
		return new InitialLdapContext(hashtable,null);
	}</pre> 
  <br>
  <span style="font-family: 微软雅黑; font-size: 14px;">然后进行相关的搜索设置：</span> 
 </div> 
 <div style="font-size: 14px;"> 
  <span style="font-family: 微软雅黑; font-size: 14px;"></span>
  <pre name="code" class="java">		LdapContext  ctx = getLdapContext();
                //设置分页大小
		ctx.setRequestControls(new Control[] { new PagedResultsControl(15, Control.NONCRITICAL) });
		
		SearchControls control = new SearchControls();
		
		//搜索方式
		control.setSearchScope(SearchControls.SUBTREE_SCOPE);//Search the entire subtree rooted at the named object. 
		//control.setSearchScope(SearchControls.ONELEVEL_SCOPE);//Search one level of the named context
		//control.setSearchScope(SearchControls.OBJECT_SCOPE);//Search the named object
		
		//搜索字段
		String returnedAtts[] = { "displayName", "mail", "telephoneNumber","thumbnailPhoto" };//姓名，邮箱，电话，头像
		control.setReturningAttributes(returnedAtts);
		
		//设置ou和filter
		String ou = "ou=users,ou=beijing,dc=abc,dc=com";
		String filter = "(&amp;(objectClass=user)(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))";</pre> 
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;">最后，发起搜索请求，解析搜索结果：</span>
 </div> 
 <div style="font-size: 14px;"> 
  <span style="font-family: 微软雅黑; font-size: 14px;"></span>
  <pre name="code" class="java">NamingEnumeration&lt;SearchResult&gt; results = ctx.search(ou, filter, control);
		while (results != null &amp;&amp; results.hasMoreElements()) {
			SearchResult entry = (SearchResult) results.next();
			String empName = getValueFromAttribute(entry.getAttributes().get(returnedAtts[0]));
			String mail = getValueFromAttribute(entry.getAttributes().get(returnedAtts[1]));
			String telephone = getValueFromAttribute(entry.getAttributes().get(returnedAtts[2]));
			byte[] photoBytes = null;
			Attribute att = (Attribute) entry.getAttributes().get("thumbnailPhoto");
			if(att!=null){
				photoBytes = (byte[])(att.get(0));
			}
			System.out.println(empName+"|"+mail+"|"+telephone+"|"+(photoBytes==null ? 0 : photoBytes.length));
		}</pre> 
 </div> 
 <div> 
  <span style="font-family: 微软雅黑;"><strong><span style="font-size: 18px;">功能：LDAP用户登录验证</span></strong></span>
  <br> 
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><br></span></span></span></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;">我的实现方式如下，有更好方法的朋友还请指教：</span><br></span></span></span>
 </div> 
 <div> 
  <span style="font-size: 14px; font-family: 微软雅黑;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"></span></span></span></span>
  <pre name="code" class="java" style="font-size: 14px;">private boolean validate(String username,String pwd)throws NamingException{
		Hashtable&lt;String,String&gt; hashtable = new Hashtable&lt;String,String&gt;();
		hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
		hashtable.put(Context.PROVIDER_URL, "ldap://192.168.116.128:389");//服务器地址
		hashtable.put(Context.SECURITY_AUTHENTICATION, "simple");
		hashtable.put(Context.SECURITY_PRINCIPAL, username);//用户名
		hashtable.put(Context.SECURITY_CREDENTIALS, pwd);//密码
		return new InitialLdapContext(hashtable,null)!=null;
	}</pre> 
  <br>
  <span style="font-family: 微软雅黑;"><strong><span style="font-size: 18px;"><br></span></strong></span> 
 </div> 
 <div> 
  <span style="font-family: 微软雅黑;"><strong><span style="font-size: 18px;">功能：Windows集成验证登录</span></strong></span>
  <br> 
 </div> 
 <div style="font-size: 14px;"> 
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"></span></span></span></span></span> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   <br>
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   这个功能，使用到了工具
   <a href="http://tomcatspnego.codeplex.com/" target="_blank" style="">http://tomcatspnego.codeplex.com/</a> 
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   下载工具后解压，然后：
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;"> 
   <div>
    复制jar包frdoumesspitc7.jar到tomcat的/lib目录下
   </div> 
   <div>
    复制SSPAuthentification.dll和SSPAuthentificationx64.dll到tomcat的/bin目录下
   </div> 
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   在应用的web.xml中增加如下配置：
  </div> 
  <pre name="code" class="html">&lt;security-constraint&gt;
      &lt;display-name&gt;Example Security Constraint&lt;/display-name&gt;
      &lt;web-resource-collection&gt;
         &lt;web-resource-name&gt;Protected Area&lt;/web-resource-name&gt;
	 	 &lt;!-- Define the context-relative URL(s) to be protected --&gt;
         &lt;url-pattern&gt;/auth.do&lt;/url-pattern&gt;
	 	 &lt;!-- If you list http methods, only those methods are protected --&gt;
	 	 &lt;http-method&gt;DELETE&lt;/http-method&gt;
         &lt;http-method&gt;GET&lt;/http-method&gt;
         &lt;http-method&gt;POST&lt;/http-method&gt;
	 	 &lt;http-method&gt;PUT&lt;/http-method&gt;
      &lt;/web-resource-collection&gt;
      &lt;auth-constraint&gt;
         &lt;!-- Anyone with one of the listed roles may access this area 
	 	 &lt;role-name&gt;utilisateurs&lt;/role-name&gt;
	 	 &lt;role-name&gt;users&lt;/role-name&gt;
	 	 &lt;role-name&gt;everyone&lt;/role-name&gt;--&gt;
	 	 &lt;role-name&gt;everyone&lt;/role-name&gt;
      &lt;/auth-constraint&gt;
    &lt;/security-constraint&gt;

    &lt;!-- Default login configuration --&gt;
    &lt;login-config&gt;
      &lt;auth-method&gt;BASIC&lt;/auth-method&gt;
      &lt;realm-name&gt;Example Spnego&lt;/realm-name&gt;
    &lt;/login-config&gt;</pre> 
  <br>
  <div style="font-family: 微软雅黑; font-size: 14px;">
   就这样，tomcatspnego就能使用了。这里使用到了Tomcat的目录保护功能，而tomcatspnego应该是对tomcat的验证功能做了修改，增加了域信息的检查。这里连没有配置域服务器ip地址都没有配置，tomcatspnego具体如何实现域信息监测就不太清楚了。
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   <br>
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   接下来，在我们自己的应用身份验证中需要用到tomcatspnego留给我们的标记：
  </div> 
  <pre name="code" class="java">    @RequestMapping(value = "/auth")
    public String ntlmAuth(HttpServletRequest request,HttpServletResponse response,HttpSession session){
    	Principal princ = request.getUserPrincipal();
    	if (isLogined()) {
                return "redirect:index.do";
        }
    	if(princ!=null){
    		session.setAttribute("princpalNameInSession", princ.getName());
    	}
    	return "redirect:index.do";
    }

    @RequestMapping("/login")
    public ModelAndView login(String loginName,String password,HttpServletResponse response,HttpSession session,HttpServletRequest request) {
    	if(!response.isCommitted()){
    		ModelAndView mav = new ModelAndView();
			mav.setViewName("login");
    		User user = null;
    		Config config = Config.getInstance(true);
    		if(notEmpty(loginName) &amp;&amp; notEmpty(password)){//如果用户名和密码都存在,普通登录，或域账号登录
    			user = userService.getEntityByProperty(User.class, "userName", loginName);
    			if(user==null || user.getStatus()!=UserStatus.Active ){
    				mav.addObject("noUserError", "用户名不存在!");
    			}else{
    				if(user.getType()==UserType.Domain &amp;&amp; config.isUseLdapValidate()){//域用户
    					if(domainLoginValidate(loginName,password,request,config)){
    						mav.setViewName("redirect:index.do");//域登录成功
    					}
    				}else{
    					if(localLoginValidate(user,password)){
    						mav.setViewName("redirect:index.do");//本地登录成功
    					}else{
    						mav.addObject("passwordError", "密码输入错误!");
    					}
    				}
    			}
    		}else if(domainLoginValidateByNtlm(request)){//存在域Ntlm变量，尝试域登录
    			//则根据域变量获取到用户名
    			String princpalName = getPrincpalUserName(request);
    			user = userService.getEntityByProperty(User.class, "userName", princpalName);
    			
    			if(user!=null){
    				log.info("用户:"+loginName+"，通过获取到本机域信息直接登录成功!");
    			}else{
    				mav.addObject("error","已经检测到您为域用户，但是在系统中没有查询到您的用户信息");
    			}
    		}else{
    			//用户名和密码以及域变量都不存在，重定向到登录页面
    			mav.setViewName("redirect:loginUI.do");
    		}
    		//然后获取到user信息，并加载权限信息，设置“已登录”标志
    		if(user!=null){
    			prepareFunctionPoint(session,user);
    		}
    		return mav;
    	}
    	return null;
    }

    @RequestMapping("/index")
    public String index(HttpServletRequest request,HttpServletResponse response,HttpSession session){
    	if(!response.isCommitted()){
			if (isLogined()) {
				return "main";
			} 
			Config config = Config.getInstance(true);
			if(!config.isUseLdapValidate()){
				return "redirect:loginUI.do";
			}
			// 则尝试根据域变量获取到用户信息;
			User user = getUserFromDbByDomainUserName(request);
			if (user != null) {
				prepareFunctionPoint(session, user);
				return "main";
			}
			return "redirect:loginUI.do";
    	}
    	return null;
	}</pre> 
  <br>
  <div style="font-family: 微软雅黑; font-size: 14px;">
   说明：
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   这里把应用的首页设置为/auth.do，以便于当用户直接访问/的时候直接跳转到/auth.do；
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   在进入ntlmAuth方法之前，系统已经利用tomcatspnego来尝试域验证；
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   在auth方法中尝试获取tomcatspnego给我们留下来的变量，并保存起来；
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   从定向到index.do，index中尝试通过域变量来获取用户信息（获取成功表示域登陆成功）；
  </div> 
  <div style="font-family: 微软雅黑; font-size: 14px;">
   login方法正常接收用户名和密码，可以进行本地账号验证和域账号验证。
  </div> 
  <br> 
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><br></span></span></span></span></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><br></span></span></span></span></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><br></span></span></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><span style="font-family: 微软雅黑; font-size: 14px;"><br></span></span>
 </div> 
 <div style="font-size: 14px;">
  <span style="font-family: 微软雅黑; font-size: 14px;"><br></span>
 </div> 
</div> 
<p></p>