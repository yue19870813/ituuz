---
title: cocos creator实现战旗类游戏移动范围效果
date: 2019-09-21 15:29:44
tags: [javascript,cocos creator]
categories: cocos creator
---
最近刚刚把《火焰纹章：风花雪月》三周目通关，作为战旗游戏来讲，无论是核心玩法还是创新的养成GalGame要素，还有让人唏嘘的剧情，作为战旗类游戏的代表，名副其实。感叹之余就想实现一下《火纹》里核心战斗时的效果。下面是火纹里的效果：  
![fhxy.png](/images/fhxy.png)  
<!--more-->
《火纹》中效果是3D的，但算法逻辑是一样的，所以这里只实现逻辑，用2D来表现了，实现后的效果如下：  
![fhxy.png](/images/myhw.gif)  
例子中不同颜色的格子代表不同地形，消耗的行动力不同，当鼠标点击某一个格子的时候计算这个格子可以行动的范围，绿色是可行动范围，红色是不可移动的边缘部分。下面我们看一下实现思路。  

#### 移动逻辑
首先我们看下图：  
![fhxy_001](/images/fhxy_001.png)  
假如黑色圆圈是要移动的角色，那么判断移动范围的第一步就是判断上下左右四个格子是否可以行走，根据周边四个格子的地形所消耗的行动力比较角色的行动力，如果行动里足够那么这个格子就可以行走，存入结果；如果行动力不够那么这个格子就不能行走。周边的四个格子都判断完成后，我们看下一轮判断：  
![fhxy_001](/images/fhxy_002.jpg) 
假如上轮判断中只有两个打对号的格子可以行走，那么我们将这两个格子存入结果，并继续从这两个格子出发，以这两个格子作为新起点继续判断他们周围的格子（上轮判断过的格子可以跳过不用再次判断）是否满足移动条件，以此类推，直到行动力不够再继续移动，那么结果列表里存储的格子就是最终结果了，我们根据结果做地图表现就可以了。

#### 代码实现
这里除了基本的实现外还增加了一些拓展功能，方便直接拿来使用，也能为业务逻辑提供便利，比如传入自定义地图数据，地图配置，是否允许行动为0，是否显示不能移动的边缘，以及可以配置角色擅长的地形和不擅长地形等，可以丰富玩法的功能。  

