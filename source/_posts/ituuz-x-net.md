---
title: ituuz-x游戏框架v2.2--网络核心模块
date: 2019-11-14 20:45:25
tags: [cocos creator,pb,protobuf,protobufjs,http,网络,框架]
categories: cocos creator
---

> v2.2版本主要是对网络核心模块进行了封装，完善了框架体系。这个版本中目前只对http进行了实现，其他类型都是未实现的接口。

----
## v2.2版本功能列表
- [`new`]网络核心模块`net`
- [`new`]GameModel增加网络通信相关接口
- [`plug-in`]增加creator插件pb-generator
- [`bug`]修改Mediator销毁接口destroy有时不会调用的bug
- [`bug`]修改场景切换时，部分数据没有销毁的bug

## 主要功能详细介绍
### 网络核心模块介绍
框架中的网络模块意在于让网络的层的具体实现脱离与业务层，在业务层的开发人员无需关心是http、websocket还是其他连接方式，也不需要关心网络层的数据格式，协议解析及映射关系等，甚至于使用起来都不知道在做网络操作，最终表现就是发送一个对象或者接收到一个对象来使用。框架目前支持Websocket、http、local以及自定义连接实现。没错，本地存储local这里也作为网络层来封装，就像上面说的一样，框架可以让开发人员脱离数据层具体实现来进行开发。<!-- more -->
### 网络核心模块架构图
![网络核心模块架构图](/images/ituuzx_net.png)
### 节点介绍
- 最下面的数据层：是指各种网络连接的功能实现，这里定义了Websocket、http、local三种常用的数据交互方式，当然框架也提供了接口来注册自定义的网络数据实现类。这层主要实现的功能接口主要是创建连接和发送数据，稍后会介绍Http的实现类HttpClient，先看基类的接口定义如下:
```javascript 
/**
 * 网络客户端基类
 * @author ituuz
 */
export default abstract class NetClientBase {
    /** 连接地址 */
    public addr: string;
    /** 端口 */
    public port: number;
    /** 构造函数 */
    public constructor(addr: string, port: number) {
        this.addr = addr;
        this.port = port;
    }
    /** 发送消息 */
    public abstract sendReq(msg: MessageBase): void;
    /** 创建连接 */
    public abstract connect(succCB: () => void, faultCB: (code: NetFailCode) => void): void;
}
```
- 中间的数据支持层：数据支持层主要是工具类接口，网络核心层数据格式主要为protobuf，所以主要提供protobuf相关接口支持。同时MessageBase也是这层的一个重要基类。在网络核心层进行数据交互的对象都是继承MessageBase的Message对象，我们发送数据时就是new一个或者create一个Message对象然后发送出去，接收数据也是会接收到一个Message对象，然后直接读取该对象的属性。下面看一下MessageBase的几个主要接口:
```javascript
// 消息协议基类，声明的pb文件会生成对应的message类文件供业务使用。
export default class MessageBase {
    // 静态的创建Message接口，是一个异步接口，在我们没有提前预加载proto文件时使用这种异步的方法创建Message。
    public static create(cb: (msg: MessageBase) => void): void;

    // 加载pb文件接口该接口提供了加载该Message的proto原型对象的实现
    // 预加载pb时需要调用该接口，预加载后就不需要调用create异步创建了，直接new就可以创建了。
    public static loadPbFile(cb: () => {}): void;

    // 将Message转成ArrayBuffer接口，用于发送二进制数据。
    public toBuffer(): ArrayBuffer;

    // 解析ArrayBuffer数据为Message对象
    public parseBuffer(buffer): void;

    // 该Message的协议pid
    public get PID(): number;
}
```
- 最上层是业务层：业务层中我们处理数据主要放在MVC的Model层来处理，所以对数据的操作的接口封装在GameModel中，重要的几个接口如下：
```javascript
/**
 * 添加新回调消息，该接口用于添加注册要监听的消息。
 * @param {number} id 消息id
 * @param {(msg: MessageBase) => void} cb 消息回调
 */
public addRS(pid: number, cb: (msg: MessageBase) => void): void;

/**
 * 移除消息注册
 * @param {number} id 消息id
 */
public removeRS(pid: number): void;

/**
 * 发送消息
 * @param {MessageBase} msg 消息对象 
 */
public sendRQ(msg: MessageBase): void;

/**
 * 配置注册的PB消息
 */
public protobuf(): Array<{pid: number, cb: (msg: MessageBase) => void}>;
```
- 其他节点：日志记录和状态切换控制，后续会增加日志记录接口和网络状态控制接口。网络状态控制主要是控制切换网络地址，以及切换网络类型，比如长短连接之间的切换，待实现。

