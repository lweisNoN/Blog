### 基于 TensorFlow.js 在网页和小程序中实现实时人脸识别和虚拟试戴



### 前言

------

现在在移动端上有很多基于 `人脸识别` 和 `OpenGL` 的 AR 应用实例，例如 FaceU，抖音等。而在 Web 端，这类的应用还处于探索阶段，因为项目需求，预研了 `TensorFlow.js` +`WebGL` 实现类似 AR 需求的可行性。并在多个平台做了验证，本文记录主要的流程和一些坑。



### 简介

------

**[TensorFlow.js](https://js.tensorflow.org/)**：TensorFlow.js 可以在浏览器中运行机器学习模型，还可以训练模型。 因为对 WebGL的支持，可以利用 GPU 加速。

**[WebGL](https://get.webgl.org/)**:	是一种  JS  版本的 OpenGL，用于在不使用插件的情况下在任何兼容的网页浏览器中呈现交互式2D和3D图形。

**[Emscripten](https://github.com/kripken/emscripten)**: 一个 LLVM  可选的后端，LLVM-IR 到 JS 的编译器。

**[OpenCV.js](https://docs.opencv.org/3.4/d5/d10/tutorial_js_root.html)**: OpenCV 的 JS  版本，支持 WebAssembly 。

**[WebAssembly](https://developer.mozilla.org/zh-CN/docs/WebAssembly)**: 可以在现代的网络浏览器中运行 － 它是一种低级的类汇编语言。



**简单流程**：

![data3](https://github.com/lweisNoN/Blog/blob/master/images/tf1.jpg)


### 通过 TensorFlow.js 识别特征点

------

* **TensorFlow 运行在浏览器的逻辑流程**

  在我们常见的移动端编译器中，例如 iOS 的 clang，可以视为 llvm 的前端，将目标代码(比如 C  或者 Swift)编译成 bitcode 等中间代码，这样  LLVM 后端可以很简单黑盒的适配各类 cpu 的机器码。目前在 Web  端也基于 LLVM 的架构实现了类似的逻辑。

```sequence
participant C++
participant LLVM IR
participant Wasm
participant WebAssembly API

C++->LLVM IR: 前端编译器
LLVM IR->Wasm: Emscriten
Wasm->WebAssembly API: .wasm
```

如图所示，C++ 的目标代码，通过 clang 等前端编译器编译成 LLVM IR 中间码，再通过后端编译器 Emscriten

编译成现在大部分浏览器都支持的 wasm 格式。通过 WebAssembly 的 API  在浏览器运行，因为汇编特性，经过测试，大概可以达到 C++ 百分之50的效果。



* **引入OpenCV.js**

  OpenCV.js 的引入主要用于计算角度，整个库也是 `WebAssembly` 的方式读取和使用，需要注意的是需要单独编译相应的模块，才会支持角度计算的方法。并且在编译过程中可以裁剪部分不需要的模块，原始的模块接近 5M，经过裁剪可以缩小到1.6M 左右，这样在前端加载的时候才能更加高效和节省流量。在 OpenCV.js  源码中，可以像如下代码一样增加钩子函数 `onWASMReady`，确定 wasm 文件是否加载完毕。

  ```
  function receiveInstantiatedSource(output) {
      receiveInstance(output["instance"], output["module"])
      onWASMReady();
  }
  ```



* **角度计算**

  以下是 OpenCV.js 实现的角度计算代码，和 C++ 的也类似。

```
function getYawPitchRoll(rVec, tVec)
{
    let r_mat             = new cv.Mat(3, 3, cv.CV_64FC1);
    let pose_mat          = new cv.Mat(3, 4, cv.CV_64FC1);
    let out_intrinsics    = new cv.Mat(3, 3, cv.CV_64FC1);
    let out_rotation      = new cv.Mat(3, 3, cv.CV_64FC1);
    let out_translation   = new cv.Mat(3, 1, cv.CV_64FC1);
    var picthYawRollEuler = new cv.Mat(3, 1, cv.CV_64F);
    var noArray           = new cv.Mat();

    cv.Rodrigues(rVec, r_mat);

    cv.Rodrigues(rVec, r_mat);
    let matvec = new cv.MatVector();
    matvec.push_back(r_mat);
    matvec.push_back(tVec);
    cv.hconcat(matvec, pose_mat);
    cv.decomposeProjectionMatrix(pose_mat, out_intrinsics, out_rotation, out_translation, noArray, noArray, noArray, picthYawRollEuler);
    return picthYawRollEuler;
}

function generatePNP(cvpointArray, w, h)
{
    let cam_mat = [
        w, 0, w * 0.5,
        0, w, h * 0.5,
        0, 0, 1
    ];

    let imgM          = cv.matFromArray(68, 2, cv.CV_64FC1, cvpointArray);
    let r_vec         = cv.matFromArray(3, 1, cv.CV_64FC1, [ 0.0, 0.0, -3.1415926 ]);
    let t_vec         = cv.matFromArray(3, 1, cv.CV_64FC1, [ -0.0, 0.0, -2053.03596872 ]);
    let camera_matrix = cv.matFromArray(3, 3, cv.CV_64FC1, cam_mat);
    let dist_coeefs   = cv.Mat.zeros(4, 1, cv.CV_64FC1);
    cv.solvePnP(mean_model_mat, imgM, camera_matrix, dist_coeefs, r_vec, t_vec, true);
    let angle          = getYawPitchRoll(r_vec, t_vec);
    let angleArray     = angle['data64F'];
    yawPitchRoll.yaw   = -Math.asin(Math.sin(angleArray[1] * Math.PI / 180.0)) * 180.0 / Math.PI;
    yawPitchRoll.pitch = -Math.asin(Math.sin(angleArray[0] * Math.PI / 180.0)) * 180.0 / Math.PI;
    yawPitchRoll.roll  = Math.asin(Math.sin(angleArray[2] * Math.PI / 180.0)) * 180.0 / Math.PI;

    return yawPitchRoll;
}
```



### 通过 WebGL 渲染 AR  模型

------

WebGL 和 OpenGL ES 接口类似。通过 Video 标签获取到视频流以后，可以作为背景的 Texture 源数据动态刷新 。

如下，通过 getUserMedia 可以获取摄像头的视频流，并且绑定到 Video 标签。

```
const stream = await navigator.mediaDevices.getUserMedia({
                                               audio : false,
                                               video : { width : videoW, height : videoH }
                                           })
                   .then(function success(stream) {
                       videoEl.srcObject = stream;
                   })
```

如下，是利用 Video 标签刷新绑定背景的 Texture。

```
function updatebackgroundTexture(video)
{
    const level          = 0;
    const internalFormat = gl.RGBA;
    const srcFormat      = gl.RGBA;
    const srcType        = gl.UNSIGNED_BYTE;
    gl.bindTexture(gl.TEXTURE_2D, backgroundGLInfo.texture);
    gl.texImage2D(gl.TEXTURE_2D, level, internalFormat,
                  srcFormat, srcType, video);
}
```

需要注意的是，在 iOS 系统下，Safari 浏览器需要设置 `videoEl.setAttribute('playsinline', '');` 。

因为 iOS 系统对 WebRTC 的支持仅限于 iOS11以上的 Safari，因此在 iOS 系统下其他的浏览器和微信浏览器、小程序中都无法运行，而 PC 浏览器和大部分的安卓，都可以测试成功。

`无跟踪`的效率记录如下。

| 平台                 | 效率/fps |
| -------------------- | -------- |
| Mac Safari           | 16+      |
| Mac Chrome           | 20+      |
| iPhone 7 plus Safari | 14+      |
| vivoX20 Chrome       | 6+       |
| vivoX20  小程序      | 6+       |



Mac Chrome 的运行效果:

![data1](https://github.com/lweisNoN/Blog/blob/master/images/data1.gif)



小程序中的运行效果：

![data2](https://github.com/lweisNoN/Blog/blob/master/images/data2.gif)



