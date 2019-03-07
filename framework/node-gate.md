## Node网关使用文档

- ### 配置节说明

以AICall客的测试环境配置举例

```json
{
  "name": "AICall客管理后台",
  "port":3000,
  "redis": {
	"host": "172.17.17.110",
	"port": 6379,
	"db": 10,
	"prefix": "proxy_aicaller_admin_",
	"password": "dichantest"
  },
  "oauth": {
	"clientId": "C9F553E0E518FA802534",
	"clientSecret": "$2a$10$iwafTJZkF.KRyalIpw.WLuF0an1umvw99CFf.wCZdXqIhaPO88XBS",
	"accessTokenUri": "https://accounts-test-e.source3g.com/oauth/token",
	"authorizationUri": "https://accounts-test-e.source3g.com/oauth/authorize",
	"redirectUri": "https://wfaa-test-e.source3g.com/internal-api/loginAuth/callback",
  	"logout":"https://accounts-test-e.source3g.com/logout",
	"scopes": ["READ"]
  },
  "login":{
	"verify": "http://api-i-test-e.source3g.com/authority/user/verify",
	"apiLogin": "https://accounts-test-e.source3g.com/api/user",
	"incharge": "http://api-i-test-e.source3g.com/authority/project/listAllByUserIdAndModuleId"
  },
  "directory": "/internal-api/",
  "anonymous":["/internal-api/.*"],
  "user": "multi",
  "defaultPage": "https://wfaa-test-e.source3g.com",
  "landingPage": "http://hermes-test.source3g.com/hermes-api/index.shtml",
  "proxy":{
	"https://api-test-e.source3g.com/aicallk": {
	  "/task": "/task",
	  "/serviceProvider": "/serviceProvider",
	  "/taskRecord": "/taskRecord",
	  "/taskRecordText": "/taskRecordText",
	  "/statistics": "/statistics",
	  "/customer": "/customer",
	  "/project": "/project",
	  "/account": "/account"
	},
	"http://api-i-test-e.source3g.com/authority": {
	  "/user": "/user"
	}
  }
}

```

相关的说明如下

| 配置节      | 说明                                                         | 其他                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name        | Node启动时候显示的项目名称                                   |                                                              |
| port        | Node监听的端口                                               |                                                              |
| redis       | 相关的登录态持久化的Redis配置信息                            |                                                              |
| oauth       | 权限中心配置信息,其中`redirectUri`为认证成功后的项目回调地址 |                                                              |
| login       | 认证成功后Node访问授权中心获取用户信息等                     |                                                              |
| directory   | 自定义的回调地址的目录                                       | 必须保证改目录是可匿名访问                                   |
| anonymous   | 可匿名访问的目录,支持正则表达式                              |                                                              |
| user        | 登录模式,单用户或则多用户.值分别为`single`和`multi`          | 单用户是指一个帐号只允许一个设备登录,后登录的会把先登录的帐号挤掉 |
| defaultPage | 当前项目的访问地址                                           |                                                              |
| landingPage | hermes地址                                                   |                                                              |
| proxy       | 转发规则配置节                                               | 生效规则按照从上倒下,注意别跟public目录下的文件夹重名        |



- ### 运行流程和相关释义

1. 设置静态文件访问目录,通常为Vue项目打包后的文件,该目录是允许匿名访问的

   ```js
   app.use(express.static(path.join(__dirname, 'public')));
   ```

2. 拦截除了`public`目录和`anonymous`配置节之外的其他目录的匿名请求,具体实现部分为`routes\oauth.js`

   ```js
   module.exports = function (req, res, next) {
   	let law = false;
   	if(req.session.user){
   		law = true;
   	}
   	else{
   		for(let path of config.anonymous){
   			if(new RegExp('^' + path + '$').test(req.path)){
   				law = true;
   				break;
   			}
   		}
   	}
   	if(law){
   		next();
   	}
   	else{
   		if(req.get('ignore-data') === '1'){
   			res.json(tools.errorJson({code:'-1', msg: '未登录', action: req.path}));
   		}
   		else {
   			res.json(tools.errorJson({code: '-1', msg: '未登录', data: auth.code.getUri(), action: req.path}));
   		}
   	}
   };
   ```

   

3. 根据`proxy`配置节使用`http-proxy-middleware`进行代理转发

   ```js
   for(let key in config.proxy){
   	let option = {
   		target: key,
   		changeOrigin: true,
   		ws: true,
   		pathRewrite: config.proxy[key],
   		onProxyReq: function(proxyReq, req, res, target){
   			let userSession = req.session;
   			if(userSession.user && userSession.user.id) {
   				proxyReq.setHeader('userId', req.session.user.id)
   			}
   			if(userSession.currentProjectId) {
   				proxyReq.setHeader('projectId', req.session.currentProjectId);
   			}
   		},
   		onProxyRes: function (proxyRes, req, res) {
   			// 删除跨域名
   			delete proxyRes.headers['access-control-allow-credentials'];
   			delete proxyRes.headers['access-control-allow-headers'];
   			delete proxyRes.headers['access-control-allow-methods'];
   			delete proxyRes.headers['access-control-allow-origin'];
   			delete proxyRes.headers['access-control-max-age'];
   		},
   		onError: function (err, req, res) {
   			res.json(tools.errorJson({code:'-998', msg: '转发异常', action: req.path}));
   		}
   	};
   	let rule = [];
   	for(let item in config.proxy[key]){
   		rule.push(item);
   	}
   	app.use(rule, proxy(option));
   }
   ```

   

4. 配置回调的URL

   ```js
   app.use(directoryPath + '/loginAuth', loginRouter);
   ```

   