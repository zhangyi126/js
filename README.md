# js
使用Node js环境，读取Modbus-RTU设备数据
1、什么是Modbus
  Modbus是一种串行通信协议，是Modicon公司（现在的施耐德电气 Schneider Electric）于1979年为使用可编程逻辑控制器（PLC）通信而发表。Modbus已经成为工业领域通信协议的业界标准（De facto），并且现在是工业电子设备之间常用的连接方式。（解释来自百度百科）
  在我的工作中，常常接触一些设备，这些设备可以使用Modbus向外传递数据，有时我想测试一下设备和电脑连接的正确性而不想运行一个“很重”的组态软件或测试工具，因此查阅资料写了这个程序。
2、当前的效果
	Modbus设备的结果并以十进制显示；读取一个设备的多个项目值；
3、可以继续的地方
	 读取到数据后存储到数据库，进行Web展示;进行中转发送到其他服务器
4、参考资料
	 node.js 的Modbus-serial库：https://www.npmjs.com/package/modbus-serial
	 将Modbus返回值转换为十进制浮点数：https://blog.csdn.net/qq_19286023/article/details/72651568?locationNum=16&fps=1
	 node.js 的串口库serialport：https://www.npmjs.com/package/serialport
5、安装Modbus-serial库的注意事项
		 我是安装了vs2012之后才安装成功的。
6、代码
---------------------------------
辅助方法
//利用项目十进制值判断项目名称
function getName(itemNum){
    var itemName;
    switch (itemNum) {
        case 256:
            itemName =  '环境温度';
            break;
        case 258:
            itemName = '水样温度';
            break;
        case 260:
            itemName = '电导率';
            break;
        case 262:
            itemName = '溶解氧(荧光法)';
            break;
        case 264:
            itemName = '溶解氧(极谱法)';
            break;
        case 268:
            itemName = '电极余氯';
            break;
        case 270:
            itemName = 'ORP';
            break;
        case 512:
            itemName = '浊度';
            break;
        case 514:
            itemName = '色度';
            break;
        case 516:
            itemName = 'pH';
            break;
        case 518:
            itemName = '余氯';
            break;
        case 520:
            itemName = '总氯';
            break;
        case 522:
            itemName = '二氧化氯';
            break;
        case 524:
            itemName = '活性氧';
            break;
        case 526:
            itemName = '氨氮';
            break;
        case 528:
            itemName = '亚硝酸盐氮';
            break;
        case 530:
            itemName = '尿素';
            break;
        case 532:
            itemName = '六价铬';
            break;
        case 534:
            itemName = '空';
            break;
        case 536:
            itemName = '空';
            break;
        case 538:
            itemName = '空';
            break;
        case 540:
            itemName = '余氯OT';
            break;
        case 542:
            itemName = '空';
            break;
        case 544:
            itemName = '空';
            break;
        case 546:
            itemName = '浊度(低量程)';
            break;
        case 548:
            itemName = '氨氮2';
            break;
        default:
            console.log('无此项目编号，返回空');
            itemName = '空';
    }
    console.log(itemName);
    return itemName
}

-----------------------------------
辅助方法
//转换串口返回的buffer为10进制浮点数
function transBufferToFloat(buffer) {
    console.log('从串口返回的buffer：',buffer);
    var buffer_to_array = Array.from(buffer).map(x => x.toString(16));
    console.log('buffer_to_array是：',buffer_to_array);
    //将buffer_to_array倒序一下
    buffer_to_array = buffer_to_array.reverse();
    console.log('倒序后的buffer_to_array是：',buffer_to_array);
    //将数据组合成字符串，使用split方法和join方法
    //data = buffer_to_array.split(',').join('');
    var comData = buffer_to_array.toString().split(',').join('');
    console.log('转换成后的：',comData);
    //转换为2进制
    var data_bit = parseInt(comData,16).toString(2);
    console.log('转换为2进制后：',data_bit);
    //截取阶码对应的十六进制
    var data_E= parseInt(data_bit.slice(0,8),2);
    console.log('data_E是：',data_E);
    // 获得尾数
    var data_M = data_bit.slice(8,64);
    console.log('data_M是：',data_M);
    //将二进制尾数转换为十进制的小数
    var data_M_10 = 0.00;
    for(var i = 0;i<data_M.length;i++){
        data_M_10 = data_M_10 + data_M[i]*Math.pow(2,(-1)*(i+1));
    }
    console.log('十进制的尾数是：',data_M_10);
    var data_float = Math.pow(2,data_E-127)*(1+data_M_10);
    console.log('data_float是：',data_float);
    console.log('项目结果是：',data_float.toFixed(2));
    return data_float.toFixed(2);
}

			//读取一个设备的多个项目 示例
var ModbusRTU = require('modbus-serial');
var client = new ModbusRTU();

//初始化连接信息
function init(port,baudRate,id,timeOut) {
    try {
        client.connectRTU(port, { baudRate:baudRate });
        client.setID(id);
        client.setTimeout(timeOut);
        console.log('串口打开成功',port,baudRate,id,timeOut);
        sleep(1000);
    }catch (e) {
        //e.message
        console.log('打开失败')
    }
		// 需要读取的项目地址列表
const metersItemList = [256,258,270,512,514];

//循环赋项目地址
const getMetersValue = async (meters) => {
    try{
        // get value of all meters
        for(let meter of meters) {
            // output value to console
            console.log(await getMeterValue(meter));
            // wait 5000ms before get another device
            await sleep(5000);
        }
    } catch(e){
        // if error, handle them here (it should not)
        console.log(e)
    } finally {
        // after get  data  repeate it again
        setImmediate(() => {
            getMetersValue(metersItemList);
        })
    }
};
//读取给定项目地址的值
const getMeterValue = async (itemNum) => {
    try {
        // set ID of slave
        await client.setID(1);
        //获取项目名称
        var itemName = getName(itemNum);
        // read the 2 registers starting at address itemNum
        let val =  await client.readHoldingRegisters(itemNum, 2);
        // return the value
        return transBufferToFloat(val.buffer);
        // return val.buffer;
    } catch(e){
        // if error return -1
        console.log(e.message);
        return -1
    }
};

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

//初始化，打开端口
init('COM4',9600,1,10000);
// start get value
getMetersValue(metersItemList);



