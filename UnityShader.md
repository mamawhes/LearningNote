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

### 漫反射Diffuse
> 光照射到物体上，物体反射出的光在摄像机上的效果
#### 思路
> 光直线照到某平面的点上最亮，意味着光沿着法线最亮；沿着切线射到点上最暗；如果大于90度，意味着该点再阴暗面。由此可得：光照向量与某点法向量的cos值为光照强度，如果该值为负，则光照强度为0
#### 公式
> diffuse_rate=max(dot(光照向量,点法向量),0)
#### 代码
``` shader
CGPROGRAM
#include "Lighting.cginc"
#pragma vertex vert
#pragma fragment frag
struct src_vert{
    float4 pos:POSITION;
    float4 normal:NORMAL;
};
struct src_frag{
    float4 sv_pos:SV_POSITION;
    float3 temp:COLOR;
};
fixed4 _Color;
src_frag vert(src_vert src){
    src_frag dst;
    dst.sv_pos=UnityObjectToClipPos(src.pos);//模型坐标转换
    fixed3 ambient_light = UNITY_LIGHTMODEL_AMBIENT.rgb;//获得环境光增量
    fixed3 light_vector = normalize(_WorldSpaceLightPos0.xyz);//获得光线向量
    fixed3 p_world_normal = normalize(mul(src.normal,(float3x3)unity_WorldToObject));//获得顶点法线向量
    fixed3 diffuse_rate = _LightColor0*max(dot(light_vector,p_world_normal),0)*_Color.rgb;//光线颜色*漫反射比率*物体颜色=漫反射颜色
    
    dst.temp=diffuse_rate+ambient_light;//漫反射+环境光
    return dst;
}
fixed4 frag(src_frag src):SV_Target {
    return fixed4(src.temp,1);
}
ENDCG
```
### 逐像素漫反射Diffuse Fragment
> 和上面基本上一样，只不过运算再片元函数中，传递的参数是顶点法向量

> 计算量大一点，教程说效果会好一点，自己实操感觉不如原来的
#### 代码
```
CGPROGRAM
#include "Lighting.cginc"
#pragma vertex vert
#pragma fragment frag
struct src_vert{
    float4 pos:POSITION;
    float4 normal:NORMAL;
};
struct src_frag{
    float4 sv_pos:SV_POSITION;
    float3 temp:COLOR;
};
fixed4 _Color;
src_frag vert(src_vert src){
    src_frag dst;
    dst.sv_pos=UnityObjectToClipPos(src.pos);
    dst.temp = normalize(mul(src.normal,(float3x3)unity_WorldToObject));
    return dst;
}
fixed4 frag(src_frag src):SV_Target {
    fixed3 ambient_light = UNITY_LIGHTMODEL_AMBIENT.rgb;
    fixed3 light_vector = normalize(_WorldSpaceLightPos0.xyz);
    fixed3 diffuse_rate = _LightColor0*max(dot(light_vector,src.temp),0)*_Color.rgb;
    fixed3 res=diffuse_rate+ambient_light;
    return fixed4(res,1);
}
ENDCG
```
### 半兰伯特（HalfLambert）光照模型
> 和漫反射差不多，区别是没有用max()和0比较，是直接获取cos值乘0.5后加0.5，这样也能保证光照增益在0——1，这样能保证不会出现纯黑的阴影面。
#### 代码
``` shader
CGPROGRAM
#include "Lighting.cginc"
#pragma vertex vert
#pragma fragment frag
struct src_vert{
    float4 pos:POSITION;
    float4 normal:NORMAL;
};
struct src_frag{
    float4 sv_pos:SV_POSITION;
    float3 temp:COLOR;
};
fixed4 _Color;
src_frag vert(src_vert src){
    src_frag dst;
    dst.sv_pos=UnityObjectToClipPos(src.pos);
    fixed3 ambient_light = UNITY_LIGHTMODEL_AMBIENT.rgb;
    fixed3 light_vector = normalize(_WorldSpaceLightPos0.xyz);
    fixed3 p_world_normal = normalize(mul(src.normal,(float3x3)unity_WorldToObject));
    fixed3 diffuse_rate = _LightColor0*(dot(light_vector,p_world_normal)*0.5+0.5)*_Color.rgb;
    
    dst.temp=diffuse_rate+ambient_light;
    return dst;
}
fixed4 frag(src_frag src):SV_Target {
    return fixed4(src.temp,1);
}
ENDCG
```
