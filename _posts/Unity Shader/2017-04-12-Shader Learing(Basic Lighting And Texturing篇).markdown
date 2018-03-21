---
layout: post
title:  "[Unity Shader]Shader Learing(Basic Lighting And Texturing篇)"
date:   2017-04-12 23:34:10
categories: Unity Shader
comments: true
---

顶点着色程序和片段着色程序的基本功能和数据输入输出，实际上现在的着色程序已经可以接受多种数据类型，并灵活的进行各种算法的处理，如，可以接受光源信息（光源位置、强度等）、材质信息（反射系数、折射系数等）、运动控制信息（纹理投影矩阵、顶点运动矩阵等），可以在顶点程序中计算光线的折射方向，并传递到片段程序中进行光照计算。

通常我们在应用程序涉及的顶点位置和法向量都是三元向量，使用时将三元向量便为四元向量，顶点位置坐标传入顶点着色程序中转化为四元向量，最后一元数据为 1 ，而顶点法向量传入顶点着色程序中转化为四元向量，最后一元数据为 0 。

### Lighting：
#### diffuse 
漫反射与 Lambert 模型粗糙的物体表面向各个方向等强度地反射光，这种等同地向各个方向散射的现象称为光的漫反射( diffuse reflection)。产生光的漫反射现象的物体表面称为理想漫反射体，也称为兰伯特(Lambert )反射体。

当方向光照射到朗伯反射体上时，漫反射光的光强与入射光的方向和入射点表面法向夹角的余弦成正比，这称之为 Lambert 定律。
 
