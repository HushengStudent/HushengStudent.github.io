---
layout: post
title:  "[Unity Shader]Shader Learing(Surface and Vertex&Fragment  Shader篇)"
date:   2017-03-04 01:34:10
categories: UnityShader
comments: true
---

### Shader原理：
着色器（英语：shader）应用于计算机图形学领域，指一组供计算机图形资源在执行渲染任务时使用的指令，用于计算图像的颜色或明暗。但近来，它也能用于处理一些特殊效果，或者视频后处理。通俗地说，着色器告诉电脑如何用特有的一种方法去绘制物体。

程序员将着色器应用于图形处理器（GPU）的可编程流水线，来实现三维应用程序。这样的图形处理器有别于传统的固定流水线处理器，为GPU编程带来更高的灵活性和适应性。以前固有的流水线只能进行一些几何变换和像素灰度计算。现在可编程流水线还能处理所有像素、顶点、纹理的位置、色调、饱和度、明度、对比度并实时地绘制图像。着色器还能产生如模糊、高光、有体积光源、失焦、卡通渲染、色调分离、畸变、凹凸贴图、边缘检测、运动检测等效果。

Shader language目前有3种主流语言：基于OpenGL的GLSL（OpenGL Shading Language），基于 Direct3D 的 HLSL （High Level Shading Language），还有NVIDIA公司的Cg （C for Graphic）语言。不过高级语言的一个重要特性是“独立于硬件”，在这一方面 shader language 暂时还做不到， shader language 完全依赖于 GPU 构架，这一特征在现阶段是非常明显的！任意一种 shader language 都必须基于图形硬件，所以 GPU 编程技术的发展本质上还是图形硬件的发展。在 shader language存在之前，展示基于图形硬件的编程能力只能靠低级的汇编语言。

Shader工作原理：GPU 上的两个组件： Programmable Vertex Processor （可编程顶点处理器，又称为顶点着色器）和 Programmable Fragment Processor （可编程片断处理器，又称为片断着色器）。顶点和片段处理器被分离成可编程单元，可编程顶点处理器是一个硬件单元，可以运行顶点程序，而可编程片段处理器则是一个可以运行片段程序的单元。顶点和片段处理器都拥有非常强大的并行计算能力，并且非常擅长于矩阵（不高于 4 阶）计算，片段处理器还可以高速查询纹理信息（目前顶点处理器还不行，这是顶点处理器的一个发展方向）。

### CG语法：
一个Cg profile定义了一个“被特定图形硬件或API所支持的Cg语言子集”，任意一种shader language都是基于可编程图形硬件的（寄存器、指令集等），这也就意味着，不同的图形硬件对应着不同的功能子集。这些可选的语言功能包括某些控制结构和标准库函数。profile还定义了数据类型的精度，并且指定数据类型是否全局部或仅部分支持。profile按照功能可以划分为顶点profile和片段profile，而顶点profile和片段profile又基于OpenGL和DirectX的不同版本或扩展，划分为各种版本。

Shader本身是一个单纯的单元，就是对输入（顶点或像素或物体）进行能做的算术运算，然后把结果送出的一个固件。

1、内置元类型 

float：32 bit浮点类型/half：16 bit浮点/int：32 bit 整形/fixd：12 bit定点/bool：布尔值

Sampler*:纹理对象的句柄，分为 6 类：sampler、sampler1D、sampler2D、sampler3D、samplerCUBE和samplerRECT。string：不是每个都支持。

上述类型很多都被cg提供内置的向量数据类型(build-in vector data types)，如float1/float4、bool2/bool3。Cg 还提供矩阵数据类型，不过最大的维数不能超过 4*4 阶：float1x1、float2x3

Cg 中向量、矩阵与数组是完全不同，向量和矩阵是内置的数据类型（矩阵基于向量），而数组则是一种数据结构，不是内置数据类型。

数组类型/结构类型

