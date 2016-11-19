#前言
当服务器的CPU使用率過高的時候，系统可能呈现出不稳定的状态，而当流量过高的時候，就需要注意是哪一個服务或者是哪一個家伙在尝试窃取我們的资料呢？因此，网络管理方面，有必要时刻清楚，我們主机的流量状态，并视流量來加以限制或者是加大带宽。
目前网络上有一套还蛮好用的程序可以用来监控主机的资料流量，这也是各大服务器常使用的程序，就是MRTG (Multi Router Traffic Grapher)

#那么什么是MRTG
MRTG最早的版本是在1995年春天所推出，以Perl所写成，因此可以跨平台使用，它利用了SNMP送出带有物件识别码（OIDs）的请求给要查询的网络设备，因此设备本身需支持SNMP。MRTG再以所收集到的资料产生HTML档案并以GIF或PNG格式绘制出图形，并可以日、周、月等单位分别绘出。它也可产生出最大值最小值的资料供统计用。
原本MRTG只能绘出网络设备的流量图，后来发展出了各种插件，因此网络以外的设备也可由MRTG监控，例如服务器的硬盘使用量、CPU的负载等。

PS.MRTG替代方案
RRDtool：MRTG原作者开发的流量记录监控软件，较MRTG强大
PRTG：Windows下的流量监控软件

#MRTG运作过程
要了解MRTG的工作，就必须了解一下 SNMP (Simple Network Management Protocol)这个协议，SNMP是一种称之为简单网络管理协议的服务，主要是用于获取系统的流量、I/O、CPU、Memory和Disk等信息，通过自带的统计功能，将信息发送于监控程序上，最后以统计报表的形式展现于管理员。不过由于不同厂商的设备可能会有无法相容的情况，因此后来又有所谓MIB(Management Information Base)协议产生。
扯远了，总的来讲MRTG基本上是通过SNMP协议，向主机询问相关的资料后，主机返回以上提到的各种数值给MRTG，最后MRTG在绘制成网页上的图表，所以你使用MRTG监控的主机必须支持SNMP服务。
另外，有一点是需要大家注意的，MRTG其实总共需要得到四个数据（前两个用来作图，后两个提供相关资讯），因此你可以按自己需求让MRTG作图，只要你能提供两个数据，这个在后面的监控CPU或者是RAM的地方，加载自己写的程序后，就可以得到了。

1.填滿顏色的線條
2.不填色的線條
3.開機時間（設備啟動時間）
4.設備名稱

#这里讲一下安装前的准备工作
1）Perl平台的支持，因为MRTG是Perl语言开发的，所以需要安装ActivePerl，下载地址如：  http://downloads.activestate.com/ActivePerl/Windows/
当然了安装过程很简单，基本默认就ok了，安装目录Perl在C盘根目录下。
2）IIS服务，这个并不是必须的，但是为了方便浏览最后生成的报表，因为报表都是Html格式的，为了实时且可以远程访问这个页面，所以需要IIS服务来支持其web页面的浏览。
3）SNMP服务，这个组件是必须的，不管是监控或者是被监控设备都必须要安装该组件，一般在控制面板的“添加删除组件”中找到简单网络管理协议组件安装就ok。
4）MRTG，这个工具的下载地址为：http://oss.oetiker.ch/mrtg/download.en.html
这里有linux和windows版本的都有，格式略有不同，请注意这里下载的windows版本的格式为zip格式的。关于版本号的选择，建议使用最新版本，目前最新是4年前发布的2.17.4，这里演示使用的是2.17.2。

#开始配置MRTG
1）设置SNMP，首先我们需要在服务里开启SNMP服务，在服务列表中可以找到如下两个服务程序如下图所示：
首先我们需要在服务里开启SNMP服务，在服务列表中可以找到如下两个服务程序如下图所示：
如上有SNMP Service和SNMP Trap Service这两个服务，其中SNMP Service是主服务，而SNMP Trap Service是一个Trap工具，也就是抓取工具，获取Service的信息。这里我们只需要把主服务设置成自动然后开启它，Trap默认为手动即可。接下来我们需要哦欸之SNMP，如图所示：
打开主服务的属性，在安全标签页，这里如上图所示默认勾选“发送身份验证陷阱”，下面的接受团体框里添加一个团体，系统默认是public，通常情况这里建议不用系统默认的关键字，至于这个关键字的作用，后面配置MRTG的时候将会说明，重新添加一个团体名称，区别于public就可以，权利可以附加只读和创建两个即可。然后在下面可以看到可以选择接受哪些主机的SNMP信息，这里需要根据实际情况来设定，一般请指定特定的主机地址，添加IP地址即可。关于SNMP的服务配置就这些了，因为这里只是介绍MRTG的工具使用，有兴趣的同学可以自行去了解SNMP的应用
2）（前面提到Perl平台的安装，这里就不讲了）现在讲MRTG的安装，同样非常简单无脑，由于下载的是zip压缩包，所以直接解压丢进C盘根目录下就可以了
3）配置IIS应用服务器，这里主要是为MRTG建立一个文件夹，用于存放监控数据文件的地方，还有就是配置web页面的浏览。通常在安装完IIS应用之后会在C盘有如此目录c:\Inetpub\wwwroot\，在此目录下新建一个文件夹作为mrtg的服务目录，（名称随意，这里作为演示我用MRTG）然后在默认网站下新建一个站点，目录指向mrtg，这样就可以通过远程访问到mrtg下的web页面了。
4）配置MRTG
1.打开我们喜爱的cmd，切换到MRTG的bin目录下，路径就是你下载下来的zip压缩包解压丢进的目录，接着指定需要监控的主机地址等相关信息，在bin目录下执行命令：
perl cfgmaker public@192.168.1.1 --global "workdir: c:\Inetpub\wwwroot\mrtg" --output "c:\Inetpub\wwwroot\mrtg\pc.cfg"
PS.perl是执行平台脚本，cfgmaker是mrtg的命令，public@192.168.1.1中public就是SNMP Service中配置的接受团体名称，这里不建议使用public，@后面是受监控主机的IP地址；workdir是指定工作目录，而output是指定生成配置文件的输出目录，最后生成的配置文件以.cfg后缀格式。这样就配置好了监控的那台主机的配置服务。
2.然后，需要生成一个web页面来显示当前监控的信息，用到的命令为indexmarker，具体命令如下：
perl indexmaker c:\Inetpub\wwwroot\mrtg\pc.cfg>c:\Inetpub\wwwroot\mrtg\index.html
PS.注意路径不要搞错，然后你就可以在mrtg目录下看到index.html文件了。 
3.最后运行监控
命令为：
perl mrtg --logging=c:\Inetpub\wwwroot\mrtg\pc.log c:\Inetpub\wwwroot\mrtg\pc.cfg
PS.运行之后会看到下面有数据信息在滚动，说明SNMP已经在发送和接收信息，因为第一次运行没有历史日志文件所以这里会提示错误，关掉重复运行三次就可以了。