![图片](http://owk5gjdrg.bkt.clouddn.com/0053Shader%20Learing%E5%88%86%E4%BA%AB%E7%AC%AC4%E7%AF%87%28Basic%20Lighting%20And%20Texturing%E7%AF%87%29.png)

漫反射利用了向量的点积公式N•L = |N|*|L|*cosθ，可以求出反射光照：

diffuse = Kd * lightColor * max(N•L,0)。

下面分别以代码实现Vertex，Fragment和半Lambert漫反射：

```java
Shader "Custom/015_DiffuseVertex" {  
    Properties{  
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)//材质漫反射颜色  
    }  
        SubShader{  
            Pass{  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
            #pragma vertex vert  
            #pragma fragment frag  
                  
            fixed4 _Diffuse;  
            //application to vertex  
            struct a2v {  
                float4 vertex:POSITION;  
                float3 normal:NORMAL;  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                fixed3 color : COLOR;  
            };  
            v2f vert(a2v v) {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                //nvidia：  
                //Returns the normalized version of a vector, meaning a vector   
                //in the same direction as the original vector   
                //but with a Euclidean length of one  
                //法线的变换与顶点的变换是有区别的（非等比例缩放）  
                fixed3 normalDir = normalize( mul( v.normal, (float3x3) _World2Object) );  
                fixed3 lightDir =  normalize( _WorldSpaceLightPos0.xyz);  
                fixed3 diffuse = _LightColor0.rgb * max(dot(normalDir, lightDir), 0) *_Diffuse.rgb;    
                f.color = diffuse + ambient;  
                return f;  
            }   
            fixed4 frag(v2f f) :SV_Target//SV_Target (render target pixel color)   
            {  
                return fixed4(f.color,1);  
            }  
            ENDCG  
        }  
    }  
    Fallback "Diffuse"  
}  
```

```java
Shader "Custom/016_DiffuseFragment" {  
    Properties{  
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)  
    }  
        SubShader{  
            Pass{  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
  
            #pragma vertex vert  
            #pragma fragment frag     
            fixed4 _Diffuse;  
            //application to vertex  
            struct a2v {  
                float4 vertex:POSITION;  
                float3 normal:NORMAL;  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                fixed3 worldNormalDir : TEXCOORD0;  
            };  
            v2f vert(a2v v) {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                f.worldNormalDir = mul(v.normal, (float3x3) _World2Object);  
                return f;  
            }  
            fixed4 frag(v2f f) :SV_Target{  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                fixed3 normalDir = normalize(f.worldNormalDir);  
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);  
                fixed3 diffuse = _LightColor0.rgb * max(dot(normalDir, lightDir), 0) *_Diffuse.rgb;   
                fixed3 tempColor = diffuse + ambient;  
                return fixed4(tempColor,1);  
            }  
            ENDCG  
        }  
    }  
    Fallback "Diffuse"  
}  
```

```java
Shader "Custom/017_HalfLambert" {  
    Properties{  
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)  
    }  
        SubShader{  
            Pass{  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
            #pragma vertex vert  
            #pragma fragment frag  
                  
            fixed4 _Diffuse;  
            //application to vertex  
            struct a2v {  
                float4 vertex:POSITION;  
                float3 normal:NORMAL;  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                fixed3 worldNormalDir : TEXCOORD0;  
            };  
            v2f vert(a2v v) {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                f.worldNormalDir = mul(v.normal, (float3x3) _World2Object);  
                return f;  
            }  
            fixed4 frag(v2f f) :SV_Target{  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                fixed3 normalDir = normalize(f.worldNormalDir);  
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);  
                float halfLambert = dot(normalDir, lightDir) *0.5 +0.5  ;  
                fixed3 diffuse = _LightColor0.rgb * halfLambert  *_Diffuse.rgb;  
                fixed3 tempColor = diffuse + ambient;  
                return fixed4(tempColor,1);  
            }  
            ENDCG  
        }  
    }  
    Fallback "Diffuse"  
}  
```

#### Specular
镜面反射（高光），是光线经过物体表面，反射到视野中，当反射光线与人的眼睛看得方向平行时，强度最大，高光效果最明显，夹角为90度时，强度最小。
　　
specular = I*R*V；
　　
specular：反射光经过物体表面反射后进入人眼的光强；

I：反射光的光强，Phong依然是理想模型，所以不考虑光的衰减，即反射光的光强和入射光的光强相等；R：反射光线的方向；V：视线的方向；
 
![图片](http://owk5gjdrg.bkt.clouddn.com/0055Shader%20Learing%E5%88%86%E4%BA%AB%E7%AC%AC4%E7%AF%87%28Basic%20Lighting%20And%20Texturing%E7%AF%87%29.png)
 
下面分别以代码实现Vertex，Fragment和Blinn-Phong高光反射：

```java
Shader "Custom/018_SpecularVertex" {  
    Properties{  
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)  
        _Specular("Specular Color",Color)=(1,1,1,1)  
        _Gloss("Gloss",Range(8,200)) = 10  
    }  
        SubShader{  
            Pass{  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
            #pragma vertex vert  
            #pragma fragment frag  
            fixed4 _Diffuse;  
            fixed4 _Specular;  
            half _Gloss;      
            //application to vertex  
            struct a2v {  
            float4 vertex:POSITION;  
            float3 normal:NORMAL;  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                fixed3 color : COLOR;  
            };   
            v2f vert(a2v v) {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                fixed3 normalDir = normalize( mul( v.normal, (float3x3) _World2Object) );  
                fixed3 lightDir =  normalize( _WorldSpaceLightPos0.xyz);  
                fixed3 diffuse = _LightColor0.rgb * max(dot(normalDir, lightDir), 0) *_Diffuse.rgb;   
                fixed3 reflectDir = normalize(reflect(-lightDir, normalDir));  
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(v.vertex, _World2Object).xyz);  
                fixed3 specular = _LightColor0.rgb *  _Specular.rgb * pow(max(dot(reflectDir, viewDir), 0), _Gloss);  
                f.color = diffuse + ambient + specular;  
                return f;  
            }   
            fixed4 frag(v2f f) :SV_Target{  
                return fixed4(f.color,1);  
            }  
            ENDCG  
        }  
    }  
    Fallback "Diffuse"  
}  
```

```java
Shader "Custom/019_SpecularFragment" {  
    Properties{  
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)  
        _Specular("Specular Color",Color)=(1,1,1,1)  
        _Gloss("Gloss",Range(8,200)) = 10  
    }  
        SubShader{  
            Pass{  
  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
            #pragma vertex vert  
            #pragma fragment frag  
            fixed4 _Diffuse;  
            fixed4 _Specular;  
            half _Gloss;  
            //application to vertex  
            struct a2v {  
                float4 vertex:POSITION;  
                float3 normal:NORMAL;  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                float3 worldNormal : TEXCOORD0;  
                float3 worldVertex :TEXCOORD1;  
            };   
            v2f vert(a2v v) {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                f.worldNormal = mul(v.normal, (float3x3) _World2Object);  
                f.worldVertex = mul(_World2Object,v.vertex).xyz;  
                return f;  
            }   
            fixed4 frag(v2f f) :SV_Target{  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                fixed3 normalDir = normalize(f.worldNormal);  
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);  
                fixed3 diffuse = _LightColor0.rgb * max(dot(normalDir, lightDir), 0) *_Diffuse.rgb;  
                fixed3 reflectDir = normalize(reflect(-lightDir, normalDir));  
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - f.worldVertex );  
                fixed3 specular = _LightColor0.rgb *  _Specular.rgb * pow(max(dot(reflectDir, viewDir), 0), _Gloss);  
                fixed3 tempColor = diffuse + ambient + specular;  
                return fixed4(tempColor,1);  
            }  
        ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

```java
Shader "Custom/020_SpecularFragmentBlinnPhong" {  
Properties{  
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)  
        _Specular("Specular Color",Color)=(1,1,1,1)  
        _Gloss("Gloss",Range(8,200)) = 10  
    }  
        SubShader{  
            Pass{  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
            #pragma vertex vert  
            #pragma fragment frag  
            fixed4 _Diffuse;  
            fixed4 _Specular;  
            half _Gloss;      
            struct a2v   
            {  
            float4 vertex:POSITION;  
            float3 normal:NORMAL;  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                float3 worldNormal : TEXCOORD0;  
                float4 worldVertex :TEXCOORD1;  
            };   
            v2f vert(a2v v)   
            {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                f.worldNormal = mul(v.normal, (float3x3) _World2Object);  
                f.worldVertex = mul(_World2Object,v.vertex);  
                return f;  
            }   
            fixed4 frag(v2f f) :SV_Target  
            {  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                fixed3 normalDir = normalize(f.worldNormal);  
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);  
              
                fixed3 diffuse = _LightColor0.rgb * max(dot(normalDir, lightDir), 0) *_Diffuse.rgb;  
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - f.worldVertex );  
                fixed3 halfDir = normalize(viewDir + lightDir);  
                fixed3 specular = _LightColor0.rgb *  _Specular.rgb * pow(max(dot(normalDir, halfDir), 0), _Gloss);  
                fixed3 tempColor = diffuse + ambient + specular;  
                return fixed4(tempColor,1);  
            }  
        ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

### Texturing

贴图，使用一张纹理，代替物体的漫反射颜色。

```java
Shader "Custom/021_SingleTexture" {  
Properties{  
        _Color("Color",Color)=(1,1,1,1)  
        _Specular("Specular Color",Color)=(1,1,1,1)  
        _Gloss("Gloss",Range(8,200)) = 10  
        _MainTex("Main Tex",2D) = "white"{}//声明纹理代替谩发射颜色(全白纹理)  
    }  
        SubShader{  
            Pass{  
  
            Tags{ "LightMode" = "ForwardBase" }  
            CGPROGRAM  
            #include "UnityCG.cginc"  
            #include "Lighting.cginc"  
  
            #pragma vertex vert  
            #pragma fragment frag  
  
            fixed4 _Specular;  
            half _Gloss;  
            sampler2D _MainTex;  
            float4 _MainTex_ST;//纹理类型的属性声明(纹理名_ST)  
            fixed4 _Color;  
  
            struct a2v   
            {  
            float4 vertex:POSITION;  
            float3 normal:NORMAL;  
            float4 texcoord:TEXCOORD0;  
            };  
  
            struct v2f {  
                float4 position:SV_POSITION;  
                float3 worldNormal : TEXCOORD0;  
                float4 worldVertex :TEXCOORD1;  
                float2 uv:TEXCOORD2;  
                  
            };   
                  
            v2f vert(a2v v)   
            {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                f.worldNormal = mul(v.normal, (float3x3) _World2Object);  
                f.worldVertex = mul(v.vertex, _World2Object);  
                //对顶点纹理坐标进行变换，得到最终的纹理坐标  
                f.uv = v.texcoord.xy * _MainTex_ST.xy  + _MainTex_ST.zw;  
                return f;  
            }   
  
            fixed4 frag(v2f f) :SV_Target  
            {  
                //材质的反射率 = tex2D纹理采样函数  
                fixed3 texColor = tex2D(_MainTex, f.uv.xy)*_Color.rgb;  
  
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb;  
                fixed3 normalDir = normalize(f.worldNormal);  
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);  
                  
                fixed3 diffuse = _LightColor0.rgb * max(dot(normalDir, lightDir), 0) *texColor.rgb;  
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - f.worldVertex );  
                fixed3 halfDir = normalize(viewDir + lightDir);  
                fixed3 specular = _LightColor0.rgb *  _Specular.rgb * pow(max(dot(normalDir, halfDir), 0), _Gloss);  
                fixed3 tempColor = diffuse + ambient*texColor + specular;  
                return fixed4(tempColor,1);  
            }  
        ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

法线贴图：使用的是底模(面数较少的模型),而通过其它的一些技术手段来达到相似的效果。

```java
Shader "Custom/022_NormalMap"   
{  
    Properties   
    {  
        _MainTex ("Base (RGB)", 2D) = "white" {}  
        _NormalMap("NormalMap",2D) = "Bump"{}  
        _SpecColor("SpecularColor",Color) = (1,1,1,1)  
        _Shininess("Shininess",Float) = 10  
    }  
    SubShader   
    {  
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
                float3 binormal= cross(v.normal,v.tangent.xyz) * v.tangent.w;  
//tangent.w决定第三个坐标轴，值为正负1  
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

效果对比：

![图片](http://owk5gjdrg.bkt.clouddn.com/0056Shader%20Learing%E5%88%86%E4%BA%AB%E7%AC%AC4%E7%AF%87%28Basic%20Lighting%20And%20Texturing%E7%AF%87%29.png)

