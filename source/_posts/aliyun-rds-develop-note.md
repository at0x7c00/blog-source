---
title: 阿里云RDS接口开发笔记
date: 2016-07-18 22:36
categories: 
- Java
tags: 
- RDS
---

这里是RDS的接口文档:阿里云开发者社区，但文档里说的那些什么签名啊什么的其实是有误导的，咱不需要关心。感觉RDS的接口最开始就是这种HTTP的，我在看这个文档的时候就很奇怪，为什么没有封装成SDK。看签名部分的时候真的很蛋疼。

其实是有SDK的，并且在文档中有引用到：aliyun-openapi-java-sdk。阿里云几乎所有的api都可以在这里找到，当然包括了RDS。

但就单个模块而言，在github中的SDK没有比较好的javadoc说明，每个接口应该传什么样的参数还得参考前面的API文档。

下面是我根据SDK写的工具类，时间仓促，仅仅实现了数据库的添加和删除操作，其他功能可以调API来具体实现。仅供参考。

```java
package com.aliyuncs.rds;

import java.util.List;

import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.exceptions.ServerException;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;
import com.aliyuncs.rds.exception.RequestTimeoutException;
import com.aliyuncs.rds.model.v20140815.CreateDatabaseRequest;
import com.aliyuncs.rds.model.v20140815.DeleteDatabaseRequest;
import com.aliyuncs.rds.model.v20140815.DescribeDatabasesRequest;
import com.aliyuncs.rds.model.v20140815.DescribeDatabasesResponse;
import com.aliyuncs.rds.model.v20140815.DescribeDatabasesResponse.Database;
import com.aliyuncs.rds.model.v20140815.GrantAccountPrivilegeRequest;

public class RDSUtil {

    public static final String DB_STATUS_CREATING = "Creating";
    public static final String DB_STATUS_RUNNING = "Running";
    public static final String DB_STATUS_DELETING = "Deleting";

    private String regionId;
    private String accessKeyId;
    private String accessKeySecret;
    private String dbInstanceId;
    private IAcsClient client;

    public RDSUtil(String regionId, String accessKeyId, String accessKeySecret,String dbInstanceId) {
        this.regionId = regionId;
        this.accessKeyId = accessKeyId;
        this.accessKeySecret = accessKeySecret;
        this.dbInstanceId = dbInstanceId;

        IClientProfile profile = DefaultProfile.getProfile(this.regionId, this.accessKeyId,this.accessKeySecret);
        client = new DefaultAcsClient(profile);
    }


    /**
     * 创建数据库
     */
    public void createDatabase(String dbName,long timeout)throws RequestTimeoutException, ServerException, ClientException{

        CreateDatabaseRequest request = new CreateDatabaseRequest();
        request.setDBInstanceId(dbInstanceId);
        request.setDBName(dbName);
        request.setCharacterSetName("utf8");

        client.getAcsResponse(request);

        boolean created = false;
        int time = 0;
        while(!created){
            try {
                Thread.sleep(1000);
                time++;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            List<Database> list = findDatabaseByName(dbName,"Running");
            created = list.size()>0;

            if(created){
                break;
            }

            if(time>timeout){
                throw new RequestTimeoutException("no database named '" + dbName + "' found after create database request has bean send " + timeout +" seconds ago.");
            }
        }
    }


    /**
     * 授权
     */
    public void grantPrivilege(String dbName,String accountName) throws ServerException, ClientException{

        GrantAccountPrivilegeRequest grantRequest = new GrantAccountPrivilegeRequest();
        grantRequest.setDBInstanceId(dbInstanceId);
        grantRequest.setDBName(dbName);
        grantRequest.setAccountPrivilege("ReadWrite");
        grantRequest.setAccountName(accountName);

        client.getAcsResponse(grantRequest);

    }


    /**
     * 删除数据库
     * @throws RequestTimeoutException 
     */
    public void dropDatabase(String dbName,long timeout) throws ServerException, ClientException, RequestTimeoutException{
        DeleteDatabaseRequest delDbRequest = new DeleteDatabaseRequest();
        delDbRequest.setDBInstanceId(dbInstanceId);
        delDbRequest.setDBName(dbName);

        client.getAcsResponse(delDbRequest);

        //等待数据库的运行中状态消失
        timeout -= waitUntil(dbName,DB_STATUS_RUNNING,false,timeout);
        //等待数据库的删除中状态消失
        waitUntil(dbName,DB_STATUS_DELETING,false,timeout);

    }

    /**
     * 安装数据库名称和状态查询
     */
    public List<Database> findDatabaseByName(String dbName,String status) throws ServerException, ClientException{
        DescribeDatabasesRequest describeDbRequest = new DescribeDatabasesRequest();
        describeDbRequest.setDBInstanceId(dbInstanceId);
        describeDbRequest.setDBName(dbName);
        describeDbRequest.setDBStatus(status);
        DescribeDatabasesResponse descDBResponse = client.getAcsResponse(describeDbRequest);
        List<Database> list = descDBResponse.getDatabases();
        return list;
    }


    /**
     * 等待数据库到（不到）指定的状态
     */
    public long waitUntil(String dbName,String status,boolean existsed,long timeout) throws ServerException, ClientException, RequestTimeoutException{

        List<Database> list = findDatabaseByName(dbName,status);

        int time = 0;
        while((existsed && list.size()==0) || (!existsed && list.size()>0) ){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list = findDatabaseByName(dbName,status);
            time++;
            if(time>timeout){
                throw new RequestTimeoutException("");
            }

        }

        return time;

    }

}
```