#查看图表
使用浏览器访问index.html页面，可以看到每个端口的数据流量图，默认是5分钟发送一次信息

#其他监控
由于要监控CPU和内存状态，需要安装SNMP4W2K-STD。下载地址: http://www.wtcs.org/snmp4tpc/snmp4w2k.htm

好了，重點說完了，再來說說在 mrtg.cfg 這個參數檔當中你看到的幾個參數的意義吧！
Target[裝置名稱]：
Target[vbird.adsldns.org_2]: 2: public@192.168.1.2

上面是一般的用法，其中半括號內的是裝置的名稱，同一個裝置的各參數中，這個名稱要一致！
Target[vbird.adsldns.org_3]:`/usr/local/apache/htdocs/mrtg/cpu/mrtg.cpu`

後面接的是一個自訂的加掛的可執行檔案，這個檔案執行之後，會顯示四個數據，這樣就可以繪圖了！在繪製非 MRTG 程式的預設咚咚中，這個是最常使用的方法了！
MaxBytes[裝置名稱]：
MaxBytes[vbird.adsldns.org_2]: 1250000

後面的數字代表資料監測時，最大的傳送速率，使用 bytes，所以 10Mbps 則為  1.25MBytes，大約是 1250000 Bytes。這個數值程式會自動判斷啦！不過你也可以自己修改，用到這個數字的時候是在你的圖表下方，每一個說明後面的(xx%)時用到的。
MaxBytes[vbird.adsldns.org_3]: 100

如果你的資料並不是 Bytes 時，例如監測 CPU 負載率時，那這個數值就需要改變啦！
Options[裝置名稱]：
Options[vbird.adsldns.org_2]: growright, bits  （用在網路流量中）
Options[vbird.adsldns.org_3]: growright, nopercent, gauge  （用在 CPU 負載中）
growright:將資料隨時間變化的順序以右而左繪圖； 
bits:資料單位為 bits； 
nopercent:在圖下方的說明文字中，不顯示百分比； 
gauge:圖表的上限固定！


### Global Config Options
WorkDir: c:\\wwwroot\\mrtg
### Global Defaults
# to get bits instead of bytes and graphs growing to the right
RunAsDaemon:yes
interval:5
Options[_]: growright
Language: chinese
EnableIPv6: no
# CPU
Target[CPU]: .1.3.6.1.4.1.311.1.1.3.1.1.2.1.3.1.48&.1.3.6.1.4.1.311.1.1.3.1.1.2.1.3.1.48:public@127.0.0.1
Ysize[CPU]: 200
Xsize[CPU]: 400
Ytics[CPU]: 10
MaxBytes[CPU]: 100
Title[CPU]: Windows2000 CPU使用率
PageTop[CPU]: <H1>;Windows2000 CPU使用率</H1>;
ShortLegend[CPU]: %
YLegend[CPU]: CPU Load
Legend1[CPU]: CPU Utilization # CPU可用资源
Legend2[CPU]: .
Legend3[CPU]: Max Value Per-Interval # 每个周期内CPU的最大负载值
Legend4[CPU]: .
LegendI[CPU]: CPU:
LegendO[CPU]:
Options[CPU]: gauge, growright, nopercent, unknaszero


### Crated by
### Global Config Options
RunAsDaemon:yes
interval:5
WorkDir: c:\\wwwroot\\mrtg
### Global Defaults
# to get bits instead of bytes and graphs growing to the right
Options[_]: growright
Language: chinese
EnableIPv6: no
#
# Memory Utilization (SNMP)
#
YLegend[memory]: % Memory Used
Options[memory]: growright,gauge
Target[memory]: .1.3.6.1.4.1.311.1.1.3.1.1.1.2.0&.1.3.6.1.4.1.311.1.1.3.1.1.1.2.0:public@127.0.0.1
MaxBytes[memory]: 523444000
Title[memory]: Memory Used
ShortLegend[memory]: %
Legend1[memory]: Vir in next minute
Legend2[memory]: Phy in next minute
Legend3[memory]: Maximal 5 Minute Vir
Legend4[memory]: Maximal 5 Minute Phy
LegendI[memory]: Vir
LegendO[memory]: Phy
PageTop[memory]: <H1>Memory Utilization</H1>