关系操作符：<  小于，<=  小于或等于 ，!=  不等于 ，==  等于 ，>=  大于或等于，>  大于。

逻辑操作符：&& ，||  ，!  

数学操作符： * 乘法；/ 除法； - 取反； + 加法；—减法； % 求余； ++ ；——； *= ； /= ； += ； -= ；

移位操作符

Swizzle 操作符：可以使用 Cg 语言中的 swizzle 操作符（ . ）将一个向量的成员取出组成一个新的向量。float4(a, b, c, d).xyz 等价于 float3(a, b, c)。

注意： swizzle 操作符只能对结构体和向量使用，不能对数组使用。

条件操作符：expr1 ? expr2 : expr3

控制流语句

输入语义关键字：

语义：某种意义上，语义是一种黏合剂，它把一个Cg程序和图形流水线的其他部分绑定在一起。当Cg程序返回它输出的结构时，语义指明了各成员的硬件资源。

Vertex Shader的输入：

POSITION/NORMAL/BINORMAL/BLENDINDICES/BLENDWEIGHT/TANGENT/PSIZE/TEXCOORD0 ~ TEXCOORD7

如：in float4 modelPos: POSITION

Vertex Shader的输出，也就是Pixel Shader的输入：

POSITION/PSIZE/FOG/COLOR0 ~ COLOR1/TEXCOORD1 ~ TEXCOORD7

Pixel Shader的输出：

COLOR

应用程序（宿主程序）将图元信息（顶点位置、法向量、纹理坐标等）传递给顶点着色程序；顶点着色程序基于图元信息进行坐标空间转换，运算得到的数据传递到片段着色程序中；片段着色程序还可以接受从应用程序中传递的纹理信息，将这些信息综合起来计算每个片段的颜色值，最后将这些颜色值输送到帧缓冲区（或颜色缓冲区）中。

图元数据存放的硬件资源（寄存器或者纹理），称之为语义词（Semantics），通常也根据其用法称之为绑定语义词（binding semantics）。

三个关键字， in /out /inout ，用于表示函数的输入参数的传递方式，称为输入 \ 输出关键字。

两个修辞符： uniform ，用于指定变量的数据初始化方式； const关键字的含义与 C\C++ 中相同，表示被修辞变量为常量变量。

Cg 语言将输入数据流分为两类：

1、Varying inputs，即数据流输入图元信息的各种组成要素。从应用程序输入到 GPU 的数据除了顶点位置数据，还有顶点的法向量数据，纹理坐标数据等。 Cg 语言提供了一组语义词，用以表明参数是由顶点的哪些数据初始化的。

2、Uniform inputs ，表示一些与三维渲染有关的离散信息数据，这些数据通常由应用程序传入，并通常不会随着图元信息的变化而变化，如材质对光的反射信息、运动矩阵等。

函数/数组形参/ 函数重载/入口函数

Cg 标准函数库主要分为五个部分：

1、数学函数（ Mathematical Functions ）;

2、几何函数 (Geometric Functions) ；

3、纹理映射函数 (Texture Map Functions) ；

4、偏导数函数 (Derivative Functions) ；

5、调试函数 (Debugging Function) ；

数学函数：

abs(x)  返回输入参数的绝对值。

acos(x)  反余切函数，输入参数范围为 [-1,1] ，返回 [ ] 0, π 区间的角度值。

dot(A,B)  返回 A 和 B 的点积 (dot product) 。参数 A 和 B可以是标量，也可以是向量（输入参数方面，点积和叉积函数有很大不同）。

log(x)计算ln x 的值， x 必须大于 0。

几何函数：

distance( pt1， pt2)  两点之间的欧几里德距离（ Euclidean distance ）。

纹理映射函数：

tex1D(sampler1D tex, float s)一维纹理查询。

Tex1D(sampler1D tex, float2 sz)一维纹理查询，并进行深度值比较。

Tex2D(sampler2D tex, float2 s)二维纹理查询。

