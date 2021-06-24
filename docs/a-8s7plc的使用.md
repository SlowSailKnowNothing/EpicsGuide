#### S7PLC使用说明

首先对s7plc的使用应该有一个大致的理论了解，这部分见[此文档](http://epics.web.psi.ch/style/software/s7plc/s7plc.html)

- 下面是提取的相关的要点：
    - 操作理论
        - 周期性交换数据
        - PV变量通过在data block中的偏置来确定
        - IOC启动的时候，driver就尝试连接TCP Server，无法建立连接的话，那么driver周期性地建立连接；一旦连接，driver等待被PLC发送的数据块；
        - 如果driver没有及时接收数据，driver考虑连接中断并且关闭连接。
        - driver从block中获取pv量之后，触发对记录的处理，为了使能这个触发，输入记录必须使用IO触发模式。
        - 整个block的数据都会被完全转换，这意味着所有的pv量都将转换，即使没有被改变，如果数据改变太快，比transfer的速度快，数据会丢失
    - 驱动配置方法：
        - st.cmd
            - s7plcConfigure (__PLCname__, __IPaddr__, __port__, __inSize__, __outSize__, __bigEndian__, __recvTimeout__, __sendIntervall__)
                - pclname随便一个名字，后面两个参数指定ip地址和端口
                - 输入大小和输出大小，可以设置为0
                - bigEndian位如果为1，那么就是MSB优先（最高有效位），否则就是LSB最低有效位优先。
                - 如果IOC在recvTimeout毫秒内没有从IOC中接收数据，那么它会中断连接并在数秒之后重新连接。
                - s7plcConfigure ("vak-4", "192.168.0.10", 2000, 1024, 32, 1, 500, 100)
                - 
        - 设备支持
            - DTYP 设置为 "S7Pplc"
            - SCAN 设置为"I/O Intr"
            - INP或者OUT字段的配置：
                - "@__PLCname__/__offset__ T=__type__ L=__low__ H=__high__ B=__bit__"
                - PLCname和st.cmd中定义的plcname相同
                - offset设置为字节相对于输入或者输出数据库的偏移量，必须是一个整数或者是整数的加和
                - T定义了数据类型，数据类型参照表格
                - 例子
                    - ```javascript
                 record(waveform, "$(NAME)") {
                 field (DTYP, "S7plc")
                 field (INP,  "@$(PLCNAME)/$(OFFSET)")
                 field (SCAN, "I/O Intr")
                 field (NELM, "$(NUMBER_OF_ELEMENTS)")
                 field (FTVL, "$(DATATYPE)")
                  }```
                 ```

#### s7plc的安装

虽然文档清楚，但是官方文档中始终没有找到如何安装s7plc，甚至连s7plc的下载都不在那个文档内，后来在[这个网页](http://epics.web.psi.ch/software/s7plc/)虽然找到下载的地址，但是版本明显老旧，也没有说明如何配合epics使用，因此后面去github上找到了最新版本，发现和StreamDevice文件结构类似，于是使用了github版本，并按照StreamDevice的安装方法安装了S7plc,下面是一个说明：

- 首先是添加dbd文件，下面是要添加的LIBS文件：

```
testPLC_DBD += base.dbd
testPLC_DBD +=modules.dbd
testPLC_LIBS+=s7plc
```

modules.dbd文件是自己新建一个再添加如下内容的：
- ```javascript
include "s7plc.dbd"```

- 另外要注意一点是，s7plc安装包解压之后，一样要在RELEASE文件里面指示epicsbase的位置，和StreamDevice一样。
- 然后是db文件的撰写，官方文档已经给出例子，参考例子改写即可：
    - ```javascript
   record(waveform, "$(NAME)") {
    field (DTYP, "S7plc")
    field (INP,  "@$(PLCNAME)/$(OFFSET)")
    field (SCAN, "I/O Intr")
    field (NELM, "$(NUMBER_OF_ELEMENTS)")
    field (FTVL, "$(DATATYPE)")
     }```
    - 官方文档将如何配置说的很清楚，照做即可。