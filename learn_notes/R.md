# CentOS7安装R语言
# yum安装最好
# yum -y install epel-release
# yum -y install R
# 然后再安装rstudio-server就行，下载很慢，建议弄个vsftpd


# 否则要装一大堆，并且最后还会失败
#### [安装zlib](http://www.zlib.net/)
#### [安装bzip2](http://www.bzip.org/downloads.html)
#### [安装liblzma](http://tukaani.org/xz/)
#### [安装pcre](https://ftp.pcre.org/pub/pcre/)
```sh
./configure --enable-utf8 
yum remove pcre 默认安装的可能没有启用uft8，重新安装，顺便安装个高级点的
```
#### [安装libcurl](https://curl.haxx.se/libcurl/)
#### [下载R](https://www.r-project.org/)
```sh
yum -y install readline-devel    
./configure --with-x=no    如果没安装图形界面需要此参数
不加参数 --enable-R-shlib  rstudio-server安装失败，加此参数编译失败，未找到解决方案
make clean
make uninstall
```

# [安装RStudio](https://www.rstudio.com/)
```sh
我安装的是Server版
yum -y install --nogpgcheck rstudio-server-rhel-1.0.136-x86_64.rpm
rstudio-server verify-installation
默认端口是8787
```

# [Ubuntu R Studio Server](https://www.rstudio.com/products/rstudio/download-server/)
```sh
sudo vi /etc/apt/sources.list
deb https://mirrors.tuna.tsinghua.edu.cn/CRAN//bin/linux/ubuntu artful/

sudo apt-get update
sudo apt-get install -y r-base
sudo apt-get install -y gdebi-core
wget https://download2.rstudio.org/rstudio-server-1.1.423-amd64.deb
sudo gdebi -y rstudio-server-1.1.423-amd64.deb
```

# 多人登录
```sh
方法1:
sudo groupadd hadoop 
sudo useradd hadoop -g hadoop;
sudo passwd hadoop 
sudo adduser hadoop sudo
sudo mkdir /home/hadoop 
sudo chown -R hadoop:hadoop /home/hadoop

方法2:
useradd username -u uid -p password
千万记得将uid设定为大于100的数字，大于500更好。
touch /etc/rstudio/rserver.conf
touch /etc/rstudio/rsession.conf

rserver.conf文件并添加以下代码：
auth-required-user-group=rstudio_users
将“rstudio_users”命名为任何你想要的群组名字。
将刚才新建立的用户名添加到该用户组：
groupadd rstudio_users
usermod -g rstudio_users -G rstudio_users username
rstudio-server restart

```