s 象征一元、二元、三元纹理坐标； z 代表使用“深度比较（ depth comparison ）”的值； q 表示一个透视值（perspective value, 其实就是透视投影后所得到的齐次坐标的最后一位），这个值被用来除以纹理坐标（S），得到新的纹理坐标（已归一化到 0 和 1 之间）然后用于纹理查询。

纹理函数非常多，总的来说，按照纹理维数进行分类，即： 1D 纹理函数，2D 纹理函数， 3D 纹理函数，以及立方体纹理。需要注意， TexREC 函数查询的纹理实际上也是二维纹理。

偏导函数：

ddx(a)  参数 a 对应一个像素位置，返回该像素值在 X 轴上的偏导数

ddy(a)  参数 a 对应一个像素位置，返回该像素值在 X 轴上的偏导数

传入片段程序的顶点属性一般有：屏幕坐标空间的顶点齐次坐标、纹理坐标、法向量、光照颜色等。假设传递给 ddx\ddy 函数的参数 myVar 是纹理坐标，则， ddx(myVar)的值为，纹理上像素点 p(i+1，j)的纹理颜色值减去 myVar 对应的纹理颜色值。

偏导数的物理含义是：在某一个方向上的变化快慢。所以 ddx 求的是 X 方向上，相邻两个像素的某属性值的变化量； ddy 球的是 Y方向上，相邻两个像素的某属性值的变化量。

调试函数：

void debug(float4 x)  如果在编译时设置了 DEBUG ，片段着色程序中调用该函数可以将值 x 作为COLOR 语义的最终输出；否则该函数什么也不做。

Unity Shader：

Unity中编写Shader的语言叫做ShaderLab，而ShaderLab说白了就是裹着一层皮的CG着色器语言而已。Cg，即C for graphics，即用于图形的C语言，是微软Microsoft和英伟达NVIDIA相互协作在标准硬件光照语言的语法和语义上达成的一种一致性协议。

Cg 是一个可以被 OpenGL 和 Direct3D 广泛支持的图形处理器编程语言。 Cg 语言和 OpenGL /DirectX 并不是同一层次的语言，而是OpenGL和DirectX的上层。

Microsoft和NVIDIA联手推出CG语言，想在经济和技术上实现双赢，从而通过这种方式联手打击他们共同的对手GLSL。

材质（Materials）用来把网格（Mesh）或粒子渲染器（Particle Renderers）贴到游戏对象上。他们在定义对象怎么被显示发挥重要组成部分。材质包括用于呈现网状或颗粒着色器的参考，所以这些组件不能在没有材质的情况下显示。

Material这个需要结合shader来讲，计算机图形学里本身就没有Material这个东西，引擎加入这个其实是在shader和主程序之间搭建了一座桥梁，可以说Material是一个着色器管理器，所以很多接口都是对shader的控制。

Mesh Filter ： 存储一个Mesh（网格，模型的网格)。

Mesh Renderer:用来渲染一个模型的外观，就是样子，通过Material（材质）控制模型渲染的样子。

Material

+贴图(可以没有，可以是一个单纯的颜色)

+Shader


1、表面着色器 Surface Shader：本质上和顶点/片元着色器是一样的，最终还是转换成顶点/片元着色器。

2、顶点/片元着色器 Vertex/Fragment Shader：更加复杂和灵活

3、固定函数着色器 Fixed Function Shader

可以这样区分：

没有嵌套CG语言，也就是代码段中没有CGPROGARAM和ENDCG关键字的，就是固定功能着色器。

嵌套了CG语言，代码段中有surf函数的，就是表面着色器。

嵌套了CG语言，代码段中有#pragma vertex name和  #pragma fragment frag声明的，就是顶点着色器&片段着色器。

Unity的固定管线Shader可以通过Inspectot中的Show generated code来查看对应的Cg Shader，这是很好的研究方法。

