---
layout: post
title:  "[Unity]Post Processing"
date:   2021-06-06 00:00:10
categories: Unity
comments: true
---

Post Processing

```csharp
private static void SetRenderTargetWithLoadStoreAction(this CommandBuffer cmd
            , RenderTargetIdentifier rt, LoadAction loadAction, StoreAction storeAction)
        {
#if UNITY_2018_2_OR_NEWER
            cmd.SetRenderTarget(rt, loadAction, storeAction);
#else
            cmd.SetRenderTarget(rt);
#endif
        }
```

```csharp
public static void BlitFullscreenTriangle(this CommandBuffer cmd, RenderTargetIdentifier source, RenderTargetIdentifier destination
            , Material mat, int pass, MaterialPropertyBlock properties, LoadAction loadAction)
        {
            cmd.SetGlobalTexture(ShaderIDs.MainTex, source);
            bool clear = false;
#if UNITY_2018_2_OR_NEWER
            if (loadAction == LoadAction.Clear)
            {
                loadAction = LoadAction.DontCare;
                clear = true;
            }
#endif
            cmd.SetRenderTargetWithLoadStoreAction(destination, loadAction, StoreAction.Store);

            if (clear)
            {
                cmd.ClearRenderTarget(true, true, Color.clear);
            }

            cmd.DrawMesh(fullscreenTriangle, Matrix4x4.identity, mat, 0, pass, properties);
        }
```

```csharp
public static void BuiltinBlit(this CommandBuffer cmd, RenderTargetIdentifier source, RenderTargetIdentifier destination)
        {
#if UNITY_2018_2_OR_NEWER
            cmd.SetRenderTarget(destination, LoadAction.DontCare, StoreAction.Store);
            destination = BuiltinRenderTextureType.CurrentActive;
#endif
            cmd.Blit(source, destination);
        }
```

```csharp
        v2f vert (appdata v)
        {
            v2f o;
            o.vertex = float4(v.vertex.xy, 0.0, 1.0);
            o.uv = TransformTriangleVertexToUV(v.vertex.xy);
            o.uv = UVStartAtTop(o.uv);
            return o;
        }
```