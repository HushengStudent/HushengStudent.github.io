---
layout: post
title:  "[Unity Shader]Shader Learing(Transparent Shader篇)"
date:   2017-05-11 00:00:10
categories: UnityShader
comments: true
---

透明物体的写法：

一种是clip，写zwrite，好处是不会排序混乱，坏处是没法blend，即透明度测试，透明度测试不能实现半透，只会删除一些片元，看上去像镂空一些面。(解释：设定一个透明度的值，如a，当物体的透明度>a时，按透明处理，直接舍弃，当透明度<a时，按正常物体处理，及深度测试，深度写入，然后深度值要小的物体就被舍弃掉了，不会渲染，也就是透明物体之后的物体不会被渲染了)。

另一种就是blend，不写zwrite，这时候就完全依赖于排序，即透明度混合。（解释：必须先渲染不透明物体，再渲染透明物体，再进行深度测试，被不透明物体挡住的就不会被渲染，挡住不透明物体的，就能正确进行颜色的混合）。

如果你有一个物体里面的面片互相穿插，可以使用两个pass，一个写深度，另外一个做alpha blending。

### 渲染顺序Queue：

Background 1000 通常用这个队列渲染需要绘制在背景上的物体

Geometry  2000 默认值；大部分物体在这个队列。不透明的物体也在这里

AlphaTest 2450 需要透明度测试物体使用的队列

Transparent 3000 在Geometry和AlphaTest渲染后，再从后往前渲染。透明度混合物体都该用

Overlay 4000 实现叠加效果[如镜头光晕]。任何需要最后渲染的，用这个。

自定义 x 用户可以定义任意值，比如"Queue"="Geometry+10"

#### RenderType[将Shader分类，用于Shader替换]：

Opaque       绝大部分不透明的物体都使用这个

Transparent   绝大部分透明的物体、包括粒子特效都使用这个

TransparentCutout蒙皮透明着色器

Background   天空盒都使用这个

Overlay       GUI/镜头光晕都使用这个

#### Alpha Blending指令：

Blend Off 关混合

Blend SrcFactor DstFactor

Blend SrcFactor DstFactor, srcFactorA DstFactorA

Blend {code for SrcFactor片元颜色} {code for DstFactor已经在颜色缓冲中的颜色}

resulting RGBA color：

float4 result = SrcFactor * fragment_output + DstFactor * pixel_color;

One                            float4(1.0, 1.0, 1.0, 1.0)

Zero                           float4(0.0, 0.0, 0.0, 0.0)

SrcColor                    fragment_output

SrcAlpha                      fragment_output.aaaa

DstColor                    pixel_color

DstAlpha                    pixel_color.aaaa

OneMinusSrcColor    float4(1.0, 1.0, 1.0, 1.0) - fragment_output

OneMinusSrcAlpha     float4(1.0, 1.0, 1.0, 1.0) - fragment_output.aaaa

OneMinusDstColor    float4(1.0, 1.0, 1.0, 1.0) - pixel_color

OneMinusDstAlpha    float4(1.0, 1.0, 1.0, 1.0) - pixel_color.aaaa
pixel_color.aaaa=float4(pixel_color.a, pixel_color.a, pixel_color.a, pixel_color.a)

Blend SrcAlpha OneMinusSrcAlpha
float4 result = fragment_output.aaaa * fragment_output + (float4(1.0, 1.0, 1.0, 1.0) - fragment_output.aaaa) * pixel_color

Blend One OneMinusSrcAlpha 
float4 result = float4(1.0, 1.0, 1.0, 1.0) * fragment_output + (float4(1.0, 1.0, 1.0, 1.0) - fragment_output.aaaa) * pixel_color

Blend One One
float4 result = float4(1.0, 1.0, 1.0, 1.0) * fragment_output + float4(1.0, 1.0, 1.0, 1.0) * pixel_color

#### AlphaTest

