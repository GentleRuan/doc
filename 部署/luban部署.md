# 服务器文件路径

```
 /data/node-web/luban-h5/
```

# 1、启动服务端

进入后端目录
```
/data/node-web/luban-h5/back-end/h5-api
```
用pm2启动
```
pm2 startOrReload pm2.json --env local
```
服务端端口：1337
 
# 2、nginx安装
 
 ```
 yum install nginx
 
 ```
 nginx 配置安装目录：
 /etc/nginx
 
进入nginx目录，在目录下创建一个文件luban.wangxiaobao.com.conf
```
 cd /etc/nginx
 vim conf.d/luban.wangxiaobao.com.conf
```
内容

```
server {                                                                                                                        
    listen       80;                                                                                                            
    server_name luban.wangxiaobao.com;                                                                                          
    #charset koi8-r;                                                                    
    #access_log  logs/host.access.log  main;                                                                         
    proxy_redirect off;                                             
    # 静态页配置
    location / {                                                                                                                
       root /data/node-web/luban-h5/front-end/h5/dist;                                                                          
    }                                                                   
    # 服务端配置
    location ~* /(upload|content-manager|users-permissions|works|admin|psd-files|workforms|third-libs|engine-assets) {          
        proxy_pass http://localhost:1337;                                                                                       
        proxy_set_header Host $host;                                                                                            
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;                                                            
    }                                                                                                                           
}                                                                                                                               
```

配置完后直接用 luban.wangxiaobao.com 访问

ps: nodejs版本要求：v10.15.3