Unity Shader内置矩阵：

UNITY_MATRIX_MVP        当前模型*视图*投影 矩阵

UNITY_MATRIX_MV           当前模型*视图 矩阵

UNITY_MATRIX_V              当前视图矩阵。

UNITY_MATRIX_P              目前的投影矩阵

UNITY_MATRIX_VP            当前视图*投影 矩阵

UNITY_MATRIX_T_MV       模型*视图*转置矩阵

UNITY_MATRIX_IT_MV      模型*视图*逆*转置矩阵

UNITY_MATRIX_TEXTURE0 ~ UNITY_MATRIX_TEXTURE3   纹理变换矩阵

_Object2World Current model matrix.     当前对象到世界 矩阵

_World2Object Inverse of current world matrix. 世界到对象 矩阵

Unity Shader内置摄像机和屏幕参数：

_WorldSpaceCameraPos      float3     相机的世界坐标

_ProjectionParams           float4  x 值是1.0或-1.0（当为反转的投影矩阵时）y 近截面，z远截面，w 1/远截面 

_ScreenParams         float4   屏幕尺寸x宽，y高，z 1+1.0/width,w 1+1.0/height

_ZBufferParams        float4   用于线性化zbuffer

unity_OrthoParams      float4   正交摄像机的相关参数,w值1时是正交投影，0时是透视投影

unity_CameraProjection   float4x4  摄像机的投影矩阵

unity_CameraInvProjection float4x4  摄像机的投影矩阵的逆矩阵

unity_CameraWorldClipPlanes[6]  float4    视锥体的左右下上远近

使用Unity创建一个新的Surface Shader实例。

```java
/*  
VS2015对Shader有部分支持（只有关键字高亮）  
VS插件：ShaderLabVS  
*/  
Shader "Custom/001_NewShader" { //Shader的路径名称  
    //资源属性代码块  
    Properties {     
        _MainTex ("Base (RGB)", 2D) = "white" {}  
  
        //_MainTex ("Base (RGB)", 2D) = "white" {}  
        // 变量名  Inspector显示名字 变量类型  默认值  
        /*   
         类型:   
         Color  一种颜色   
         2D     2的阶数大小的贴图   
         Rect   非2阶数大小的贴图   
         Cube   立方体纹理   
         Range(min, max) 介于最小值最大值的浮点数   
         Float 任意一个浮点数   
         Vector 一个四维数   
         */    
//Properties example===================================  
        _Color("Color",Color)=(1,1,1,1)  
        _Vector("Vector",Vector)=(1,2,3,4)  
        _Int("Int",Int)= 34234  
        _Float("Float",Float) = 4.5  
        _Range("Range",Range(1,11))=6  
        _2D("Texture",2D) = "red"{}  
        _Cube("Cube",Cube) = "white"{}  
        _3D("Texure",3D) = "black"{}  
//====================================================  
    }  
    /*  
    SubShader可以写很多个 显卡运行效果的时候，从第一个SubShader开始  
    如果第一个SubShader里面的效果都可以实现，那么就使用第一个SubShader  
    如果显卡这个SubShader里面某些效果它实现不了，它会自动去运行下一个SubShader  
    */  
    SubShader {  
        //标签：控制以怎样的方式？何时渲染模型?  
        /*   
        重要的Tags:   
        "RenderType"="Transparent"     透明/半透明像素渲染   
        "ForceNoShadowCasting"="True"  从不产生阴影   
        "Queue"="xxx"                    
        渲染顺序队列，主要参数：   
        "Backgroung"  最早调用，用来渲染天空盒或者背景 1000   
        "Geometry"    渲染非透明物体，默认值 2000   
        "AlphaText"    "Transparent" 从后往前的顺序渲染透明物体 3000   
        "Overlay" 渲染叠加效果，渲染的最后阶段，镜头光晕之类 4000   
        */    
        Tags { "RenderType"="Opaque" }  
        LOD 200 //Level of Detail  
          
        CGPROGRAM  
        /*  
        告诉着色器使用哪个光照模型，此处surf函数使用Lambert光照模型  
        surface指明这是一个表面着色器  
        */  
        #pragma surface surf Lambert   
  
        sampler2D _MainTex;  
  
//在CG语言中调用=====================================  
            float4 _Color;   
            float4 _Vector;  
            float _Int;  
            float _Float;  
            float _Range;   
            sampler2D _2D;  
            samplerCUBE _Cube;  
            sampler3D _3D;  
            //float  32位来存储  
            //half  16  -6万 ~ +6万  
            //fixed 11 -2 到 +2  
//====================================================  
        struct Input {  
            float2 uv_MainTex;  
        };  
  
        //处理函数，输入输出必须按规定写  
        void surf (Input IN, inout SurfaceOutput o) {  
            half4 c = tex2D (_MainTex, IN.uv_MainTex);  
            o.Albedo = c.rgb;  
            o.Alpha = c.a;  
        }  
        ENDCG  
    }   
    //没有匹配的SubShader  
    FallBack "Diffuse"  
}  
  
/*  
     Surface Shader 结构  
     Shader "Custom/myshader01" {  
         Properties {  
         }  
         SubShader {   
         }  
         FallBack "Diffuse"  
     }  
*/  
```