```java
Shader "Custom/023_AlphaTest" {  
    Properties   
    {  
        _MainTex ("Base (RGB)", 2D) = "white" {}  
        _NormalMap("NormalMap",2D) = "Bump"{}  
        _SpecColor("SpecularColor",Color) = (1,1,1,1)  
        _Shininess("Shininess",Float) = 10  
  
        _Cutoff ("Alpha Cutoff", Range(0, 1)) = 0.5  
    }  
    SubShader   
    {  
        Tags {"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}  
  
        Pass  
        {  
            Tags { "LightMode"="ForwardBase" }  
            CGPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
            #include "UnityCG.cginc"  
            uniform sampler2D _MainTex;  
            uniform sampler2D _NormalMap;  
            uniform float4 _SpecColor;  
            //控制凸起程度  
            uniform float _Shininess;  
            uniform float4 _LightColor0;  
  
            fixed _Cutoff;  
  
            struct VertexOutput   
            {  
                float4 pos:SV_POSITION;  
                float2 uv:TEXCOORD0;  
                float3 lightDir:TEXCOORD1;  
                float3 viewDir:TEXCOORD2;  
            };  
  
            VertexOutput vert(appdata_tan v)  
            {  
                VertexOutput o;  
                o.pos = mul(UNITY_MATRIX_MVP,v.vertex);  
                o.uv = v.texcoord.xy;  
                //得到Object2TangentMatrix矩阵  
                //这四行代码可以使用Unity定义的一个宏TANGENT_SPACE_ROTATION来代替  
                float3 normal = v.normal;  
                float3 tangent = v.tangent;  
                float3 binormal= cross(v.normal,v.tangent.xyz) * v.tangent.w;//tangent.w决定第三个坐标轴，值为正负1  
                float3x3 Object2TangentMatrix = float3x3(tangent,binormal,normal);  
                //之所以变换到切线空间，是为了减少片段着色器的计算量  
                o.lightDir = mul(Object2TangentMatrix,ObjSpaceLightDir(v.vertex));  
                o.viewDir = mul(Object2TangentMatrix,ObjSpaceViewDir(v.vertex));  
                return o;  
            }  
  
            float4 frag(VertexOutput input):COLOR  
            {  
                float3 lightDir = normalize(input.lightDir);  
                float3 viewDir = normalize(input.viewDir);  
                float4 encodedNormal = tex2D(_NormalMap,input.uv);  
                //法线[-1,1],颜色[0,1]  
                float3 normal = float3(2.0*encodedNormal.ag - 1,0.0);  
                //保证normal为正  
                normal.z = sqrt(1 - dot(normal,normal));  
                float4 texColor = tex2D(_MainTex,input.uv);  
                  
                clip (texColor.a -_Cutoff*0.2);  
  
                float3 ambient = texColor.rgb * UNITY_LIGHTMODEL_AMBIENT.rgb;  
                float3 diffuseReflection = texColor.rgb * _LightColor0.rgb * max(0,dot(normal,lightDir));  
                float facing;  
                if(dot(normal,lightDir)<0)  
                {  
                    facing = 0;  
                }  
                else  
                {  
                    facing = 1;  
                }  
                float3 specularRelection =   
                _SpecColor.rgb * _LightColor0.rgb * facing * pow(max(0,dot(reflect(-lightDir,normal),viewDir)),_Shininess);  
                return float4(ambient + diffuseReflection + specularRelection,1);  
            }  
            ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

效果图如下：

![图片](http://owk5gjdrg.bkt.clouddn.com/0057Shader%20Learing%28Transparent%20Shader%E7%AF%87%29.png)

#### AlphaBlend
```java
//AlphaBlend  
Shader "Custom/024_AlphaBlend" {  
  
    Properties   
    {  
        _MainTex ("Base (RGB)", 2D) = "white" {}  
        _NormalMap("NormalMap",2D) = "Bump"{}  
        _SpecColor("SpecularColor",Color) = (1,1,1,1)  
        _Shininess("Shininess",Float) = 10  
  
        _AlphaScale ("Alpha Scale", Range(0, 1)) = 1  
  
    }  
    SubShader   
    {  
        Tags {"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}  
  
        Pass  
        {  
            Tags { "LightMode"="ForwardBase" }  
  
            ZWrite Off  
            Blend SrcAlpha OneMinusSrcAlpha  
            //float4 result = fragment_output.aaaa * fragment_output + (float4(1.0, 1.0, 1.0, 1.0) - fragment_output.aaaa) * pixel_color  
            CGPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
            #include "UnityCG.cginc"  
            uniform sampler2D _MainTex;  
            uniform sampler2D _NormalMap;  
            uniform float4 _SpecColor;  
            //控制凸起程度  
            uniform float _Shininess;  
            uniform float4 _LightColor0;  
  
            fixed _AlphaScale;  
  
            struct VertexOutput   
            {  
                float4 pos:SV_POSITION;  
                float2 uv:TEXCOORD0;  
                float3 lightDir:TEXCOORD1;  
                float3 viewDir:TEXCOORD2;  
            };  
  
            VertexOutput vert(appdata_tan v)  
            {  
                VertexOutput o;  
                o.pos = mul(UNITY_MATRIX_MVP,v.vertex);  
                o.uv = v.texcoord.xy;  
                //得到Object2TangentMatrix矩阵  
                //这四行代码可以使用Unity定义的一个宏TANGENT_SPACE_ROTATION来代替  
                float3 normal = v.normal;  
                float3 tangent = v.tangent;  
                float3 binormal= cross(v.normal,v.tangent.xyz) * v.tangent.w;//tangent.w决定第三个坐标轴，值为正负1  
                float3x3 Object2TangentMatrix = float3x3(tangent,binormal,normal);  
                //之所以变换到切线空间，是为了减少片段着色器的计算量  
                o.lightDir = mul(Object2TangentMatrix,ObjSpaceLightDir(v.vertex));  
                o.viewDir = mul(Object2TangentMatrix,ObjSpaceViewDir(v.vertex));  
                return o;  
            }  
  
            float4 frag(VertexOutput input):COLOR  
            {  
                float3 lightDir = normalize(input.lightDir);  
                float3 viewDir = normalize(input.viewDir);  
                float4 encodedNormal = tex2D(_NormalMap,input.uv);  
                //法线[-1,1],颜色[0,1]  
                float3 normal = float3(2.0*encodedNormal.ag - 1,0.0);  
                //保证normal为正  
                normal.z = sqrt(1 - dot(normal,normal));  
                float4 texColor = tex2D(_MainTex,input.uv);  
                  
                //clip (texColor.a -_Cutoff*0.2);  
  
                float3 ambient = texColor.rgb * UNITY_LIGHTMODEL_AMBIENT.rgb;  
                float3 diffuseReflection = texColor.rgb * _LightColor0.rgb * max(0,dot(normal,lightDir));  
                float facing;  
                if(dot(normal,lightDir)<0)  
                {  
                    facing = 0;  
                }  
                else  
                {  
                    facing = 1;  
                }  
                float3 specularRelection =   
                _SpecColor.rgb * _LightColor0.rgb * facing * pow(max(0,dot(reflect(-lightDir,normal),viewDir)),_Shininess);  
                return float4(ambient + diffuseReflection + specularRelection,min(1,texColor.a*_AlphaScale*50));  
            }  
            ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

效果图如下：

![图片](http://owk5gjdrg.bkt.clouddn.com/0058Shader%20Learing%28Transparent%20Shader%E7%AF%87%29.png)

#### One Pass ZWrite,Next Pass AlphaBlend
```java
Shader "Custom/025_AlphaBlendZWrite" {  
    Properties   
    {  
        _MainTex ("Base (RGB)", 2D) = "white" {}  
        _NormalMap("NormalMap",2D) = "Bump"{}  
        _SpecColor("SpecularColor",Color) = (1,1,1,1)  
        _Shininess("Shininess",Float) = 10  
  
        _AlphaScale ("Alpha Scale", Range(0, 1)) = 1  
  
    }  
    SubShader   
    {  
        Tags {"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}  
  
        Pass {  
            ZWrite On  
            ColorMask 0 //该pass不写入任何颜色通道  
        }  
        Pass  
        {  
            Tags { "LightMode"="ForwardBase" }  
  
            ZWrite Off  
            Blend SrcAlpha OneMinusSrcAlpha  
  
            CGPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
            #include "UnityCG.cginc"  
            uniform sampler2D _MainTex;  
            uniform sampler2D _NormalMap;  
            uniform float4 _SpecColor;  
            //控制凸起程度  
            uniform float _Shininess;  
            uniform float4 _LightColor0;  
  
            fixed _AlphaScale;  
  
            struct VertexOutput   
            {  
                float4 pos:SV_POSITION;  
                float2 uv:TEXCOORD0;  
                float3 lightDir:TEXCOORD1;  
                float3 viewDir:TEXCOORD2;  
            };  
  
            VertexOutput vert(appdata_tan v)  
            {  
                VertexOutput o;  
                o.pos = mul(UNITY_MATRIX_MVP,v.vertex);  
                o.uv = v.texcoord.xy;  
                //得到Object2TangentMatrix矩阵  
                //这四行代码可以使用Unity定义的一个宏TANGENT_SPACE_ROTATION来代替  
                float3 normal = v.normal;  
                float3 tangent = v.tangent;  
                float3 binormal= cross(v.normal,v.tangent.xyz) * v.tangent.w;//tangent.w决定第三个坐标轴，值为正负1  
                float3x3 Object2TangentMatrix = float3x3(tangent,binormal,normal);  
                //之所以变换到切线空间，是为了减少片段着色器的计算量  
                o.lightDir = mul(Object2TangentMatrix,ObjSpaceLightDir(v.vertex));  
                o.viewDir = mul(Object2TangentMatrix,ObjSpaceViewDir(v.vertex));  
                return o;  
            }  
  
            float4 frag(VertexOutput input):COLOR  
            {  
                float3 lightDir = normalize(input.lightDir);  
                float3 viewDir = normalize(input.viewDir);  
                float4 encodedNormal = tex2D(_NormalMap,input.uv);  
                //法线[-1,1],颜色[0,1]  
                float3 normal = float3(2.0*encodedNormal.ag - 1,0.0);  
                //保证normal为正  
                normal.z = sqrt(1 - dot(normal,normal));  
                float4 texColor = tex2D(_MainTex,input.uv);  
                  
                //clip (texColor.a -_Cutoff*0.2);  
  
                float3 ambient = texColor.rgb * UNITY_LIGHTMODEL_AMBIENT.rgb;  
                float3 diffuseReflection = texColor.rgb * _LightColor0.rgb * max(0,dot(normal,lightDir));  
                float facing;  
                if(dot(normal,lightDir)<0)  
                {  
                    facing = 0;  
                }  
                else  
                {  
                    facing = 1;  
                }  
                float3 specularRelection =   
                _SpecColor.rgb * _LightColor0.rgb * facing * pow(max(0,dot(reflect(-lightDir,normal),viewDir)),_Shininess);  
                return float4(ambient + diffuseReflection + specularRelection,min(1,texColor.a*_AlphaScale*50));  
            }  
            ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

效果图如下：

![图片](http://owk5gjdrg.bkt.clouddn.com/0059Shader%20Learing%28Transparent%20Shader%E7%AF%87%29.png)

