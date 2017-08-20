#Pipenet 入门教程

## 获取Pipenet
 [点击此处](https://github.com/RobertIndie/Pipenet) 下载Pipenet最新发布版，然后导入到项目工程中。
 
 Pipenet基于.NET Standard开发，可以导入到基于.NET Core或.NET Framework项目中。
 
## Pipeline
Pipenet通信的基本工具是使用Pipeline，Pipeline类似管道连接在各个进程之中进行可靠数据传输。在Pipeline中可以选择多种底层通讯方式(目前自带的只有Socket TCP传输)，也可以选择多种应用层上的通讯方式(目前只有事件驱动模式即EventPipeline)。

### 主动与被动Pipeline
Pipeline分为主动与被动模式，主动Pipeline一般用作客户端，被动Pipeline一般用作服务端。

主动Pipeline采取主动连接到指定IP和端口；被动Pipeline会侦听所设置的IP和端口并等待主动Pipeline连接。

### 单端与多端Pipeline
Pipeline又分单端与多端，单端Pipeline一次只能连接一个进程，而多端Pipeline一次可以连接多个进程。

然而，主动多端型Pipeline尚未实现，将在后续版本中实现。

## 利用PipelineSettings设置Pipeline
通过PipelineSettings可以设置Pipeline的属性，包括将Pipeline设置成主动或被动、单端或多端。其源码如下：

    public class PipelineSettings
    {
        public enum ConnectionType
        {
            TCP
        }
        public string Ip
        {
            get; set;
        }
        public int Port
        {
            get; set;
        }
        public bool IsListen//如果希望Pipeline是被动型，则设置此项为true，充当服务端
        {
            get;set;
        }
        public bool IsMultiConnect//如果希望Pipeline是多端型，则设置此项为true
        {
            get;set;
        }
        public ConnectionType transportType
        {
            get;set;
        }
        public PipelineSettings()
        {
            Ip = "127.000.000.001";
            Port = 8078;
            IsListen = false;
            transportType = ConnectionType.TCP;
            IsMultiConnect = false;
            //默认的设置为连接地址为127.0.0.1:8078的主动单端Pipeline，即客户端
        }
    }


## 定义Pipeline
Pipeline继承IEventPipeline,IMultiTransport接口。如果希望Pipeline是采用事件驱动模式和单端模式则使用IEventPipeline对Pipeline进行操作，如果希望Pipeline是采用事件驱动模式和多端模式则使用IMultiTransport对Pipeline进行操作。

	//定义一个单端的Pipeline：
	IEventPipeline pipeline = new Pipeline(settings);//settings为前文设置好的PipelineSettings实例
	
	//定义一个多端的Pipeline
	IMultiTransport pipeline = new Pipeline(settings);
	
这样，一个Pipeline就定义好了。

## 为事件驱动型Pipeline添加事件
事件驱动型Pipeline的通讯方式为：在进程两端设置好相应的事件，由一端触发另一端的事件实现通信。

所以，我们需要在Pipeline连接前添加一些事件供另一端的进程进行触发。

### 有返回值事件与无返回值事件
触发有返回值事件时会返回数据到触发的一端，无返回值则不返回数据。

	pipeline.AddEvent("Output", OutputMessage);//添加一个无返回值事件，第一个参数为事件名，第二个参数为触发事件时调用的方法
	//方法格式如下:
	void OutputMessage(ITransport transport, object[] parameters) 
	{
		//代码...
	}
	
	pipeline.AddReturnEvent("Output",OutputMessage);//添加一个有返回值的事件
	//OutputMessage方法格式如下：
	object OutputMessage(ITransport transport, object[] parameters) 
	{
		//代码...
	}
	
如果触发的是有返回值事件，那么触发的一端所触发用的线程会等待，直到被触发端返回数据。

## 连接Pipeline
	pipeline.Connect();

##  触发事件
	pipeline.Invoke("Output", new object[] { message });//这是在触发远程一端名为Output的时间，后面是参数

## 更多使用方法
更多使用方法详见Pipenet代码库的Demo文件夹下的Demo，里面有网络聊天室的例子，可从中学得更多的Pipenet使用方法。