# 例子
```sh
install.packages("vcd")
install.packages("vioplot")
install.packages("plotrix")

x <- (0:20) * pi  / 12
y <- cos(x)
plot(x,y);

点图
dotchart(x, labels = NULL, groups = NULL, gdata = NULL,
         cex = par("cex"), pt.cex = cex,
         pch = 21, gpch = 21, bg = par("bg"),
         color = par("fg"), gcolor = par("fg"), lcolor = "gray",
         xlim = range(x[is.finite(x)]),
         main = NULL, xlab = NULL, ylab = NULL, ...)
x是数据来源，也就是要作图的数据；labels 是数据标签，groups分组或分类方式，gdata分组的值，cex字体大小，pch是作图线条类型，bg背景，color颜色，xlim横坐标范围，main是图形标题，xlab横坐标标签，相应的ylab是纵坐标。

eg1:dotchart(mtcars$mpg,labels = row.names(mtcars),cex = .7,
         main = "Gas Mileage for Car Models",
         xlab = "Miles Per gallon")

eg2:x <- mtcars[order(mtcars$mpg),]        #按照mpg排序
x$cyl <-factor(x$cyl)      #将cyl变成因子数据结构类型
x$color[x$cyl==4] <-"red"   #新建一个color变量，油缸数cyl不同，颜色不同
x$color[x$cyl==6] <-"blue"
x$color[x$cyl==8] <-"darkgreen"
dotchart(x$mpg,        #数据对象
         labels = row.names(x),     #标签
         cex = .7,#字体大小
         groups = x$cyl,      #按照cyl分组
         gcolor = "black",    #分组颜色
         color = x$color,     #数据点颜色
         pch = 19,#点类型
         main = "Gas Mileage for car modes \n grouped by cylinder",    #标题
         xlab = "miles per gallon")     #x轴标签



条形图
barplot(height, ...)

eg1:library(vcd)
counts <- table(Arthritis$Improved)    #引入vcd包只是想要Arthritis中的数据
barplot(counts,main = "bar plot",xlab = "improved",ylab = "counts")

eg2:barplot(counts,main = " horizontal bar plot",
xlab = "frequency",
ylab = "improved",
horiz = TRUE)#horizon 值默认是FALSE，为TRUE的时候表示图形变为水平的

eg3:library(vcd)
counts <- table(Arthritis$Improved,Arthritis$Treatment)
counts

eg4:barplot(counts,main = " stacked bar plot",xlab = "treated",ylab = "frequency",
        col = c("red","yellow","green"),       #设置颜色
        legend = rownames(counts))       #设置图例

eg5:barplot(counts,main = " grouped bar plot",xlab = "treated",ylab = "frequency",
        col = c("red","yellow","green"),
        legend = rownames(counts),beside = TRUE)



荆棘图
荆棘图是对堆砌条形图的扩展，每个条形图高度都是1，因此高度就表示其比例
eg1:
library(vcd)
attach(Arthritis)
counts <- table (Treatment,Improved)
spine(counts,main = "Spinogram Example")
detach(Arthritis)



直方图
hist(x, ...)
eg1:
par (mfrow = c(2,2))        #设置四幅图片一起显示
hist(mtcars$mpg)       #基本直方图

hist(mtcars$mpg,
     breaks = 12,       #指定组数
     col= "red",        #指定颜色
     xlab = "Miles per Gallon",
     main = "colored histogram with 12 bins")

hist(mtcars$mpg,
     freq = FALSE,      #表示不按照频数绘图
     breaks = 12,
     col = "red",
     xlab = "Miles per Gallon",
     main = "Histogram,rug plot,density curve")
rug(jitter(mtcars$mpg))        #添加轴须图
lines(density(mtcars$mpg),col= "blue",lwd=2)       #添加密度曲线

x <-mtcars$mpg
h <-hist(x,breaks = 12,
         col = "red",
         xlab = "Miles per Gallon",
         main = "Histogram with normal and box")
xfit <- seq(min(x),max(x),length=40)
yfit <-dnorm(xfit,mean = mean(x),sd=sd(x))
yfit <- yfit *diff(h$mids[1:2])*length(x)
lines(xfit,yfit,col="blue",lwd=2)       #添加正太分布密度曲线
box()



饼图
pie(x, labels = names(x), edges = 200, radius = 0.8,
    clockwise = FALSE, init.angle = if(clockwise) 90 else 0,
    density = NULL, angle = 45, col = NULL, border = NULL,
    lty = NULL, main = NULL, ...)

eg1:
par(mfrow = c(2,2))
slices <- c(10,12,4,16,8)       #数据
lbls <- c("US","UK","Australis","Germany","France")         #标签数据
pie(slices,lbls)        #基本饼图

pct <- round(slices/sum(slices)*100)        #数据比例
lbls2 <- paste(lbls," ",pct ,"%",sep = "")
pie(slices,labels = lbls2,col = rainbow(length(lbls2)),         #rainbow是一个彩虹色调色板
    main = "Pie Chart with Percentages")

library(plotrix)
pie3D(slices,labels=lbls,explode=0.1,main="3D pie chart")       #三维饼图

mytable <- table (state.region)
lbls3 <- paste(names(mytable),"\n",mytable,sep = "")
pie(mytable,labels = lbls3,
    main = "pie  chart from a table \n (with sample sizes")


扇形图
library(plotrix)
slices<-c(10,12,4,16,8)
lbls<-c("US","UK","Australia","Germany","France")#国家名称，扇区的字符型向量
fan.plot(slices,labels = lbls,main = "Fan Plot")



箱线图
boxplot(x, ...)

eg1:
boxplot(mtcars$mpg,main="Box plot",ylab ="Miles per Gallon")       #标准箱线图

boxplot(mpg cyl,data= mtcars,
       main="car milesge data",
       xlab= "Number of cylinders",
       ylab= "Miles per Gallon")

boxplot(mpg cyl,data= mtcars,
        notch=TRUE,         #含有凹槽的箱线图
        varwidth = TRUE,        #宽度和样本大小成正比
        col= "red",
        main="car milesge data",
        xlab= "Number of cylinders",
        ylab= "Miles per Gallon")

mtcars$cyl.f<- factor(mtcars$cyl,         #转换成因子结构
                      levels= c(4,6,8),
                      labels = c("4","6","8"))
mtcars$am.f <- factor(mtcars$am,levels = c(0,1),
                      labels = c("auto","standard"))
boxplot(mpgam.f*cyl.f,       #分组的箱线图
        data = mtcars,
        varwidth=TRUE,
        col= c("gold","darkgreen"),
        main= "MPG Distribution by Auto Type",
        xlab="Auto Type",
        ylxb="Miles per Gallon")



小提琴图
小提琴图是箱线图和密度图的结合。使用vioplot包中的vioplot函数进行绘图。
vioplot( x, ..., range=1.5, h, ylim, names, horizontal=FALSE, 
  col="magenta", border="black", lty=1, lwd=1, rectCol="black", 
  colMed="white", pchMed=19, at, add=FALSE, wex=1, 
  drawRect=TRUE)

eg1:library(vioplot)
x1 <- mtcars$mpg[mtcars$cyl==4]
x2 <- mtcars$mpg[mtcars$cyl==6]
x3 <- mtcars$mpg[mtcars$cyl==8]
vioplot(x1,x2,x3,names= c("4 cyl","6 cyl","8 cyl"),col = "gold")
title(main="Violin plots of Miles Per Gallon",xlab = "number of cylinders",ylab = "Miles per gallon")
白点是中位数，中间细线表示须，粗线对应上下四分位点，外部形状是其分布核密度。




1.画出函数f(x)=x^3+2x^2+x+1在区间[1,10]上的图形，曲线颜色设置为红色。 
x<-seq(1,10,by=0.1) 
y=x^3+2*x^2+x+1 
plot(x,y,"l",col="red")

2.将窗口分割成2×1的窗格，在第一个、第二个窗口中分别绘制出正弦、余弦函数的图像，并画出x轴。 
x<-seq(0,2*pi,by=pi/20) 
y1<-sin(x) 
y2<-cos(x)  
par(mfrow=c(2,1))  
plot(x,y1,"l",xlim=c(0,7),ylim=c(-2,2)) 
lines(c(0,3*pi),c(0,0))  
plot(x,y2,"l",xlim=c(0,7),ylim=c(-2,2)) 
lines(c(0,3*pi),c(0,0)) 

3.在同一张图上画出函数y1=2x,y2=x^2的曲线并利用legend函数对曲线加标注“y1=2x,y2=x^2”，并加上标题  “2x和x^2的曲线”。  
x<-seq(0,10,by=0.5) 
y1=2*x 
y2=x^2  
plot(x,y1,"o",main="2*x和x^2 的曲线",pch=8) 
lines(x,y2,"o",pch=24,col="red")  
legend("topright",legend=c("2*x","x^2"),col=c("black","red"),pch=c(8,24),lty=1)

4. 将屏幕分割为四块，并分别画出y=sin(x)；z=cos(x)；a=sin(x)*cos(x)；b=sin(x)/cos(x)。 
par(mfrow=c(2,2))  
x<-seq(0,2*pi,by=pi/20) 
y<-sin(x) 
z<-cos(x)  
a<-sin(x)*cos(x) 
b<-sin(x)/cos(x) 
plot(x,y,"l") 
plot(x,z,"l") 
plot(x,a,"l") 
plot(x,b,"l")

5.画出下列函数的三维透视图z=(x-2)^2+(y-1.2)^2  
x<-y<-seq(-2,6,by=0.1) 
f<-function(x,y){ 
    z<-(x-2)^2+(y-1.2)^2 
    }  
    z<-outer(x,y,f)  
    persp(x,y,z,col="lightblue")
```

