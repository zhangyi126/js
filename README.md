使用Node.js环境，使用js读取Modbus-RTU设备数据  
什么是Modbus  

Modbus是一种串行通信协议，是Modicon公司（现在的施耐德电气 Schneider Electric）于1979年为使用可编程逻辑控制器（PLC）通信而发表。Modbus已经成为工业领域通信协议的业界标准（De facto），并且现在是工业电子设备之间常用的连接方式。（解释来自百度百科）   
在我的工作中，常常接触一些设备，这些设备可以使用Modbus向外传递数据，有时我想测试一下设备和电脑连接的正确性而不想运行一个“很重”的组态软件或测试工具，因此查阅资料写了这个程序。   
当前的效果  
读取Modbus设备的结果并以十进制显示；  
读取一个设备的多个项目值； 
可以继续的地方   
读取到数据后存储到数据库  
进行Web展示;进行中转发送到其他服务器   
参考资料  
​node.js 的Modbus-serial库：https://www.npmjs.com/package/modbus-serial   
将Modbus返回值转换为十进制浮点数：https://blog.csdn.net/qq_19286023/article/details/72651568?locationNum=16&fps=1   
node.js 的串口库serialport：https://www.npmjs.com/package/serialport   
安装Modbus-serial库的注意事项 我是安装了vs2012之后才安装成功的。  
