# UnityShader学习
## 基本学习
### 格式
``` shader
Shader "Custom/diffuse"
{    
    Properties
    {
    }
    SubShader
    {

        Pass
        {         
            CGPROGRAM
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```
### Properties
> 内联unity的变量
#### 例子
``` shader
  Properties
    {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }
```
#### 格式解释
||||
|-|-|-|
|shader代码中的变量名| ("unity中显示的变量名",变量类型)|=变量的值|

#### 常见变量类型
|类型名称|类型说明|值举例|
|-|-|-|
|Color|实际上是fixed4，但是unity会给一个调色板|(1,1,1,1)
|Vector|实际上是float4，四维向量|(0.1,0.2,0.3,0.4)
|Int|整形|123
|Float|浮点型|3.14
|Range(min,max)|实际上是浮点型,编辑器内会生成一个滑块|3.14
|2D|2D纹理("默认纯色图片"{})|"white"{}
|Cube|立方体纹理(天空盒)|同上
|3D|3D纹理|同上

### SubShader

> 子Shader,每一个shader文件中允许有多段子Shader,如果不兼容会向后引用
### 函数
> 用于实现某个功能，初期只有一个函数就可以
### CGPROGRAM
> Shader的主体，用于实现GPU的内联函数来实现功能
#### 例子
```
CGPROGRAM
#include "Lighting.cginc"
#pragma vertex vert
#pragma fragment frag
    struct vert_return{
        float4 position:SV_POSITION;
        fixed3 c:COLOR;
    };
    vert_return vert(float4 v:POSITION,float3 normal:NORMAL){
        vert_return r;
        r.position=UnityObjectToClipPos(v);
        fixed3 normaldir = normalize(mul(normal,(float3x3) unity_WorldToObject));
        fixed3 lightdir = normalize(_WorldSpaceLightPos0.xyz);
                
        fixed3 diffuse=_LightColor0.rgb*max(dot(normaldir,lightdir),0);
        r.c=diffuse;
        return r;
    }

    fixed4 frag(vert_return r):SV_TARGET0{
        return fixed4(r.c,1);
    }
ENDCG
```
#### CG中的变量解释
|类型名称|类型说明
|-|-|
float|32位浮点型
half|16位浮点型
fixed|-1~1
float2、float3、float4 |2维、3维、4维向量(其他的同理)


## 进阶学习
### struct 结构体
> 传参用，其实类似于python的字典传参
#### 格式
```
struct src{
   float4 postion:POSITION;
   fixed color:COLOR;
   ....
};//记住这里有分号！
```
### 顶点函数片元函数
#### 顶点函数
> 传入模型坐标系的坐标(f4)，返回裁剪面坐标系的坐标(f4) 

> 每个顶点参与计算
#### 片元函数
> 传入裁剪面坐标系的坐标(f4)，传出这个坐标对应的裁剪面像素的颜色值(f4) 

> 每个裁剪面像素参与计算

> 调用次数多余顶点函数
#### 代码
```
CGPROGRAM
#pragma vertex vert
#pragma fragment frag
struct src_vert{
    float4 pos:POSITION;
};
struct src_frag{
    float4 sv_pos:SV_POSITION;
};
src_frag vert(src_vert src){
    src_frag dst;
    dst.sv_pos=UnityObjectToClipPos(src.pos);
    return dst;
}
fixed4 frag(src_frag src):SV_Target {
    return fixed4(1,1,1,1);
}
ENDCG
```

### 漫反射
> 光照射到物体上，物体反射出的光在摄像机上的效果