# Windows R Studio 连接 SQL Server
```sh
首先添加ODBC，名为test
library(RODBC)
odbcDataSources()
conn=odbcConnect('test',uid='sa',pwd='123456')
result=sqlQuery(conn,'select * from tbname')
result
odbcClose(conn)
```

# Ubuntu R Server连接SQL Server
```sh
dpkg -s unixODBC  
dpkg -s unixODBC-dev

sudo apt-get install -y unixodbc
sudo apt-get install -y unixodbc-dev

sudo apt-get install -y msodbcsql

/etc/odbcinst.ini
[ODBC Driver 13 for SQL Server]
Description=Microsoft ODBC Driver 13 for SQL Server
Driver=/opt/microsoft/msodbcsql/lib64/libmsodbcsql-13.1.so.9.2
UsageCount=1

/etc/odbc.ini
[testsql]
Driver=ODBC Driver 13 for SQL Server
Server=192.168.1.2
Database=MS_User
Port=1433

odbcinst -j
odbcinst -q -s
odbcinst -q -d

isql testsql sa XXXX

install.packages("RODBC")
不安装unixodbc报错configure: error: "ODBC headers sql.h and sqlext.h not found"

library(RODBC);
odbcDataSources()

library(RODBC);
dbhandle <- odbcDriverConnect('driver={SQL Server};server=192.168.1.2;database=dbname;uid=sa;pwd=123456')
r <- sqlQuery(dbhandle, 'select * from tbname')
print(r)
odbcClose(dbhandle)
```

