# [实时获取局域网](http://blog.chinaunix.net/uid-31401119-id-5781310.html)
```sh
全都使用同一账号登录
Office365中下载Data GateWay，PowerBI Desktop，PowerBI for Mobile
在局域网服务器上安装Data GateWay，然后在Office365的web中设置-管理网关连接局域网服务器
使用PowerBI Desktop连接好本地数据源Publish
在Office365中可以看到数据集，刷新频率修改为15分钟，使用数据集创建报表，调整好页面大小380*380左右
每个报表包含一个图形即可，然后将报表固定到活动页面，这样手机端才可正常显示。

```

# PowerBI CLI
```sh
用node命令控制台新建工作区
npm install powerbi-cli -g

powerbi config -c ColloctionName -k key1 -b https://api.powerbi.cn
powerbi get-workspaces
powerbi create-workspace

//这两句好像不用？
azure login 是登陆Global Azure
azure login -e AzureChinaCloud 是登陆 Mooncake Azure 
```

# [Mac PowerBI CLI BugFix First](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/power-bi-embedded/aog-power-bi-embedded-cli-cannot-be-used-in-mac-os)
```sh
cd /usr/local/lib/node_modules/powerbi-cli/bin

for i in `ls`; 
do mv -f $i `echo $i | sed 's/^.../powerbi/'`; 
done;

mv powerbi cli
chmod +x powerbi-*
```

# [Git](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/power-bi-embedded/aog-power-bi-embedded-sample-configuration-steps)
```sh
https://github.com/kustbilla/Mooncake_PowerBI_Embedded.git
将https://management.core.windows.net/ 替换为 https://management.core.windows.net/  
App.config把对应信息填好，上传用6，如果是连的WAN数据库，上传成功后用8看一下DatasetId，之后用7设置字符串Data Source=www.azure.cn;Initial Catalog=dbname;
传完了之后把Web.config对应信息填好看结果
最后发布到Azure上，要先创建移动服务

https://github.com/Azure-Samples/power-bi-embedded-integrate-report-into-web-app.git
有两个china的config拷到对应的文件夹下
```

# Cloud.config
```sh
<?xml version="1.0" encoding="utf-8" ?>
<appSettings>
	<add key="clientId" value="ea0616ba-638b-4df5-95b9-636659ae5121"/>
	<!-- The Power BI API Endpoint -->
  <!--<add key="powerBiApiEndpoint" value="https://api.powerbi.com" />-->
  <add key="powerBiApiEndpoint" value="https://api.powerbi.cn" />
	<!-- The Azure Resource Manager API Endpoint-->
  <!--<add key="azureApiEndpoint" value="https://management.azure.com" />-->
  <add key="azureApiEndpoint" value="https://management.chinacloudapi.cn" />
	<!-- The Azure Resource Manager Resource -->
  <!--<add key="armResource" value="https://management.core.windows.net/"/>-->
  <add key="armResource" value="https://management.core.chinacloudapi.cn"/>
	<!-- The Azure API version -->
	<add key="apiVersion" value="?api-version=2016-01-29"/>
	<!-- Authentication Url-->
  <!--<add key="AuthenticationUrl" value="https://login.windows.net"/>-->
  <add key="AuthenticationUrl" value="https://login.chinacloudapi.cn"/>
	<!-- tenants Url -->
	<add key="tenantsUrl" value="https://management.azure.com/tenants?api-version=2016-01-29" />
	<!-- defaultRegion -->
	<add key="defaultRegion" value="southcentralus"/>
	<!-- The Default Embed URL -->
	<add key="defaultEmbedUrl" value="https://embedded.powerbi.com/appTokenReportEmbed" />
</appSettings>
```