### 使用方式
- 定义proto文件：proto协议语法遵循标准的的protobuf语法，除语法外，需要指定亮点格式声明，只有遵循这两点格式才能使用后面提供的代码生成工具，下面我们声明一个简单的proto文件：
```javascript
syntax = "proto3";
package msg;
// 登陆请求 
// PID_KEY:10000
message LoginSend {
    string userId = 1;
    string token = 2;
}
// 登陆回复 
// PID_KEY:20001
message LoginBack {
    string userId = 1;
    string nickname = 2; // 昵称
}

// --------- 以下为说明内容 ---------
/**
 * 上面的协议声明遵循了protobuf的标准格式声明。
 * 除标准格式外，这里需要特别注意两点：
 * 1. PID_KEY的声明，在每个message协议体的上面都必须声明协议id：”PID_KEY:10000“，
 *    协议号10000是自定义的，可以按照自己的规则来声明，"// PID_KEY:xxxx"这是固定格式，
 *    message协议上面第一行必须是PID_KEY的声明，中间不能插入其他注释，注释可以卸载PID_KEY声明上面。
 * 2. message协议体内不能有换行注释，可以将注释放在字段声明的后面，就像上面的例子一样，
 *    因为插件工具解析协议时是按照换行解析协议体的，所以空行或者纯注释行会报错，这里暂时没有优化。
 * 
 * 以上两点就是声明协议需要注意的地方。pb代码生成工具还不是很完善，所以有bug在所难免，望大家指出。
 */
```
- 使用插件生成json文件和ts代码：pb-generator插件（参考下面的`协议生成工具介绍`）可以根据proto生成对应的json配置文件和ts代码文件，proto里定义的每个message都会对应生成一个类，在使用时直接new这个类然后send发送出去就可以了，接收到的也是一个对象，直接读取使用这个对象的属性就可以了。例如上面的登陆协议在代码里就像下面这样处理：
```javascript
let msg = new LoginSendMessage();
msg.userId = "1001";
// 直接将msg发送出去就可以了，具体方式下面介绍。
// 接收到的数据对象直接就是LoginBackMessage对象。
```
- 初始化网络管理器及创建连接
```javascript
/**
 * 初始化网络链接
 * @param {NetType} type 链接类型，目前只实现了HTTP类型，其他类型待实现。
 * @param {string} addr 链接地址url
 * @param {number} port 端口，当为http和local网络类型时端口参数无效，custom依赖于实现方式
 * @param {rsMap: Map<number, {new(): MessageBase}>} rsMap 协议映射关系表，协议id对应对象关系表
 * @param {new() => NetClientBase} customClient 自定义client网络类型，后面会介绍如何自定义网络实现
 */
NetHelper.init(NetType.HTTP, "http://10.194.6.66:10000/ituuz/", 0, MessageType.rsMap);

// 创建网络连接，当类型为http和local时必定成功，custom依赖实现
NetHelper.connect(() => {
    it.log("链接成功");
}, (code) => {
    it.log("创建连接失败:", code);
});
```
- 缓存pb协议
```javascript
/**
 * 加载pb文件，建议在创建连接connect之前调用。
 * 这个接口是提供加载在init接口中注册的pb协议，可选，影响的是后续消息创建的方式
 * 如果没有提前调用此接口那么创建消息时就是异步创建的，反之则是使用缓存同步创建，建议提前加载；
 */
NetHelper.loadPbFiles(() => {
    it.log("pb协议缓存成功");
});
```
- 发送和接收网络消息
```javascript
// 消息收发主要逻辑控制都在NetHelper中提供接口
// 首先是注册消息监听，注册后，收到服务器对应的消息后就会调用注册的接口cb
/**
 * NetHelper 注册回调消息
 * @param {number} id 消息id，是NetHelper.init接口中初始化的映射关系中的key值
 * @param {(msg: MessageBase) => void} cb 注册的回调，在回调中可直接读取message的属性
 */
public static registerRS(id: number, cb: (msg: MessageBase) => void): void；

// 然后是发送消息，发送消息首先要创建一个消息，然后发送；
// 创建消息分两种方式，没有提前缓存(loadPbFiles)的情况下只能异步创建，缓存过可以同步创建；
// 1.先看异步创建
LoginSendMessage.create((msg: LoginSendMessage) => {
    msg.userId = "ituuz";
    msg.token = "xxxx";
    // 然后就可以将msg发送给服务器了
});
// 2.然后同步创建，前提是有提前加载缓存，否则创建的message可能会无效。
let msg = new LoginSendMessage();
msg.userId = "ituuz";
msg.token = "xxxx";
// 然后就可以将msg发送给服务器了
```
- 在MVC中使用网络模块:在ituuz-x框架中核心的mvc模块中集成了数据模块Model，数据的交互都在Model中进行，所以消息的收发都集成在了Model中，主要接口如下：
```javascript 
/**
 * 发送消息
 * @param {MessageBase} msg 消息对象 
 */
public sendRQ(msg: MessageBase): void;
/**
 * 配置注册的PB消息
 * 该接口需要在自己实现的model中进行重写，并返回该model中注册的消息监听。
 */
public protobuf(): Array<{pid: number, cb: (msg: MessageBase) => void}>;

// 除上面两个必须的接口外，还提供了更为灵活的注册方式：
/**
 * 添加新回调消息
 * @param {number} id 消息id
 * @param {(msg: MessageBase) => void} cb 回调
 */
public addRS(pid: number, cb: (msg: MessageBase) => void): void;
/**
 * 移除消息注册
 * @param {number} id 消息id
 */
public removeRS(pid: number): void;
```
### 协议生成工具(pb-generator)介绍
#### 该工具是由nodejs实现，根据上面我们声明的pb文件，自动生成pb目标文件，和根据代码模版生成的ts代码，也就是对应的Message对象，可以在业务中直接使用。
#### 工具配置，插件在plug-in目录下，在插件目录下的config.js文件就是插件的配置文件，使用时直接将插件目录copy到项目的插件目录下，然后修改config.js中的配置，最后重启creator就可以看见拓展选项中的插件了，点击生成就可以生成代码了。