# C# And R（Nuget R.NET）
```c#
ConsolR项目
using RDotNet;
using System;
using System.Linq;

namespace ConsoleR
{
    class Program
    {
        static void Main(string[] args)
        {
            test1();
            test2();
            test3();
        }
        static void test1()
        {
            RDotNet.REngine.SetEnvironmentVariables();//设置R语言运行环境//<span style="font-family: Arial, Helvetica, sans-serif;">REngine.SetEnvironmentVariables(null, null);</span 
            var v = RDotNet.REngine.GetInstance();//得到R语言实例//REngine engine = REngine.GetInstance(null, true, null, null);  
            v.Initialize();//初始化
            double[] cpiDou = { 99.89, 100.01, 100.2 };
            double[] gdpDou = { 351564.8167, 230025.5833, 108486.35 };
            double[] preDou = { 77498269.32, 140859820.9, 53267665.13 };
            double[] incDou = { 15506102, 14774593, 12597942 };
            RDotNet.NumericVector x1 = v.CreateNumericVector(cpiDou);
            v.SetSymbol("x1", x1);
            RDotNet.NumericVector x2 = v.CreateNumericVector(gdpDou);
            v.SetSymbol("x2", x2);
            RDotNet.NumericVector x3 = v.CreateNumericVector(preDou);
            v.SetSymbol("x3", x3);
            RDotNet.NumericVector x4 = v.CreateNumericVector(incDou);
            v.SetSymbol("x4", x4);
            v.Evaluate("y1 = log(x1)");
            v.Evaluate("y2 = log(x2)");
            v.Evaluate("y3 = log(x3)");
            v.Evaluate("y4 = log(x4)");
            v.Evaluate("m1=lm(y1~y2+y3+y4)").AsList();
            RDotNet.NumericVector dd = v.Evaluate("summary(m1)$coefficients").AsNumeric();//dd即表示返回值C#操作
            object o = dd;
        }
        static void test2()
        {
            REngine.SetEnvironmentVariables();
            // There are several options to initialize the engine, but by default the following suffice:
            REngine engine = REngine.GetInstance();

            // .NET Framework array to R vector.
            NumericVector group1 = engine.CreateNumericVector(new double[] { 30.02, 29.99, 30.11, 29.97, 30.01, 29.99 });
            engine.SetSymbol("group1", group1);
            // Direct parsing from R script.
            NumericVector group2 = engine.Evaluate("group2 <- c(29.89, 29.93, 29.72, 29.98, 30.02, 29.98)").AsNumeric();

            // Test difference of mean and get the P-value.
            GenericVector testResult = engine.Evaluate("t.test(group1, group2)").AsList();
            double p = testResult["p.value"].AsNumeric().First();

            Console.WriteLine("Group1: [{0}]", string.Join(", ", group1));
            Console.WriteLine("Group2: [{0}]", string.Join(", ", group2));
            Console.WriteLine("P-value = {0:0.000}", p);
            Console.ReadKey();
            // you should always dispose of the REngine properly.
            // After disposing of the engine, you cannot reinitialize nor reuse it
            engine.Dispose();
        }
        static void test3()
        {
            REngine.SetEnvironmentVariables(); // <-- May be omitted; the next line would call it.
            REngine engine = REngine.GetInstance();
            // A somewhat contrived but customary Hello World:
            CharacterVector charVec = engine.CreateCharacterVector(new[] { "Hello, R world!, .NET speaking" });
            engine.SetSymbol("greetings", charVec);
            engine.Evaluate("str(greetings)"); // print out in the console
            string[] a = engine.Evaluate("'Hi there .NET, from the R engine'").AsCharacter().ToArray();
            Console.WriteLine("R answered: '{0}'", a[0]);
            Console.WriteLine("Press any key to exit the program");
            Console.ReadKey();
            engine.Dispose();
        }
    }
   
}








WinFormR项目
using System;
using System.Drawing;
using System.Windows.Forms;

namespace WinFormR
{
    using RDotNet;
    public partial class Main : Form
    {
        public Main()
        {
            InitializeComponent();
        }
        REngine engine = null;

        string Rcode = "";
        private void btnPlot_Click(object sender, EventArgs e)
        {
            try
            {
                if (this.txtRcode.Text == "")
                {
                    //需要安装32位R语言，否则此种方式均报错
                    Rcode = @"library('scatterplot3d')
       z <- seq(-10, 10, 0.01) 
       x <- cos(z)
       y <- sin(z) 
       scatterplot3d(x, y, z, highlight.3d=TRUE, col.axis='blue', col.grid='lightblue',main='3d绘图',pch=20)
       ";

                    //Rcode = @"source('c:\Users\user1\source\repos\NetR1\RProject1\Script.R')"; --err
                    //Rcode = "source('c:/Users/user1/source/repos/NetR1/RProject1/Script.R')"; --ok
                }
                else
                {
                    Rcode = this.txtRcode.Text;
                }

                //R.3.2.4
                //System.Environment.SetEnvironmentVariable("Path", panth2);
                //REngine.SetEnvironmentVariables();
                engine = REngine.GetInstance();
                engine.Initialize();
                //图片加入GUID，防止重名（还有一种就是先删除后保存）
                string rnd = System.Guid.NewGuid().ToString().Replace("-", "");
                string filename = "i" + rnd + "__Rimage.png";
                engine.Evaluate(string.Format("png(file='{0}',bg ='transparent',width={1},height={2})", filename, this.ptbGraphic.Width, this.ptbGraphic.Height));

                //engine.Evaluate(@"x <- (0:12) * pi / 12
                //    y <- cos(x)
                //    plot(x,y);
                //    ");
                /*
                 * 
x<-seq(1,10,by=0.1) 
y=x^3+2*x^2+x+1 
plot(x,y,"l",col="red")


x<-seq(0,10,by=0.5) 
y1=2*x 
y2=x^2  
plot(x,y1,"o",main="2*x和x^2 的曲线",pch=8) 
lines(x,y2,"o",pch=24,col="red")  
legend("topright",legend=c("2*x","x^2"),col=c("black","red"),pch=c(8,24),lty=1)


par(mfrow=c(2,2))  
x<-seq(0,2*pi,by=pi/20) 
y<-sin(x) 
z<-cos(x)  
a<-sin(x)*cos(x) 
b<-sin(x)/cos(x) 
plot(x,y,"l") 
plot(x,z,"l") 
plot(x,a,"l") 
plot(x,b,"l")




                    */
                //engine.Evaluate(".libPaths('C:/Users/user1/Documents/R/win-library/3.4");

                engine.Evaluate(Rcode);
                engine.Evaluate("dev.off()");
                string path = System.IO.Path.GetFullPath(filename);

                Bitmap image = new Bitmap(path);
                ptbGraphic.Image = image;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }

        }

        private void Main_FormClosing(object sender, FormClosingEventArgs e)
        {
            if (engine != null)
            {
                //clean up
                engine.Dispose();
            }
        }
    }
}
```






