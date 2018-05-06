# OwinSelfHost
```c#
类库项目，Window服务项目，控制台项目（便于调试），添加NuGet  Microsoft.AspNet.WebApi.OwinSelfHost,Microsoft.AspNet.WebApi.SelfHost
```
# OwinDLL
```c#
public class IniControllerType
    {
        public static void Ini(IAppBuilder appBuilder)
        {
            //Type OtherController = typeof(ControllerPro.ClassName);
            HttpConfiguration config = new HttpConfiguration();
            config.Formatters.Remove(config.Formatters.XmlFormatter);
            config.Formatters.JsonFormatter.SerializerSettings.Formatting =
       Newtonsoft.Json.Formatting.Indented;
            config.Formatters.JsonFormatter.SerializerSettings.ReferenceLoopHandling = ReferenceLoopHandling.Ignore;
            config.Formatters.JsonFormatter.SerializerSettings.DateFormatString = "yyyy-MM-dd HH:mm";
            config.Formatters.JsonFormatter.SerializerSettings.NullValueHandling = NullValueHandling.Ignore;
            
            config.Routes.MapHttpRoute(
                    name: "DefaultApi",
                    routeTemplate: "api/{controller}/{id}",
                    defaults: new { id = RouteParameter.Optional }
                    );
            appBuilder.UseWebApi(config);
        }
    }
```

# Console
```c#
Startup.cs
 public class Startup
    {
        public void Configuration(IAppBuilder appBuilder)
        {
            IniControllerType.Ini(appBuilder);
        }
    }

Program.cs
static void Main(string[] args)
        {
            try
            {
                using (WebApp.Start<Startup>(url: "http://192.168.1.2:9527"))
                {
                    Console.WriteLine("ok");
                    Console.ReadLine();
                }

            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
```

# Winservice
```c#
Startup.cs
public class Startup
    {
        public void Configuration(IAppBuilder appBuilder)
        {
            IniControllerType.Ini(appBuilder);
        }
    }

Program.cs
static void Main()
        {
            ServiceBase[] ServicesToRun;
            ServicesToRun = new ServiceBase[]
            {
                new Service1()
            };
            ServiceBase.Run(ServicesToRun);
        }

Service1.cs
public partial class Service1 : ServiceBase
    {
        private IDisposable _server = null;

        public Service1()
        {
            InitializeComponent();
        }

        protected override void OnStart(string[] args)
        {
            try
            {
                //#if (DEBUG)
                //     Debugger.Launch();
                //#endif
                _server = WebApp.Start<Startup>(url: "http://192.168.1.2:9527");
            }
            catch (Exception ex)
            {

            }
        }

        protected override void OnStop()
        {
            try
            {
                if (_server != null)
                {
                    _server.Dispose();
                }
                base.OnStop();
            }
            catch (Exception ex)
            {

            }
        }
    }

回到Service1设计窗口点右键选择-添加安装程序 -生成ProjectInstaller.cs并且自动添加serviceInstaller1和 serviceProcessInstaller1两个组件 
把serviceInstaller1的属性ServiceName改写为你的服务程序名，并把启动模 式设置为AUTOMATIC  
把serviceProcessInstaller1的属性account改写为 LocalSystem  

install.cmd
path = %path%;C:\Windows\Microsoft.NET\Framework\v4.0.30319;
installutil /u WindowsService1.exe
path = %path%;C:\Windows\Microsoft.NET\Framework\v4.0.30319;
installutil WindowsService1.exe

uninstall.cmd
path = %path%;C:\Windows\Microsoft.NET\Framework\v4.0.30319;
installutil /u WindowsService1.exe

安装卸载的两个批处理需要使用管理员身份
```

# ControllerDLL
```c#
Nuget Microsoft.AspNet.WebApi.Core  System.Web.Http
public class MSServiceController : ApiController
    {
         System.ServiceProcess.ServiceController service = new System.ServiceProcess.ServiceController("ORAPS_MW");
        public  string restart()
        {
            string res = "OK";
            try
            {
                if (service.Status == ServiceControllerStatus.Running)
                {
                    service.Stop();
                    service.WaitForStatus(ServiceControllerStatus.Stopped);
                }
                service.Start();
                service.WaitForStatus(ServiceControllerStatus.Running);
                return res;
            }
            catch (Exception ex)
            {
                res = ex.ToString();
                return res;
            }
        }
        // GET api/<controller>
        public string Get()
        //public IEnumerable<string> Get()
        {
            //BodyClass zz = new BodyClass();
            //zz.id = 1;
            //zz.name = "zs";
            //string s = JsonConvert.SerializeObject(zz);
            return restart();//new string[] { "value1", "value2" };
        }

        // GET api/<controller>/5
        public string Get(int id)
        {
            return "value";
        }

        // POST api/<controller>
        public void Post([FromBody]dynamic value)
        {
            //如果不用dynamic直接改为BodyClass也可以

            //Type t = value.GetType();
            JObject o = value;
            BodyClass aa2 = o.ToObject<BodyClass>();

            //这样不行，因为value如果用string来接就什么都接不到
            //BodyClass abc = JsonConvert.DeserializeObject<BodyClass>(value);
        }

        // PUT api/<controller>/5
        public void Put(int id, [FromBody]string value)
        {
        }

        // DELETE api/<controller>/5
        public void Delete(int id)
        {
        }
    }
    public class BodyClass
    {
        public int id { get; set; }
        public string name { get; set; }
    }
```