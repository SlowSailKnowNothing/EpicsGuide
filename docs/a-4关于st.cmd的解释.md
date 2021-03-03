### 关于st.cmd的解释

下面是一个典型的st.cmd的文件：

```html

 #!../../bin/linux-x86_64/e2260IOC                                                       
  5 < envPaths
  6 epicsEnvSet("STREAM_PROTOCOL_PATH","${TOP}/protocols")
  7 cd "${TOP}"                    
  8   
  9 ## Register all support components
 10 dbLoadDatabase "dbd/e2260IOC.dbd"
 11 e2260IOC_registerRecordDeviceDriver pdbbase
 12           
 13 ## Load record instances       
 14 dbLoadRecords("db/e2260ai.db","user=opi")
 15         
 16 drvAsynIPPortConfigure "modbus", "192.168.127.254:502"
 17       
 18     
 19     
 20  
 21 cd "${TOP}/iocBoot/${IOC}"
 22 iocInit
 23  
 24 ## Start any sequence programs
 25 #seq sncxxx,"user=opi"

```

5：执行文件envPaths，那么envPaths是什么内容呢？打开st.cmd的同一个目录，可以看到makefile之外就是envPaths，打开envPaths，可以看到本次建立的IOC的envPaths的内容如下：

```html
 
   epicsEnvSet("IOC","ioce2260IOC")
   epicsEnvSet("TOP","/hls/epics/learn/learn2/StreamDeviceTry/e2260IOC")
   epicsEnvSet("EPICS_BASE","/hls/epics/base-3.15.7")
   epicsEnvSet("asyn","/hls/epics/asyn4-37")
   epicsEnvSet("stream","/hls/epics/Soft/StreamDevice-master")
                                                                                                                 

```

其实该文件就是调用了epicsEnvSet来做一个宏的替换。比如IOC就替换为ioce2260IOC。该文件其实就是之前在configure的release中定义的路径，通过make生成的。如何生成不用管。

6：继续调用epicsEnvSet，这是用户自己做的一个宏的替换。这个STREAM_PROTOCOL_PATH是StreamDevice为我们取好的名字，我们调用epicsEnvSet指定协议文件的地址。其中TOP路径已经在envPaths被定义了。

7：切换到{TOP}所在的文件夹。

9：注释而已。

10：Load dbd文件，一般不用管。如果出现缺少dbd文件，可能是make里面的libs没有配置好。

11：不懂。望文生义，注册dbd。

16：asyn函数

21：切换目录

22：ioc初始化命令。

最开始的#!符号是指定执行该命令的解释器。可以切换到bin目录看一看，这个解释器是自动生成的，如果因为某些原因报错（比如修改别人的st.cmd）发现是解释器错误，可以看看解释器的名字和bin目录下的文件的名字是否相同。

如果希望对ioc进行更新，但是希望保持原来的ioc不要变化，可以先复制原来的ioc的目录，然后更改，然后将ioc复制到需要的位置；但是这个时候会出现上面的解释器损坏的问题，因此可以先将bin文件夹删除，然后重新make，就可以正常运行st.cmd了。