# Java And R JRI
```java
install.packages("rJava")
PATH后添加C:\Program Files\R\R-3.3.1\bin\x64;C:\Program Files\R\R-3.3.1\library\rJava\jri;
（可以用%R_HOME%的写法），但是要特别注意“bin\x64”，系统是64就指定x64文件夹，32位就指定i386文件夹，否则会找不到依赖库；
同理rJava\jri下用的dll文件也要与计算机位数一致；

在R的安装目录（\R-3.1.2\library\rJava\jri）下，找到三个jar包：JRI.jar，JRIEngine.jar，REngine.jar。
在idea中新建一个java工程，将三个jar包添加到工程编译路径下。 

1.R
test_no_param <- function (){
  return(100)
}

# 入参为一个list和一个整数，出参为一个list
test_param_list <- function (x, y) {
  x[,2] <- x[,2] * y
  
  return(x)
}

RStudio
c1 <- c('a','b','c') 
c2 <- c(1, 2, 3) 
x <- data.frame(c1,c2) 
y <- test_param_list(x, 10) 




pom.xml

<properties>
	<rjava.version>0.9-7</rjava.version>
</properties>

<dependency>
	<groupId>com.github.lucarosellini.rJava</groupId>
    <artifactId>JRIEngine</artifactId>
    <version>${rjava.version}</version>
</dependency>
<dependency>
    <groupId>com.github.lucarosellini.rJava</groupId>
    <artifactId>REngine</artifactId>
    <version>${rjava.version}</version>
</dependency>
<dependency>
    <groupId>com.github.lucarosellini.rJava</groupId>
    <artifactId>JRI</artifactId>
    <version>${rjava.version}</version>
</dependency>




Test2.java
import java.util.Arrays;

import org.rosuda.JRI.REXP;
import org.rosuda.JRI.RList;
import org.rosuda.JRI.RVector;
import org.rosuda.JRI.Rengine;

/**
 * 使用rJava包，调用R代码中的函数
 * 
 * @author frank
 * @since 2016-11-14
 *
 */
public class Test2 {

    public static void main(String[] args) {
        Test2 test2 = new Test2();
        test2.method2();
    }

    /**
     * 加载R文件，调用函数
     */
    @SuppressWarnings("deprecation")
    public void method2(){
        // R文件全路径
        String filePath = "D:\\1.R";

        // 初始化R解析类
        Rengine engine = new Rengine(null, false, null);
        System.out.println("Rengine created, waiting for R");

        // 等待解析类初始化完毕
        if (!engine.waitForR()) {
            System.out.println("Cannot load R");
            return;
        }
        // 将文件全路径复制给R中的一个变量
        engine.assign("fileName", filePath);
        // 在R中执行文件。执行后，文件中的两个函数加载到R环境中，后续可以直接调用
        engine.eval("source(fileName)");
        System.err.println("R文件执行完毕");

        {
            // 直接调用无参的函数，将结果保存到一个对象中
            REXP rexp = engine.eval("test_no_param()");
            System.err.println(rexp);
            // 已知返回值的类型，故将其转换为double，供其他代码使用
            double d = rexp.asDouble();
            System.err.println(d);
            System.err.println("---------------1");
        }

        {
            // 定义一个数组，与R中c1集合对应
            String[] arr1 = new String[]{"a", "b", "c"};
            // 将数组复制给R中的变量c1。R中变量无需预先定义
            engine.assign("c1", arr1);

            // 定义一个数组，与R中c2集合对应
            double[] arr2 = new double[]{1, 2, 3};
            // 将数组复制给R中的变量c2
            engine.assign("c2", arr2);
            // 将c1 c2连接为一个集合（R中的数据集，类似java的list），赋值给一个变量
            engine.eval("x <- data.frame(c1, c2)");
            // 将一个数值保存到一个变量中
            engine.eval("y <- 10");

            // 入参为list，出参为list。调用R中函数，将结果保存到一个对象中。
            REXP rexp = engine.eval("test_param_list(x, y)");
            System.err.println(rexp);

            // 解析rexp对象，转换数据格式

            // list的标题
            RList list = rexp.asList();
            String[] key = list.keys();
            System.err.println(Arrays.toString(key));
            if(key != null){
                int i = 0;
                while (i < key.length){
                    i++;
                }
            }
            // list的数据
            RVector v =  rexp.asVector();
            for(int i=0; i<v.size(); i++){
                REXP rexpTemp = (REXP) v.get(i);
                if(REXP.INTSXP == rexpTemp.rtype){
                    int[] arr = rexpTemp.asIntArray();
                    System.err.println(Arrays.toString(arr));
                } else if(REXP.STRSXP == rexpTemp.rtype){
                    String[] arr = rexpTemp.asStringArray();
                    System.err.println(Arrays.toString(arr));
                } else if(REXP.REALSXP == rexpTemp.rtype){
                    double[] arr = rexpTemp.asDoubleArray();
                    System.err.println(Arrays.toString(arr));
                }
            }
            System.err.println("---------------2");
        }

        engine.stop();
    }
}
``` 

