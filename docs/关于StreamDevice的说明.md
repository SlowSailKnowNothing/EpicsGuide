### 关于StreamDevice的说明

StreamDevice是epics软件体系的一个**模块**，通过使用这个模块，编写对应的**协议**，就可以实现本地的ioc和硬件的交互。

想要使用StreamDevice，应该在src文件夹中添加上对应的libs文件。具体的配置步骤可以见后面的实战案例。下面首先介绍如何使用StreamDevice和硬件交互，然后在源码级别分析使用StreamDevice的细节。

上面已经介绍过了，IOC的核心是数据库。我们编写db文件，添加对应的记录，来实现对“数据的控制”。不同的记录类型，拥有对数据进行操作的不同能力。而与硬件交互的，这里主要分析ao记录和ai记录。

[这里是StreamDevice官网的一个使用StreamDevice的入门例子，可以作为参考](https://paulscherrerinstitute.github.io/StreamDevice/setup.html)

1.首先配置好**configure/RELEASE**文件。开始epics学习的时候，出现的一半问题在路径设置不对，当make出现问题的时候应当仔细看编译的错误提示，缺什么补什么。

2.进入StreamDevice的目录make，文件夹中会建立stream libary和stream.dbd以及IOC的例子应用。

3.如果在自己的IOC中要使用StreamDevice，在src的makefile文件中添加下面的libs依赖：

```html
XXX_DBD += modules.dbd //注意，这里的XXX参看makefile文件中的其他dbd的文件名
PROD_LIBS += stream
PROD_LIBS += asyn
```

在同文件夹下建立modules.dbd,添加内容如下：

```html
include "base.dbd"
include "stream.dbd"
include "asyn.dbd"
registrar(drvAsynIPPortRegisterCommands)
registrar(drvAsynSerialPortRegisterCommands)
registrar(vxi11RegisterCommands
```

4.添加自己编写的db文件。

5.修改iocBoot里面的st.cmd。这里的st.cmd的例子和之前不同的地方在于会利用

epicsEnvSet设置protocols文件，以及调用asyn的函数。比如官网的这个例子：

```html
epicsEnvSet ("STREAM_PROTOCOL_PATH", ".:../protocols")#设置#STREAM_PROTOCOL_PATH=../protocols

drvAsynSerialPortConfigure ("PS1","/dev/ttyS1")
#drvAsynSerialPortConfigure("portName","ttyName",priority,noAutoConnect,
noProcessEosIn)#具体的函数可见asyndriver的说明，注释给出了每个参数的接口，下面的查询方式类似
asynSetOption ("PS1", 0, "baud", "9600")
asynSetOption ("PS1", 0, "bits", "8")
asynSetOption ("PS1", 0, "parity", "none")
asynSetOption ("PS1", 0, "stop", "1")
asynSetOption ("PS1", 0, "clocal", "Y")
asynSetOption ("PS1", 0, "crtscts", "N")
```

6.建立proto文件，注意文件保存的位置要和epicsEnvSet的环境相同。

官网的proto文件如下所示：

```html
getCurent {
        out "CURRENT?";
        in "CURRENT %f A";
    }

setCurrent {
    out "CURRENT %.2f";
    @init {
        getCurent;
    }
}
```

对于上面的文件做如下解释：

1.getCurent的命名是用户随意的，符合C语言规范就好。尽量有意义，可以和例子一样使用驼峰命名法。也即getCuent还是getCurent都是没关系的，都是用户自己定的，名字会在记录中用到，起到一个指定的作用。就像函数会有函数名，当记录需要做一些动作的时候，也要指定协议里面的名字。

2.协议文件实际只做两个事情，in和out。动作发出的主体可以理解为使用“函数”（姑且将getCurent和setCurrent称作函数）的记录。

3.函数和函数的不同在于，in和out的字符组合是不一样的。即StreamDevice是通过字符和硬件交流的。即StreaDevice和硬件的交互是字符级别的，比特级别的不用管。

4.函数的格式是 

```html
functionName{
in "";
out "";
}
```

5.这里确定用什么字符，就要看具体协议了。如果主机和从机之间的字符交互很简单，比如

ioc向设备发送一个“CURRENT?”，设备就会向ioc返回一个“CURRENT 2.6 A”，那么协议就可以向这样写：

```html
getCurent {
        out "CURRENT?";
        in "CURRENT %f A";
    }
```

6.对于得到的数据，StreamDevice采用的是类似C语言的占位符的方式。比如，可以确定返回的数据是“CURRENT val A”的格式的时候，那么就可以利用占位符%f替代val。

7.往往协议不会这么简单，比如modbus协议，那么就要根据modbus协议的内容编写协议。

比如下面的利用modbus通信的协议文件如下：

```html
getT{

        out  "\x03\x8A\x00\x00\x00\x06\x01\x04\x00\x00\x00\x01";
        in   "\x03\x8A\x00\x00\x00\x05\x01\x04\x02%2r";

}
```

其中，out 里面的字符串的确认方式是根据modbus协议的字节规范，编写对指定地址的读写协议。而in里面的%2r的逻辑与上面的%f的逻辑相同。就是输入中会有输入值是变化的，前面是modbus的协议码，为了将输入值取出来，就使用占位符，这和C语言中的printf和scanf的道理机制是相似的，可以联系思考。



如果出现了问题，可以利用vscode工具打开EpicsBase，asyn和StreamDevice的源码来查看问题。下面在源码级别解释下ioc的具体记录时如何和硬件通信的。