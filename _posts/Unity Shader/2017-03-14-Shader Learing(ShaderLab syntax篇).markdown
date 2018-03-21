---
layout: post
title:  "[Unity Shader]Shader Learing(ShaderLab syntax篇)"
date:   2017-03-14 22:59:10
categories: Unity Shader
comments: true
---

#### 回顾：
回顾Unity中，数据的传递及Shader的工作原理：

![图片](http://owk5gjdrg.bkt.clouddn.com/0046Shader%20Learing%28ShaderLab%20syntax%E7%AF%87%29.png)

从上图我们可以看出，在渲染管线中，顶点着色器输入的数据经过顶点着色器后，顶点着色器会把相关数据传递给片段着色器，上图即表明了在Unity的顶点着色器和片段着色器中数据的传递类型和传递方向。

```java
/*  
齐次坐标表示是计算机图形学的重要手段之一，它既能够用来明确区分向量和点  
同时也更易用于进行仿射（线性）几何变换。  
*/  
Shader "Custom/004_RGBCube" {  
    SubShader {   
        Pass {   
        CGPROGRAM   
        #pragma vertex vert // vert function is the vertex shader   
        #pragma fragment frag // frag function is the fragment shader  
        // for multiple vertex output parameters an output structure   
        // is defined:  
        struct vertexOutput {  
           float4 pos : SV_POSITION;  
           float4 col : TEXCOORD0;  
        };  
        //POSITION语意用于顶点着色器，用来指定这些个位置坐标值，是在变换前的顶点的object space坐标。  
        //SV_POSITION语意则用于像素着色器，用来标识经过顶点着色器变换之后的顶点坐标。  
        vertexOutput vert(float4 vertexPos : POSITION)   
        // vertex shader   
        {  
           vertexOutput output; // we don't need to type 'struct' here  
           output.pos =  mul(UNITY_MATRIX_MVP, vertexPos);  
           output.col = vertexPos + float4(0.5, 0.5, 0.5, 0.0);  
              // Here the vertex shader writes output data  
              // to the output structure. We add 0.5 to the   
              // x, y, and z coordinates, because the   
              // coordinates of the cube are between -0.5 and  
              // 0.5 but we need them between 0.0 and 1.0.   
            return output;  
        }  
        float4 frag(vertexOutput input) : COLOR // fragment shader  
        {  
            return input.col;   
              // Here the fragment shader returns the "col" input   
              // parameter with semantic TEXCOORD0 as nameless  
              // output parameter with semantic COLOR.  
        }  
        ENDCG    
      }  
   }  
}  
```

#### ShaderBuilt-in Vertex Input：
我们知道Unity Shader编程中，可以使用结构体来把顶点输入数据，如上面的示例，其实在Unity Shader编程中，有很多内建的结构体可以让我们很方便的使用，下面使用Unity内建的结构体实现上面的示例：

```java
Shader "Custom/005_Build_InShader" {  
    SubShader {   
      Pass {   
         CGPROGRAM   
        #pragma vertex vert    
        #pragma fragment frag   
        #include "UnityCG.cginc"  
        struct vertexOutput {  
           float4 pos : SV_POSITION;  
           float4 col : TEXCOORD0;  
        };  
        vertexOutput vert(appdata_full input)   
        {  
           vertexOutput output;  
           output.pos =  mul(UNITY_MATRIX_MVP, input.vertex);  
           output.col = input.texcoord;  
  
           return output;  
        }  
        float4 frag(vertexOutput input) : COLOR   
        {  
           return input.col;   
        }  
        ENDCG    
     }  
   }  
}  
  
/*  
struct vertexInput {  
         float4 vertex : POSITION; // position (in object coordinates,   
                                 // i.e. local or model coordinates)  
         float4 tangent : TANGENT;    
                                // vector orthogonal to the surface normal  
         float3 normal : NORMAL; // surface normal vector (in object  
                                // coordinates; usually normalized to unit length)  
         float4 texcoord : TEXCOORD0;  // 0th set of texture   
                                    // coordinates (a.k.a. “UV”; between 0 and 1)   
         float4 texcoord1 : TEXCOORD1; // 1st set of tex. coors.   
         float4 texcoord2 : TEXCOORD2; // 2nd set of tex. coors.   
         float4 texcoord3 : TEXCOORD3; // 3rd set of tex. coors.   
         fixed4 color : COLOR; // color (usually constant)  
      };  
    预定义结构体：  
    pre-defined input structures appdata_base, appdata_tan, appdata_full  
    and appdata_img for the most common cases.   
    These are defined in the file UnityCG.cginc   
    (in the directory Unity > Editor > Data > CGIncludes):  
  
    struct appdata_base {  
      float4 vertex : POSITION;  
      float3 normal : NORMAL;  
      float4 texcoord : TEXCOORD0;  
    };  
    struct appdata_tan {  
      float4 vertex : POSITION;  
      float4 tangent : TANGENT;  
      float3 normal : NORMAL;  
      float4 texcoord : TEXCOORD0;  
    };  
    struct appdata_full {  
      float4 vertex : POSITION;  
      float4 tangent : TANGENT;  
      float3 normal : NORMAL;  
      float4 texcoord : TEXCOORD0;  
      float4 texcoord1 : TEXCOORD1;  
      float4 texcoord2 : TEXCOORD2;  
      float4 texcoord3 : TEXCOORD3;  
      fixed4 color : COLOR;  
      // and additional texture coordinates only on XBOX360  
    };  
    struct appdata_img {  
      float4 vertex : POSITION;  
      half2 texcoord : TEXCOORD0;  
    };  
*/  
```

uniform：

uniform变量一般用来表示：变换矩阵，材质，光照参数和颜色等信息。
```java
Shader "Custom/006_Uniforms" {  
    SubShader {  
      Pass {  
         CGPROGRAM  
        #pragma vertex vert    
        #pragma fragment frag   
        // uniform float4x4 _Object2World;   
        // automatic definition of a Unity-specific uniform parameter  
        struct vertexInput {  
           float4 vertex : POSITION;  
        };  
        struct vertexOutput {  
           float4 pos : SV_POSITION;  
           float4 position_in_world_space : TEXCOORD0;  
        };  
        vertexOutput vert(vertexInput input)   
        {  
            vertexOutput output;   
            output.pos =  mul(UNITY_MATRIX_MVP, input.vertex);  
            output.position_in_world_space =   
               mul(_Object2World, input.vertex);  
               // transformation of input.vertex from object   
               // coordinates to world coordinates;  
            return output;  
        }  
        float4 frag(vertexOutput input) : COLOR   
        {  
            float dist = distance(input.position_in_world_space,   
            float4(0.0, 0.0, 0.0, 1.0));  
               // computes the distance between the fragment position   
               // and the origin (the 4th coordinate should always be   
               // 1 for points).  
            if (dist < 5.0)  
            {  
               return float4(0.0, 1.0, 0.0, 1.0);   
                  // color near origin  
            }  
            else  
            {  
               return float4(0.1, 0.1, 0.1, 1.0);   
                  // color far from origin  
            }  
        }  
        ENDCG    
      }  
   }  
}  
/*  
uniform float4 _Time, _SinTime, _CosTime; // time values  
   uniform float4 _ProjectionParams;  
                              // x = 1 or -1 (-1 if projection is flipped)  
                              // y = near plane; z = far plane; w = 1/far plane  
   uniform float4 _ScreenParams;   
                        // x = width; y = height; z = 1 + 1/width; w = 1 + 1/height  
   uniform float3 _WorldSpaceCameraPos;  
   uniform float4x4 _Object2World;      // model matrix  
   uniform float4x4 _World2Object;     // inverse model matrix   
   uniform float4 _WorldSpaceLightPos0;   
   // position or direction of light source for forward rendering  
  
   uniform float4x4 UNITY_MATRIX_MVP;   // model view projection matrix   
   uniform float4x4 UNITY_MATRIX_MV;   // model view matrix  
   uniform float4x4 UNITY_MATRIX_V;   // view matrix  
   uniform float4x4 UNITY_MATRIX_P;   // projection matrix  
   uniform float4x4 UNITY_MATRIX_VP;   // view projection matrix  
   uniform float4x4 UNITY_MATRIX_T_MV;   
      // transpose of model view matrix  
   uniform float4x4 UNITY_MATRIX_IT_MV;   
      // transpose of the inverse model view matrix  
   uniform float4 UNITY_LIGHTMODEL_AMBIENT; // ambient color  
   */  
```

Uniform应用的第二个示例：
```java
Shader "Custom/007_UserSpecifiedUniforms" {  
    Properties {  
    _Point ("a point in world space", Vector) = (0., 0., 0., 1.0)  
    _DistanceNear ("threshold distance", Float) = 5.0  
    _ColorNear ("color near to point", Color) = (0.0, 1.0, 0.0, 1.0)  
    _ColorFar ("color far from point", Color) = (0.3, 0.3, 0.3, 1.0)  
   }  
   SubShader {  
      Pass {  
        CGPROGRAM  
        #pragma vertex vert    
        #pragma fragment frag   
        #include "UnityCG.cginc"   
            // defines _Object2World and _World2Object  
         // uniforms corresponding to properties  
        uniform float4 _Point;  
        uniform float _DistanceNear;  
        uniform float4 _ColorNear;  
        uniform float4 _ColorFar;  
        struct vertexInput {  
           float4 vertex : POSITION;  
        };  
        struct vertexOutput {  
           float4 pos : SV_POSITION;  
           float4 position_in_world_space : TEXCOORD0;  
        };  
        vertexOutput vert(vertexInput input)   
        {  
            vertexOutput output;   
            output.pos =  mul(UNITY_MATRIX_MVP, input.vertex);  
            output.position_in_world_space =   
               mul(_Object2World, input.vertex);  
            return output;  
        }  
        float4 frag(vertexOutput input) : COLOR   
        {  
            float dist = distance(input.position_in_world_space,   
               _Point);  
               // computes the distance between the fragment position   
               // and the position _Point.  
              
            if (dist < _DistanceNear)  
            {  
               return _ColorNear;   
            }  
            else  
            {  
               return _ColorFar;   
            }  
        }  
        ENDCG    
      }  
   }  
}  
  
/*  
Shader的属性可以在脚本中被使用：  
eg：  
GetComponent(Renderer).sharedMaterial.SetVector("_Point",   
      Vector4(1.0, 0.0, 0.0, 1.0));  
   GetComponent(Renderer).sharedMaterial.SetFloat("_DistanceNear",   
      10.0);  
   GetComponent(Renderer).sharedMaterial.SetColor("_ColorNear",   
      Color(1.0, 0.0, 0.0));  
   GetComponent(Renderer).sharedMaterial.SetColor("_ColorFar",   
      Color(1.0, 1.0, 1.0));  
*/  
```

这样，我们可以通过持有属性来达到修改的目的。

#### Pass（通道）：RenderSetup+固定功能着色器命令

Unity中一个SubShader（渲染方案）是由一个个Pass块来执行的。每个Pass都会消耗对应的一个DrawCall。在满足渲染效果的情况下尽可能地减少Pass的数量。

Pass（通道编程）

1个Pass块可以使一个几何物体被一次渲染。

Pass { [Name and Tags] [RenderSetup] [TextureSetup] }

最基础的pass命令包含有1个可选的渲染设置命令列表，及1个可选的可用纹理列表。

一个Pass 可以定义它的名字和任意数量的标签-name/value 字符串 将Pass的含义告诉给渲染引擎。

Render Setup 渲染设置

Pass设置了一系列的显卡状态，比如说alpha混合是否开启，是否允许雾化等等。 这些命令有：

![](http://owk5gjdrg.bkt.clouddn.com/0048Shader%20Learing%28ShaderLab%20syntax%E7%AF%87%29.png)

①Color color
```java
Shader "Custom/NewShader" {  
    SubShader {  
        Pass { Color (1,0,0,0) }  //将对象设置为固体颜色  
    }  
}  
```

②Material {Material Block}  定义一个使用顶点光照管线的材质
```java
Shader "Custom/NewShader" {  
    SubShader {  
        Pass {  
            Material {  //材质块用于定义对象的材质属性  
                Diffuse (1,1,1,1)  
                Ambient (1,1,1,1)  
            }  
            Lighting On  
        }  
    }  
} 
```

③Lighting On|Off     开启或关闭顶点光照
```java
Shader "Custom/NewShader" {  
    Properties {  
        _Color ("Main Color", COLOR) = (1,1,1,1)  
    }  
    SubShader {  
        Pass {  
            Material {  
                Diffuse [_Color]  
                Ambient [_Color]  
            }  
            Lighting On  
 //对于在材质块中定义的设置有任何影响，您必须启用灯光与灯光的命令。如果灯光是关闭的，颜色是直接从颜色命令。  
        }  
    }  
}  
```

④SeparateSpecular On|Off  此命令使镜面照明添加到着色通道的末端，所以镜面照明不受纹理的影响。

⑤ColorMaterial AmbientAndDiffuse|Emission  使用每个顶点颜色代替材质中设置的颜色。
```java
Shader "Custom/NewShader" {  
    Properties {  
        _Color ("Main Color", Color) = (1,1,1,0)  
        _SpecColor ("Spec Color", Color) = (1,1,1,1)  
        _Emission ("Emmisive Color", Color) = (0,0,0,0)  
        _Shininess ("Shininess", Range (0.01, 1)) = 0.7  
        _MainTex ("Base (RGB)", 2D) = "white" {}  
    }  
    SubShader {  
        Pass {  
            Material {  
                Diffuse [_Color]//Diffuse color: 漫反射颜色分量。这是一个对象的基本颜色。  
                Ambient [_Color]//Ambient color: 环境颜色分量。这是对象的颜色，在照明窗口中被环境光照射  
                Shininess [_Shininess]//Shininess number: 突出的清晰度，在1和0之间  
                Specular [_SpecColor]//Specular color: 对象的高光的颜色  
                Emission [_Emission]//Emission color: 物体不被任何光线击中时的颜色  
            }  
            Lighting On  
            SeparateSpecular On  
            SetTexture [_MainTex] {  
                Combine texture * primary DOUBLE, texture * primary  
            }  
        }  
    }  
}  
```

![图片](http://owk5gjdrg.bkt.clouddn.com/0049Shader%20Learing%28ShaderLab%20syntax%E7%AF%87%29.png)

⑥Cull Back|Front|Off     设置多边形剔除模式，控制多边形应该剔除(不绘制)的面
```java
Shader "Custom/NewShader" {  //对象只渲染对象的背面  
  SubShader {  
        Pass {  
            Material {  
                Diffuse (1,1,1,1)  
            }  
            Lighting On  
            Cull Front  
        }  
    }  
}  
```

⑦ZWrite On|Off     设置深度写模式，控制像素从这个对象是否写入深度缓冲
```java
Shader "Custom/NewShader" {  
Properties {  
    _Color ("Main Color", Color) = (1,1,1,1)  
    _MainTex ("Base (RGB) Trans (A)", 2D) = "white" {}  
}  
SubShader {  
    Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}  
    LOD 200  
    // extra pass that renders to depth buffer only  
    Pass {  
        ZWrite On     // 开启深度缓冲写  
        ColorMask 0   //关闭所有渲染染色通道  
    }  
    // paste in forward rendering passes from Transparent/Diffuse  
    UsePass "Transparent/Diffuse/FORWARD"  
}  
//Fallback "Transparent/VertexLit"  
}  
```

⑧ZTest Less|Greater|LEqual|GEqual|Equal|NotEqual|Always     设置深度测试模式

⑨Offset Factor, Units     允许您指定两个参数的深度偏移量
```java
Shader "Custom/NewShader" {  
    Properties {  
        _Color ("Main Color", Color) = (1,1,1,0)  
        _SpecColor ("Spec Color", Color) = (1,1,1,1)  
        _Emission ("Emmisive Color", Color) = (0,0,0,0)  
        _Shininess ("Shininess", Range (0.01, 1)) = 0.7  
        _MainTex ("Base (RGB)", 2D) = "white" { }  
    }  
    SubShader {  
        // We use the material in many passes by defining them in the subshader.  
        // Anything defined here becomes default values for all contained passes.  
        Material {  
            Diffuse [_Color]  
            Ambient [_Color]  
            Shininess [_Shininess]  
            Specular [_SpecColor]  
            Emission [_Emission]  
        }  
        Lighting On  
        SeparateSpecular On  
        // Set up alpha blending  
        Blend SrcAlpha OneMinusSrcAlpha  
        // Render the back facing parts of the object.  
        // If the object is convex, these will always be further away  
        // than the front-faces.  
        Pass {  
            Cull Front  
            SetTexture [_MainTex] {  
                Combine Primary * Texture  
            }  
        }  
        // Render the parts of the object facing us.  
        // If the object is convex, these will be closer than the  
        // back-faces.  
        Pass {  
            Cull Back  
            SetTexture [_MainTex] {  
                Combine Primary * Texture  
            }  
        }  
    }  
}  
```

![图片](http://owk5gjdrg.bkt.clouddn.com/0050Shader%20Learing%28ShaderLab%20syntax%E7%AF%87%29.png)

⑩SetTexture [TextureName] {Texture Block}
```java
Shader "Custom/NewShader" {  
    Properties {  
        _MainTex ("Base (RGB)", 2D) = "white" { }  
    }  
    SubShader {  
        // Render the front-facing parts of the object.  
        // We use a simple white material, and apply the main texture.  
        Pass {  
            Material {  
                Diffuse (1,1,1,1)  
            }  
            Lighting On  
            SetTexture [_MainTex] {  
                Combine Primary * Texture  
            }  
        }  
        // Now we render the back-facing triangles in the most  
        // irritating color in the world: BRIGHT PINK!  
        Pass {  
            Color (1,0,1,1)  
            Cull Front  
        }  
    }  
}  
//传统的纹理合成  
//Previous 是之前SetTexture的结果。          
//Primary 是从照明计算或顶点颜色的颜色。      
//Texture 纹理是在SetTexture TextureName指定纹理的颜色（见上图）。  
//Constant 是ConstantColor指定的颜色。  
```

⑪SetTexture [_MainTex] { combine previous * texture, previous + texture }

⑫Fog {Fog Commands}     设置雾参数

⑬Mode Off|Global|Linear|Exp|Exp2

⑭Color ColorValue

⑮Density FloatValue

⑯Range FloatValue, FloatValue

![图片](http://owk5gjdrg.bkt.clouddn.com/0051Shader%20Learing%28ShaderLab%20syntax%E7%AF%87%29.png)

⑰AlphaTest Off     开启alpha测试

⑱AlphaTest comparison AlphaValue
```java
Shader "Custom/NewShader" {  
    Properties {  
        _Color ("Main Color", Color) = (.5, .5, .5, .5)  
        _MainTex ("Base (RGB) Alpha (A)", 2D) = "white" {}  
        _Cutoff ("Base Alpha cutoff", Range (0,.9)) = .5  
    }  
    SubShader {  
        // Set up basic lighting  
        Material {  
            Diffuse [_Color]  
            Ambient [_Color]  
        }  
        Lighting On  
        // Render both front and back facing polygons.  
        Cull Off  
        // first pass:  
        // render any pixels that are more than [_Cutoff] opaque  
        Pass {  
            AlphaTest Greater [_Cutoff]  
            SetTexture [_MainTex] {  
                combine texture * primary, texture  
            }  
        }  
        // Second pass:  
        // render in the semitransparent details.  
        Pass {  
            // Dont write to the depth buffer  
            ZWrite off  
            // Don't write pixels we have already written.  
            ZTest Less  
            // Only render pixels less or equal to the value  
            AlphaTest LEqual [_Cutoff]  
            // Set up alpha blending  
            Blend SrcAlpha OneMinusSrcAlpha  
            SetTexture [_MainTex] {  
                combine texture * primary, texture  
            }  
        }  
    }  
}  
```

![图片](http://owk5gjdrg.bkt.clouddn.com/0052Shader%20Learing%28ShaderLab%20syntax%E7%AF%87%29.png)

⑲Blend Off     设置α混合模式

Blend SrcFactor DstFactor

Blend SrcFactor DstFactor, SrcFactorA DstFactorA

BlendOp BlendOp
```java
Shader "Custom/NewShader" {  
    Properties {  
        _Color ("Main Color", Color) = (1,1,1,1)  
        _MainTex ("Base (RGB) Transparency (A)", 2D) = "white" {}  
        _Reflections ("Base (RGB) Gloss (A)", Cube) = "skybox" { TexGen CubeReflect }  
    }  
    SubShader {  
        Tags { "Queue" = "Transparent" }  
        Pass {  
            Blend SrcAlpha OneMinusSrcAlpha  
            Material {  
                Diffuse [_Color]  
            }  
            Lighting On  
            SetTexture [_MainTex] {  
                combine texture * primary double, texture * primary  
            }  
        }  
        Pass {  
            Blend One One  
            Material {  
                Diffuse [_Color]  
            }  
            Lighting On  
            SetTexture [_Reflections] {  
                combine texture  
                Matrix [_Reflection]  
            }  
        }  
    }  
}  
```

Per-pixel Lighting 逐像素光照

逐像素光照管道的工作依赖于渲染对象在多个pass中。Unity3D一旦获得环境光和任何顶点的光照就开始渲染物体。然后，它在一个单独添加的pass中影响渲染的物体每1个光照的像素。

Per-vertex Lighting 逐顶点光照

逐顶点光照是Direct3d/OpenGl的标准光照模式，它的计算依赖于每一个顶点。打开光照后才能启动逐顶点光照。光照受Materialblock, ColorMaterial 和 SeparateSpecular命令影响。 

有几个特殊的pass可以重用公共的功能或实现一些高级效果。

UsePass 包含从另外的Shader中已被命名的pass。

GrabPass 抓取屏幕内容并转换成纹理，可以给之后的pass使用，用来做特效最好了。