通过以上实例，我们能对Unity Surface Shader的组成有了一定的了解。

下面再通过一个实例，对Unity Vertex&FragmentShader进行了解。

```java
Shader "Custom/002_Vertex&FragmentShader" {  
    SubShader {  
        Pass{  
            CGPROGRAM  
            //顶点函数声明顶点函数的函数名  
            //基本作用是 完成顶点坐标从模型空间到剪裁空间的转换（从游戏环境转换到视野相机屏幕上）  
            #pragma vertex vert  
            //片元函数声明了片元函数的函数名  
            //基本作用返回模型对应的屏幕上的每一个像素的颜色值  
            #pragma fragment frag  
            float4 vert(float4 v : POSITION) :SV_POSITION {  
            //POSITION:顶点坐标  
            //SV_POSITION:返回值是剪裁空间下的顶点坐标  
            //UNITY_MATRIX_MVP:Unity内置矩阵：模型观察投影矩阵  
            return mul(UNITY_MATRIX_MVP,v);  
            }  
            //SV_Target：将输出的颜色输出到默认的帧缓冲  
            fixed4 frag() :SV_Target   
            {  
                return fixed4(0.5,0.5,1,1);  
            }  
            ENDCG  
        }  
    }   
    FallBack "Diffuse"  
}  
```

语义中的数据，由MeshRender提供。

下面再通过一个示例，来实现在顶点着色器和片元着色器之间传递信息。

```java
Shader "Custom/003_UseStruct Shader" {  
    SubShader{  
        Pass{  
            CGPROGRAM  
    #pragma vertex vert  
    #pragma fragment frag  
            //application to vertex  
            struct a2v {  
                float4 vertex:POSITION;  
                //告诉unity把模型空间下的顶点坐标填充给vertex  
                float3 normal:NORMAL;  
                //告诉unity把模型空间下的法线方向填充给normal  
                float4 texcoord : TEXCOORD0;  
                //告诉unity把第一套纹理坐标填充给texcoord  
            };  
            struct v2f {  
                float4 position:SV_POSITION;  
                float3 temp:COLOR0;  
                //这个语义可以由用户自己定义，一般都存储颜色  
            };  
            v2f vert(a2v v) {   
                v2f f;  
                f.position = mul(UNITY_MATRIX_MVP,v.vertex);  
                f.temp = v.normal;  
                return f;  
            }   
            fixed4 frag(v2f f) :SV_Target{  
                return fixed4(f.temp,1);  
            }  
            ENDCG  
        }  
    }  
    FallBack "Diffuse"  
}  
```

