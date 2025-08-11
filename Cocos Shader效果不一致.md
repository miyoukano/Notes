# Cocos Creator Shader合图UV计算问题总结

## 问题背景
在Cocos Creator中，编辑器中Shader效果正常，但运行后（如浏览器环境）却出现异常，主要原因是合图（Atlas）机制导致纹理的UV坐标发生变化。

## 渲染流程与UV坐标
- 渲染流程包括顶点着色器、片元着色器、纹理等。
- UV坐标是纹理的百分比坐标，范围为0~1，原点在左上角。

## 合图机制
- **自动合图（Auto Atlas）**：项目构建时静态合图。
- **动态合图（Dynamic Atlas）**：运行时动态将贴图合并到大图中，减少DrawCall。
- 合图后，纹理会被放到一张大图中，Shader接收到的UV坐标是合图后的坐标。

## 异常现象
- 编辑器预览正常，运行后水波、圆角等特效出现边缘异常或放大。
- 这是因为Shader中使用的UV坐标已被合图机制重新计算。

## 原因分析
- 合图后，原始纹理的UV坐标会变成大图中的一部分。
- Cocos会自动计算并传递新的UV坐标（如`v_uv0`）给Shader。
- 相关源码：`CCSpriteFrame.js` 的 `_calculateUV()` 方法。

## 解决方案
1. **关闭动态合图**
	```js
	onLoad() {
		 cc.dynamicAtlasManager.enabled = false;
	}
	```
	可能会增加DrawCall。
2. **禁止贴图参与合图**
	- 在纹理属性中关闭Packable（不参与合图），同样可能增加DrawCall。
3. **动态计算归一化UV**
	- 获取纹理在合图中的四个边界UV坐标及旋转状态，传递给Shader。
	- 示例代码：
	  ```js
	  let frame = this.sprite.spriteFrame;
	  let l = frame.uv[0]; // xMin
	  let r = frame.uv[6]; // xMax
	  let b = frame.uv[3]; // yMax
	  let t = frame.uv[5]; // yMin
	  let u_uvOffset = new cc.Vec4(l, t, r, b);
	  let u_uvRotated = frame.isRotated() ? 1.0 : 0.0;
	  this.material.setProperty("u_uvOffset", u_uvOffset);
	  this.material.setProperty("u_uvRotated", u_uvRotated);
	  ```
	- Shader中归一化UV计算：
	  ```glsl
	  vec2 uvNormalize;
	  uvNormalize.x = (v_uv0.x - u_uvOffset.x) / (u_uvOffset.z - u_uvOffset.x);
	  uvNormalize.y = (v_uv0.y - u_uvOffset.y) / (u_uvOffset.w - u_uvOffset.y);
	  if(u_uvRotated > 0.5) {
			float temp = uvNormalize.x;
			uvNormalize.x = uvNormalize.y;
			uvNormalize.y = 1.0 - temp;
	  }
	  ```
	- 这样可获得合图前的UV坐标，保证特效正常。

## 总结
- 合图机制会影响Shader中的UV坐标，导致运行时特效异常。
- 可通过关闭合图、禁止贴图参与合图或动态计算归一化UV来解决。
- 推荐在需要自定义Shader特效时，优先考虑动态UV归一化方案。

---
> 原文链接：[CSDN博客](https://blog.csdn.net/u010799737/article/details/119991857)
