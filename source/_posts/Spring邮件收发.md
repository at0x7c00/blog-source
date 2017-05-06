---
title: Spring邮件收发
date: 2011-03-30 19:46
categories: 
- Java
tags: 
- Spring
---
```java
package cn.chinacti.crm.util;

import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;

import cn.chinacti.crm.entity.Mail;

/**
 * 邮件收发引擎
 * @author huqiao
 */
public class MailEngine {

	private MailEngine(){
		
	};
	
	/**
	 * 获取邮件收发引擎实例
	 * @return
	 */
	public static MailEngine getInstance(){
		return new MailEngine();
	}
    /**
     * 发送邮件
	 * @param hostAddress  smtp.163.com
	 * @param password
	 * @param userName
	 * @param from
	 * @param to
	 * @param subject
	 * @param body
	 * @throws MessagingException
	 */
	public  void sendEmail(Mail mail) throws MessagingException {   
	        JavaMailSenderImpl javaMail = new JavaMailSenderImpl();
	        //服务器地址
	        javaMail.setHost("smtp.gmail.com");   
	        //登录密码
	        javaMail.setPassword("*********");   
	        //登录用户名
	        javaMail.setUsername("*********");
	        //设置验证
	        Properties prop = new Properties();   
	        prop.setProperty("mail.smtp.auth", "true");  
	        prop.setProperty("mail.debug", "true");
	        prop.setProperty("mail.smtp.socketFactory.port", "465");
	        prop.setProperty("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory"); 
            prop.setProperty("mail.smtp.socketFactory.fallback", "false"); 
            prop.setProperty("mail.smtp.port", "465");
            prop.setProperty("mail.smtp.socketFactory.port", "465"); 

	      	        
	        javaMail.setJavaMailProperties(prop);
	        javaMail.setProtocol("smtp");
	        //生成邮件体
	        String receivers=mail.getTo();
	        String[] tos=receivers.split(";");
	        List&lt;MimeMessage&gt; messageList=new ArrayList&lt;MimeMessage&gt;();
	        for(String to:tos){
	            MimeMessage message = javaMail.createMimeMessage();   
	            MimeMessageHelper messageHelp = new MimeMessageHelper(message,true,"UTF-8");
	            //邮件来源
		        messageHelp.setFrom(mail.getFrom());   
		        //邮件主题
		        messageHelp.setSubject(mail.getSubject());   
		        //邮件内容		          
		        messageHelp.setText(mail.getBody(), true);
		    	messageHelp.setTo(to); 
		    	messageList.add(message);
	        }
	        
	      
	      	        //邮件发送地址
	        	        MimeMessage[] msgs=new MimeMessage[messageList.size()];
	        for(int i=0;i&lt;msgs.length;i++){
	        	msgs[i]=messageList.get(i);
	        }
	        javaMail.send(msgs);
	    }
}
```

```java
package cn.chinacti.crm.entity;

/**
 * 邮件
 * @author huqiao 2011-03-30
 */
public class Mail {

	private String hostAddress;//邮件服务器地址
	private String userName;//登录用户名
	private String password;//登录密码
	private String from;//发件人
	private String to;//收件人
	private String subject;//主题
	private String body;//内容
	
	public String getHostAddress() {
		return hostAddress;
	}
	public void setHostAddress(String hostAddress) {
		this.hostAddress = hostAddress;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
	public String getFrom() {
		return from;
	}
	public void setFrom(String from) {
		this.from = from;
	}
	public String getTo() {
		return to;
	}
	public void setTo(String to) {
		this.to = to;
	}
	public String getSubject() {
		return subject;
	}
	public void setSubject(String subject) {
		this.subject = subject;
	}
	public String getBody() {
		return body;
	}
	public void setBody(String body) {
		this.body = body;
	}
}
```

注意：我原先使用163的时，发送单个邮件还要靠运气。群发的完全成功率更是0%。有时候能发，说明不是我配置的问题，我认为还是163的问题，我这个账号功能有限制。试着注册了一个gmail的账号，群发单发都能100%成功。 爽！