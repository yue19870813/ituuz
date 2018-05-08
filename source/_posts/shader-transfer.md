---
title: 在cocos creator中用shader实现图片的渐变切换
date: 2018-05-02 20:07:12
tags: shader
categories: shader
---
# 在cocos creator中用shader实现图片的渐变切换
## 功能介绍
很多时候我们需要使用在场景中进行背景切换，或者图片的过渡等，需要一下过渡的效果，很多引擎都没有提供这种切换效果，使用遮罩来实现效果也并不是很好，所以这里使用shader来实现一下这个效果。本文中的shader是基于cocos creator来实现的，其中使用了cocos内置的变量，不过shader是基于标准的GLSL语言写的，很容易移植到其他引擎或者平台的。

## 效果图如下
![shader](/images/shader-transfer-1.gif)
![shader](/images/shader-transfer-2.gif)
<!--more-->

## shader实现方案
在图片中选取一条垂直竖线，竖线左边透明度逐渐变高，同时在update中不断调整竖线的位置，从最左边移动到最右边就实现了一个渐变的过渡切换效果，透明度渐变程度和渐变移动速度，包括渐变方向都可以自己修改参数调整。

## 代码实现
```
// 顶点着色器，这里什么修改也不做。
public static TRANSFER_VERT:string = `
    attribute vec4 a_position;
    attribute vec2 a_texCoord;
    attribute vec4 a_color;
    varying vec4 v_fragmentColor; 
    varying vec2 v_texCoord; 
    void main() 
    { 
        gl_Position = CC_PMatrix * a_position;
        v_fragmentColor = a_color; 
        v_texCoord = a_texCoord; 
    }
    `;

// 片段着色器，主要逻辑都是在片段着色器上实现
public static TRANSFER_FRAG:string = `
    #ifdef GL_ES
    precision lowp float;
    #endif
    
	/**
	 * Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式，并且它是全局的，也就是说我们可以在代码中来修改这个变量。 
	 * 这里我们使用这个特性来定义一个time，time值决定我们上面说的垂直竖线的位置，再通过外部访问来修改time的值，来达到移动的目的。
	 */
    uniform float time;

    varying vec4 v_fragmentColor;
    varying vec2 v_texCoord;
    void main()
    {
        vec4 c = v_fragmentColor * texture2D(CC_Texture0, v_texCoord);
        gl_FragColor = c;

        float temp = v_texCoord.x - time;
        if (temp <= 0.0) {
            float temp2 = abs(temp);
            if (temp2 <= 0.2) {
                gl_FragColor.w = 1.0 - temp2/0.2;
            } else {
                gl_FragColor.w = 0.0;
            }
        } else {
            gl_FragColor.w = 1.0;
        }
    }
    `;
	
// 下面开始是cocos的代码，来测试shader。
private _program:any;
private _time:number = 0;
    
start () {
    this.testShader();
}

testShader() {
    let bgSp:cc.Sprite = this.bgNode.getChildByName("bg").getComponent(cc.Sprite);
    this._program = new cc.GLProgram();
    
    if (cc.sys.isNative) {  
        this._program.initWithString(StoryHandlerScript.TRANSFER_VERT, StoryHandlerScript.TRANSFER_FRAG);
    } else {  
        this._program.initWithVertexShaderByteArray(StoryHandlerScript.TRANSFER_VERT, StoryHandlerScript.TRANSFER_FRAG);
        this._program.addAttribute(cc.macro.ATTRIBUTE_NAME_POSITION, cc.macro.VERTEX_ATTRIB_POSITION);  
        this._program.addAttribute(cc.macro.ATTRIBUTE_NAME_COLOR, cc.macro.VERTEX_ATTRIB_COLOR);  
        this._program.addAttribute(cc.macro.ATTRIBUTE_NAME_TEX_COORD, cc.macro.VERTEX_ATTRIB_TEX_COORDS);  
    }
    this._program.link();  
    this._program.updateUniforms();
    this._program.use();

    if (cc.sys.isNative) {  
        var glProgram_state = cc.GLProgramState.getOrCreateWithGLProgram(this._program);
        glProgram_state.setUniformFloat( "time", this._time );    
    } else {
        let time = this._program.getUniformLocationForName("time");
        this._program.setUniformLocationWith1f(time, this._time);
    }
    bgSp._sgNode.setShaderProgram(this._program);
}

// 在update里来修改time的值
update(dt){
    this._time += 0.02;
    if (this._program) {
        this._program.use();
        if (cc.sys.isNative) {
            var glProgram_state = cc.GLProgramState.getOrCreateWithGLProgram(this._program);
            glProgram_state.setUniformFloat( "time", this._time );    
        } else {
            let time = this._program.getUniformLocationForName("time");
            this._program.setUniformLocationWith1f(time, this._time);
        }
    }
}
```
以上就是所有的代码实现，逻辑很简单，同时有一些简单的注释，应该很好理解。