首先先封装了一个初始化配置的接口，用来初始化地图数据，地形配置，还有拓展的功能做小移动配置。地图配置就是二维的坐标数据，每个坐标存储了当前坐标的地形；地形配置是告诉程序每种地形消耗的行动力是多少；最小移动配置，就是当行动力不足以移动时，是否允许移动一步的配置；
```javascript
/**
 * 初始化地图配置，注意地形类型不要设定为0，推荐为大于0的整数，算法中0作为默认值存在。
 * @param {Map<string, number>} mapData 地图数据，map的key是地图坐标x + “_” + y, value是地面类型；
 * @param {Map<number, number>} mapCfg 地形配置，对应类型消耗的行动力；默认为消耗1行动力地形；
 * @param {number} minStep 最小移动配置，就是当行动力不足以移动时，是否允许移动一步的配置；
 */
initMapConfig(mapData: Map<string, number>, mapCfg?: Map<number, number>, minStep?: number) {
    this._mapData = mapData;
    if (mapCfg) {
        this._mapCfg = mapCfg;
    } else {
        this._mapCfg = new Map<number, number>();
        this._mapCfg.set(1, 1);
    }
    if (minStep) {
        this._minStep = minStep;
    }
}
```
下面这个函数就是我们在业务中控制角色每次要移动前去获取可移动数据的接口了，该接口中首先把角色站的点存入结果列表，作为肯定可以行走的原点开始逐步扫描，最终把扫描结果返回，其中做了是否返回边缘不可行走格子的判断逻辑，需要说明的是这里增加了个targetCfg参数，如参数说明所示，它是用来做角色移动多样化的参数：
```javascript
/**
 * 获取移动数据
 * @param {cc.Vec2} pos 移动原点 
 * @param {number} limit 行动力
 * @param {Map<number, number>} targetCfg 当前移动对象的配置，用于配置目标擅长或者劣势地形，默认为空(可选)
 * @param {boolean} isShowEdge 是否返回不能移动的边缘数据，默认为不返回(可选)
 */
getCanMoveData(pos: cc.Vec2, limit: number, targetCfg?: Map<number, number>, isShowEdge: boolean = false): MapPos[] {
    if (targetCfg) {
        this._targetCfg = targetCfg;
    } else {
        this._targetCfg = null;
    }
    // 存储可移动坐标的结果数组
    let resultPos: MapPos[] = [];
    // 将原点存入结果
    let center = new MapPos(pos.x, pos.y, 0, MapPosStatus.CAN_MOVE);
    resultPos.push(center);
    let stepCount = 0;
    let start = 0;
    // 逐步判断
    while (stepCount < limit) {
        start = this.scanMap(resultPos, limit, start);
        stepCount++;
    }
    // 是否显示不可移动的边缘
    if (isShowEdge) {
        let r = resultPos.concat(this._canntList);
        this._canntList = [];
        return r;
    } else {
        return resultPos;
    }
}
```
下面我们看一下扫描地图的函数实现，这里就是我们上面画图来表示的那部分逻辑，分别对目标格子的上下左右进行判断，看是否可以移动，函数里的start是标记上次判断的位置，第二次扫描时直接从上次扫描过的位置开始就可以了：
```javascript
/**
 * 开始扫描地图
 * @param {MapPos[]} resultPos 结果列表
 * @param {number} limitStep 行动力
 * @param {number} start 当前检索的起始位置，优化算法使用
 */
scanMap(resultPos: MapPos[], limitStep: number, start: number) {
    let len = resultPos.length;
    for (; start < resultPos.length; start++) {
        let pos = resultPos[start];
        // 检查四个方向
        this.checkMapPos(new MapPos(pos.x, pos.y - 1, pos.limit), resultPos, limitStep);   // 上
        this.checkMapPos(new MapPos(pos.x, pos.y + 1, pos.limit), resultPos, limitStep);   // 下
        this.checkMapPos(new MapPos(pos.x - 1, pos.y, pos.limit), resultPos, limitStep);   // 左
        this.checkMapPos(new MapPos(pos.x + 1, pos.y, pos.limit), resultPos, limitStep);   // 右
    }
    return len;
}
```
最后我们看一下扫描函数里调用的检查单个格子是否可以移动的逻辑，这里判断了坐标点是否有效、是否已经在结果列表里了、还有行动力是否足够的判断。关于行动力的判断逻辑，这里做了几个功能拓展，一个是地形消耗行动力的配置，一个是角色擅长和不擅长地形的配置，这两个功能都能起到丰富游戏玩法的作用：
```javascript
/**
 * 检查指定坐标点能否移动
 * @param {MapPos} pos 目标坐标点
 * @param {MapPos[]} resultPos 结果列表
 * @param {number} limit 行动力
 */
checkMapPos(pos: MapPos, resultPos: MapPos[], limit: number) {
    // 判断该点是否有效、是否以加入可行动队列、行动力是否足够
    if (pos.x > 0 && pos.y > 0) {
        let targetPos = this._mapData.get(pos.x + "_" + pos.y);
        if (targetPos) {
            let newPos = resultPos.find((p: MapPos) => {
                return (p.x === pos.x && p.y === pos.y);
            });
            if (!newPos) {
                let value = pos.limit + this.getStepByType(targetPos) + this.getTargetStepByType(targetPos);
                if (value <= 0) {
                    // 如果计算的最终步数小于等于0，那么移动最小步数。
                    value = this._minStep;
                }
                if (value <= limit) {
                    pos.limit = value;
                    pos.status = MapPosStatus.CAN_MOVE;
                    resultPos.push(pos);
                } else {
                    pos.status = MapPosStatus.CAN_NOT_MOVE;
                    this._canntList.push(pos);
                }
            }
        } else {
            console.log("位置:", pos.x, pos.y, "值为:", targetPos, "不能移动");
        }
    }
}
```
以上就是战旗计算可移动范围的核心逻辑了，完整可运行的代码在[CocosDemo的FireEmblem例子](https://github.com/yue19870813/CocosDemo)里查看。由于花费的时间比较短，所以难免出现一些bug等纰漏，如果发现还请指出，谢谢。

#### 最后
当我们得到了可移动范围后，如果还想表现移动过程，那么我们可以使用A*算法来实现移动路线的功能。除此之外，战旗类游戏除了本文提到的《火纹》新作这种四向地格移动的形式外，还有《英雄无敌》中六边形六向移动的表现形式，注意思路应该和4向的差不多，后面有机会我会再实现一版6向或者更复杂的8向的来分享给大家。最后再安利一下《火焰纹章：风花雪月》这款作品，真的值得体验一下。