# Java And R Rserve 
```java
RStudio
install.packages("Rserve")
library("Rserve")
Rserve()

myAdd.R
myAdd <- function (x, y) {
  return(x+y)
}

pom.xml
        <!-- https://mvnrepository.com/artifact/org.rosuda.REngine/Rserve -->
        <dependency>
            <groupId>org.rosuda.REngine</groupId>
            <artifactId>Rserve</artifactId>
            <version>1.8.1</version>
        </dependency>


Temp.java
import org.rosuda.REngine.Rserve.RConnection;  
import org.rosuda.REngine.Rserve.RserveException;  
import org.rosuda.REngine.REXPMismatchException;;  
public class Temp {  
  
    public static void main(String[] args) throws REXPMismatchException {  
        // TODO Auto-generated method stub  
        RConnection connection = null;  
        System.out.println("平均值");  
        try {  
            //创建对象  
            connection = new RConnection();  
            String vetor="c(1,2,3,4)";  
            connection.eval("meanVal<-mean("+vetor+")");  
      
            //System.out.println("the mean of given vector is="+mean);  
            double mean=connection.eval("meanVal").asDouble();  
            System.out.println("the mean of given vector is="+mean);  
            //connection.eval(arg0)  
              
        } catch (RserveException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        System.out.println("执行脚本");  
        try {  
            connection.eval("source('D:/myAdd.R')");//此处路径也可以这样写D:\\\\myAdd.R  
            int num1=20;  
            int num2=10;  
            int sum=connection.eval("myAdd("+num1+","+num2+")").asInteger();  
            System.out.println("the sum="+sum);  
        } catch (RserveException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        connection.close();  
    }  
}  
```