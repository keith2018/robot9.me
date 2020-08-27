---
title: "裸眼3D小实验"
date: 2018-07-02T22:54:08+08:00
tags: ["3D", "技术"]
draft: false
---

最近偶然看到篇3D显示的[论文](http://roxanneluo.github.io/PeppersCone.html)，感觉挺有意思，决定动手实现一下。

常见的一种裸眼3D是通过全息投影来实现，如：

![](/media/show-3d-test/e17dc9885251b8c54c7351551c11b5cf.png)

即通过图像在半透明薄片上的投影来产生立体感，由于投影面是标准的平面，如果又正好是45°角，投影图像不需要做任何形变，实现起来比较简单，而这篇论文作者采用的是圆锥体：

![](/media/show-3d-test/00ff5a670712819310857bb662d02193.png)

圆锥相比凌锥可以适应更大的观察角度，另外还可以放到旋转平台上，那么问题来了，对于圆锥，投影图像需要做怎样的形变呢？

![](/media/show-3d-test/3748c7de072ed271784ba3b82df7d29a.png)

如果按数学公式算会相对复杂，特别是，DIY的圆锥不能保证是标准形状，很可能会扁一点，也不能保证正好垂直摆放，因而实际上我们需要假设它就是不标准的形状、不标准的姿态，那如何得到图像在这样曲面上投影的形变关系呢？结构光是一种不错的方法。
Kinect和iPhoneX的人脸ID都用到了结构光技术，其主要用于3D重建，获取物体表面形状信息。原理不难理解，向物体表面投射已知结构的光，然后拍摄图像，根据图像上结构的形变来计算物体表面(深度)信息，在这个实验里，我用的比较简单的格雷码结构光。

### 结构光

基本思路：将像素点的坐标(x，y)编码进光的结构，投影后拍摄图像，解码每个像素点(X, Y)的值，从而得到(x, y) -> (X, Y)的映射关系图。格雷码结构光是时间编码方式，即投射多张图案，每张代表一个二进制bit，假如1024分辨率，就需要log2(1024)=10张图案来编码，而且对于x、y两个轴，需要在横纵方向投影两次。格雷码相比普通二进制码有一定优势，具体可参看 [https://en.wikipedia.org/wiki/Gray_code](https://en.wikipedia.org/wiki/Gray_code)

![](/media/show-3d-test/aa905c5cc729d7faa14b9ca33390944c.png)

比如上图，Line1的码字就是00000，Line2是01110，解码后将格雷码转为二进制码，就得到该像素点的坐标值

格雷码结构光生成的伪代码(x轴)：

``` cpp
encodePattern(idx) {
    for (y = 0 : height - 1) {
        for (x = 0 : width -1) {
            int gray = binaryToGray(x);              // 坐标x转为格雷码
            bool one = (gray >> idx) & 1;            // 第idx位是否是1
            image[x, y].color = one ? white : black; // 像素点颜色
        }
    }
}
```

![](/media/show-3d-test/db7004920e45dd5eb4ada9b0afaedda0.png)

解码的伪代码(x轴)：

``` cpp
decodePattern(images) {
    for (idx = 0 : images.size) {
        for (y = 0 : height - 1) {
            for (x = 0 : width -1) {
                bool one = isPixelWhite(images[idx][x, y]); // 判断该像素点编码为0还是1
                if (one) {
                    pattern[x, y] |= 1 << idx;               // 第idx位赋1
                }
            }
        }
    }

    for (y = 0 : height - 1) {
        for (x = 0 : width -1) {
            pattern[x, y] = grayToBinary(pattern);          // 格雷码转二进制
        }
    }
}
```

由于解码后的像素映射关系图需要用到3D渲染，我们将像素坐标值int型压缩到颜色rgba，即rg共16位表示x，ba共16位表示y，这样解码结果就变成了一张纹理贴图，从而方便后续的shader处理。

``` cpp
uchar *p = image[x, y];
p[0] = patternX >> 8;       // r
p[1] = patternX & 0xFF;     // g
p[2] = patternY >> 8;       // b
p[3] = patternY & 0xFF;     // a
```

最终得到的结果图如下，由于x、y压缩到r、g、b、a，所以图片看起来是有断纹的，实际坐标值是连续变化的。

![](/media/show-3d-test/a48990e5588a2aa4d25caab10ad3e6c7.png)

结构光编解码实现具体见：[https://github.com/keith2018/GrayCode](https://github.com/keith2018/GrayCode)

### Post Process

经过结构光的标定过程，我们得到了圆锥的形变映射关系图，并做成纹理贴图，下一步就是应用到3D渲染。这里我们使用Post Process技术，将原3D场景渲染到FrameBuffer，然后用shader来实现形变，最终输出形变后的3D画面。

首先创建FrameBuffer

``` cpp
_frameBuffer = FrameBuffer::create("PostProcess", FRAMEBUFFER_WIDTH, FRAMEBUFFER_HEIGHT);
DepthStencilTarget* dst = DepthStencilTarget::create("PostProcess", DepthStencilTarget::DEPTH_STENCIL, FRAMEBUFFER_WIDTH, FRAMEBUFFER_HEIGHT);
_frameBuffer->setDepthStencilTarget(dst);
```

然后创建后处理纹理

``` cpp
Material* material = Material::create(materialPath);
Texture::Sampler* sampler = Texture::Sampler::create(srcBuffer->getRenderTarget()->getTexture());
material->getParameter("u_texture")->setValue(sampler);

Mesh* mesh = Mesh::createQuadFullscreen();
_quadModel = Model::create(mesh);
```

渲染过程：

``` cpp
Rectangle defaultViewport = getViewport();

// draw into the framebuffer
setViewport(Rectangle(FRAMEBUFFER_WIDTH, FRAMEBUFFER_HEIGHT));
FrameBuffer* previousFrameBuffer = _frameBuffer->bind();

// main scene
_scene->draw();

// post process
setViewport(defaultViewport);      
previousFrameBuffer->bind();

clear(CLEAR_COLOR, Vector4(0, 0, 0, 1), 1.0f, 0);
_quadModel->draw();
```

其中用于形变映射的shader:

``` c
#ifdef OPENGL_ES
precision highp float;
#endif

// Uniforms
uniform sampler2D u_texture;
uniform sampler2D u_mapTexture;

// Inputs
varying vec2 v_texCoord;

void main()
{
    vec4 map = texture2D(u_mapTexture, v_texCoord);
    vec2 mapUV = vec2((map.b * 256.0 + map.g) / 4.0, (map.r * 256.0 + map.a) / 4.0);
    gl_FragColor = texture2D(u_texture, mapUV);
}
```

u_texture是3D场景正常渲染的画面，u_mapTexture是结构光解码后的像素映射图，mapUV的计算则是将rgba解压为x、y坐标(u、v)，如下左右分别是形变前和形变后的图像。

![](/media/show-3d-test/6f343e69e4dc63577c4fa502e85b9aeb.png)

### 实验结果

![](/media/show-3d-test/6e5970055c1605801564eff8996214ac.png)

如上是最终实际的效果，由于使用iPad来投影，要求比较暗的环境才行，否则可能亮度不够。