### 自定义网络类型
#### 上面讲到初始化网络时有个类型是自定义类型，这里讲以下如何使用自定义网络类型。
#### 在框架中提供的协议解析方式不能满足需求时可以自己实现拓展
- 继承NetClientBase并实现必要的几个接口：
```javascript
/**
 * 发送协议接口
 * @param {MessageBase} msg pb消息对象，该对象提供了将Message对象进行序列化和反序列化的接口提供使用
 * 实现该接口可以自定义数据结构以及发送方式。
 */
public abstract sendReq(msg: MessageBase): void;
/**
 * 创建连接接口，需要实现连接的功能，如websocket可以在这里处理创建的成功和失败，如果自定义的网络类型不需要
 * 创建连接的阶段，那么直接返回成功回调即可。
 */
public abstract connect(succCB: () => void, faultCB: (code: NetFailCode) => void): void;
```
- 自定义NetClientBase例子
```javascript
export default class HttpClient extends NetClientBase {

    public static readonly DATA_TOTAL_LEN = 4;	// 数据总长度
    public static readonly PROTOCOLTYPE_LEN = 4;	// 协议号长度

    /**
     * 发送消息协议
     * @param {MessageBase} msg 消息对象
     */
    public sendReq(msg: MessageBase): void {
        let xhr = new XMLHttpRequest();
        xhr.onreadystatechange = () => {
            if (xhr.readyState === 4 && (xhr.status >= 200 && xhr.status < 400)) {
                let response: ArrayBuffer = xhr.response;
                this.encode(response);
            }
        };
        xhr.open("POST", this.addr, true);
        xhr.responseType = "arraybuffer";
        let buffer = this.decode(msg);
        xhr.send(buffer);
    }

    /**
     * 对消息体进行压包
     * @param {MessageBase} msg 消息对象
     * @return {ArrayBuffer} 压包后的而进行数据
     */
    private decode(msg: MessageBase): ArrayBuffer {
        let buffer = msg.toBuffer();
        let dataView = new DataView(buffer);
        let dataLen = buffer.byteLength;
        let sendBuf = new ArrayBuffer(HttpClient.DATA_TOTAL_LEN + HttpClient.PROTOCOLTYPE_LEN + dataLen);
        let sendView = new DataView(sendBuf);
        sendView.setInt32(0, msg.PID);
        sendView.setInt32(HttpClient.PROTOCOLTYPE_LEN, dataLen);
        for (let i = 0; i < dataLen; i++) {
            sendView.setInt8(HttpClient.PROTOCOLTYPE_LEN + HttpClient.DATA_TOTAL_LEN + i, dataView.getInt8(i));
        }
        return sendBuf;
    }

    /**
     * 对二进制数据进行解包
     * @param {ArrayBuffer} recvBuf 接收到的二进制数据
     */
    private encode(recvBuf: ArrayBuffer) {
        let recvView = new DataView(recvBuf);
        let PID = recvView.getInt32(0);
        // let len = recvView.getInt32(HttpClient.PROTOCOLTYPE_LEN);
        let data = recvBuf.slice(HttpClient.DATA_TOTAL_LEN + HttpClient.PROTOCOLTYPE_LEN, recvBuf.byteLength);
        let cls = NetHelper.getMessageCls(PID);
        let msg: MessageBase = (cls as any).create(() => {
            msg.parseBuffer(data);
        });
    }

    public connect(succCB: () => void, faultCB: (code: NetFailCode) => void): void {
        if (succCB) {
            succCB();
        }
    }
}
```

## 最后有任何问题都可以给我留言，通过关注我的公众号留言或者网站留言都可以。



