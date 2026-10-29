> 文档模板：v0.4.2

<p align="center">
  <h1 align="center"> <code>expo-gl</code> </h1>
</p>

本项目基于 [expo-gl](https://github.com/expo/expo/tree/main/packages/expo-gl) 开发，为 React Native 鸿蒙（OpenHarmony）适配版本。

该三方库的鸿蒙适配版本支持直接从仓库下载 tgz 使用，版本所属关系如下：

| 三方库名称    | 三方库版本    | 发布信息     | 支持RN版本    | Autolink     | 编译API版本     | 社区基线版本    |  源码地址   |
| ------------ | ------------ | ------------------------------ | ------------- | ------------- |------------------------ | ------------- | ------------- |
| @react-native-ohos/expo-gl | [~ 1.0.0-beta.1](https://github.com/tyBrave/expo/tree/main/packages/expo-gl) | [Github Releases](https://github.com/tyBrave/expo/releases) | 0.72.* / 0.77.* / 0.82.* / 0.84.* | Partially | API13+ | 58.0.0 | [main](https://github.com/tyBrave/expo/tree/main/packages/expo-gl) |

## 简介

expo-gl 提供 `GLView`（OpenGL ES 渲染目标视图）与 `ExpoWebGLRenderingContext`（WebGL 2.0 规范子集，经 JSI 直装到 JS 运行时），是 [gl-react](../README.md) 等上层渲染库在 RNOH 上的原生底座。

## 下载安装

进入到工程目录并输入以下命令：

**npm**

```bash
npm install @react-native-ohos/expo-gl
```

**yarn**

```bash
yarn add @react-native-ohos/expo-gl
```

## 依赖关系说明

请根据应用工程使用的 RN 框架版本，选择对应版本的三方库进行引入，版本对应关系如下：

| RN 框架版本 | 鸿蒙化依赖库 | 支持版本范围 |
| --- | --- | --- |
| 0.72 、0.77、0.82、0.84 | @react-native-ohos/expo-modules-core | >=1.0.0 <1.1.0 |
| | @react-native-ohos/expo-asset | >=1.0.0 <1.1.0 |

## Link

|                                      | 是否支持autolink | RN框架版本 |
|--------------------------------------|----------------|-----------|
| ~ 1.0.0-beta.1                       | 是             |  0.72     |
| ~ 1.0.0-beta.1                       | 否             |  0.77     |
| ~ 1.0.0-beta.1                       | 否             |  0.82     |
| ~ 1.0.0-beta.1                       | 否             |  0.84     |

使用AutoLink的工程需要根据该文档配置，Autolink框架指导文档：https://gitcode.com/openharmony-sig/ohos_react_native/blob/master/docs/zh-cn/Autolinking.md

如您使用的版本支持 Autolink，并且工程已接入 Autolink，可跳过ManualLink配置。

> [!TIP] 本库在 RNOH 0.77 / 0.82 / 0.84 上按下方 ManualLink 方式验证通过（har 手动引入 + C++/ArkTS 双侧注册）；若这些版本使用 autolinking，请按 RNOH 对应版本的 autolinking 文档配置。

<details>
  <summary>ManualLink: 此步骤为手动配置原生依赖项的指导</summary>

首先需要使用 DevEco Studio 打开项目里的 HarmonyOS 工程 `harmony`。

### 1. Overrides RN SDK

为了让工程依赖同一个版本的 RN SDK，需要在工程根目录的 `oh-package.json5` 添加 overrides 字段，指向工程需要使用的 RN SDK 版本。替换的版本既可以是一个具体的版本号，也可以是一个模糊版本，还可以是本地存在的 HAR 包或源码目录。

关于该字段的作用请阅读[官方说明](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#zh-cn_topic_0000001792256137_overrides)

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony": "~0.72.139" // ohpm 在线版本
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony.har" // 指向本地 har 包的路径
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony" // 指向源码路径
  }
}
```

### 2. 引入原生端代码

> [!TIP] har 包位于三方库安装路径的 `harmony` 文件夹下。

打开 `entry/oh-package.json5`，添加以下依赖

```json
"dependencies": {
    "@react-native-ohos/expo-gl": "file:../../node_modules/@react-native-ohos/expo-gl/harmony/expo_gl.har"
  }
```

`harmony/oh-package.json5` 中按实际路径相应添加（`file:../node_modules/...`）。

### 3. 配置 CMakeLists 和引入 ExpoGlPackage

打开 `entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
 project(rnapp)
 cmake_minimum_required(VERSION 3.4.1)
 set(CMAKE_SHARED_LINKER_FLAGS "--rtlib=compiler-rt -fuse-ld=lld -Wl,--build-id=sha1 -Wl,--warn-shared-textrel -Wl,--fatal-warnings -lunwind -Wl,--no-undefined -Qunused-arguments -Wl,-z,noexecstack")
 set(CMAKE_SKIP_BUILD_RPATH TRUE)
 set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
 set(NODE_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../node_modules")
+set(OH_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
 set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../../react-native-harmony/harmony/cpp")
+set(RNOH_CPP_DIR "${OH_MODULES}/@rnoh/react-native-openharmony/src/main/cpp")
 set(LOG_VERBOSITY_LEVEL 1)
 set(CMAKE_ASM_FLAGS "-Wno-error=unused-command-line-argument -Qunused-arguments")
 set(CMAKE_CXX_FLAGS "-fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
 set(WITH_HITRACE_SYSTRACE 1) # for other CMakeLists.txt files to use
 add_compile_definitions(WITH_HITRACE_SYSTRACE)

 add_subdirectory("${RNOH_CPP_DIR}" ./rn)

 # RNOH_BEGIN: manual_package_linking_1
 add_subdirectory("../../../../sample_package/src/main/cpp" ./sample-package)
+add_subdirectory("${OH_MODULES}/@react-native-ohos/expo-gl/src/main/cpp" ./expo_gl)
 # RNOH_END: manual_package_linking_1

file(GLOB GENERATED_CPP_FILES "./generated/*.cpp")

 add_library(rnoh_app SHARED
   ${GENERATED_CPP_FILES}
     "./PackageProvider.cpp"
     "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
 )
 target_link_libraries(rnoh_app PUBLIC rnoh)

 # RNOH_BEGIN: manual_package_linking_2
 target_link_libraries(rnoh_app PUBLIC rnoh_sample_package)
+target_link_libraries(rnoh_app PUBLIC expo_gl)
 # RNOH_END: manual_package_linking_2
```

打开 `entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
 #include "RNOH/PackageProvider.h"
 #include "SamplePackage.h"
+#include "ExpoGlPackage.h"

 using namespace rnoh;

 std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
     return {
       std::make_shared<SamplePackage>(ctx),
+      std::make_shared<ExpoGlPackage>(ctx)
     };
 }
```

### 4. 在 ArkTs 侧引入 ExpoGLView

找到 **function buildCustomRNComponent()**，一般位于 `entry/src/main/ets/pages/Index.ets` 或 `entry/src/main/ets/rn/LoadBundle.ets`，添加：

```diff
  ...
+ import { ExpoGLView } from "@react-native-ohos/expo-gl"

@Builder
export function buildCustomRNComponent(ctx: ComponentBuilderContext) {
  ...
+ if (ctx.componentName === ExpoGLView.NAME) {
+   ExpoGLView({
+     ctx: ctx.rnComponentContext,
+     tag: ctx.tag
+   })
+ }
  ...
}
```

在`entry/src/main/ets/pages/Index.ets` 或 `entry/src/main/ets/rn/LoadBundle.ets` 找到常量 `arkTsComponentNames` 在其数组里添加组件名

```diff
const arkTsComponentNames: Array<string> = [
+ ExpoGLView.NAME
  ];
```

### 5. 在 ArkTs 侧引入 ExpoGlPackage

打开 `entry/src/main/ets/RNPackagesFactory.ets`，添加：

```diff
  ...
+ import ExpoGlPackage from '@react-native-ohos/expo-gl';

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
+    new ExpoGlPackage(ctx)
  ];
}
```
</details>

### 运行

点击右上角的 `sync` 按钮

或者在命令行终端执行：

```bash
cd entry
ohpm install
```

然后编译、运行即可。

## 约束与限制

### 兼容性

本文档内容基于以下版本验证通过：

1. RNOH: 0.72.139; RN: 0.72.1; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;
2. RNOH: 0.77.48; RN: 0.77.1; react: 18.3.1; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;
3. RNOH: 0.82.34; RN: 0.82.1; react: 19.1.1; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;
4. RNOH: 0.84.2; RN: 0.84.1; react: 19.2.3; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;

### 权限要求

- 如果纹理 source 使用网络 url，应用需要申请网络权限

  在 `entry/src/main/module.json5` 中添加

```json
"requestPermissions": [
  {
    "name": "ohos.permission.INTERNET",
  },
],
```

## 使用示例
下面的代码展示了这个库的基本使用场景：

> [!WARNING] 使用时 import 的库名不变。

```tsx
import React from "react";
import { StyleSheet } from "react-native";
import { GLView } from "expo-gl";

// 1. 视图渲染：<GLView> 挂载后回调 onContextCreate，拿到 WebGL 上下文
function GLViewScreen() {
  return (
    <GLView
      style={styles.gl}
      onContextCreate={(gl) => {
        gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
        gl.clearColor(0, 0, 1, 1);
        gl.clear(gl.COLOR_BUFFER_BIT);
        gl.endFrameEXP(); // 每帧结束后调用，驱动上屏
      }}
    />
  );
}

// 2. 无头上下文（无视图渲染，配合自定义 framebuffer + 快照）
const gl = await GLView.createContextAsync();
//   ...texImage2D / framebufferTexture2D / draw...
const snapshot = await GLView.takeSnapshotAsync(gl, {
  framebuffer: fbo,
  format: "png",
  compress: 1.0,
});
// snapshot = { uri, localUri, width, height }，文件写入应用沙箱 cache 目录
const destroyed = await GLView.destroyContextAsync(gl);

const styles = StyleSheet.create({ gl: { flex: 1 } });
```

## 使用说明

**离屏（headless）上下文**

```tsx
const gl = await GLView.createContextAsync();
// 自建 FBO 挂纹理
const fbo = gl.createFramebuffer();
gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, tex, 0);
gl.viewport(0, 0, size, size);
// headless 没有渲染循环，EXGL 命令批不会自动提交，导出前必须手动 flush
gl.flushEXP();
const shot = await GLView.takeSnapshotAsync(gl, { framebuffer: fbo, format: "png" });
await GLView.destroyContextAsync(gl);
```

> [!TIP] headless 场景不调用 `gl.flushEXP()` 会报 `E_GL_NO_FRAMEBUFFER: No framebuffer bound.`（commands 未提交，原生 objects 表里查不到 fbo）。

**快照 SnapshotOptions**

```tsx
// format：jpeg(默认) / png / webp（HarmonyOS 上降级导出 .png 并打印告警，与 iOS 一致）
// flip：垂直翻转；rect：按像素裁剪；compress：0~1，1.0 不压缩
const shot = await GLView.takeSnapshotAsync(gl, {
  format: "png",
  flip: false,
  rect: { x: 200, y: 200, width: 400, height: 400 },
  compress: 1.0,
});
// shot.uri / shot.localUri 可直接给 <Image source={{ uri }} />，也可回灌为纹理输入
```

**GL 调用日志**

```tsx
import { GLLoggingOption } from "expo-gl";

gl.__expoSetLogging(
  GLLoggingOption.METHOD_CALLS | GLLoggingOption.GET_ERRORS, // 每次调用后 gl.getError() 有性能损耗
);
gl.__expoSetLogging(GLLoggingOption.DISABLED); // 关闭
```

**远程调试限制**

GL 上下文经 JSI 直装到 `global.__EXGLContexts`，与原库一致，**远程调试（Chrome Debug）模式下不可用**，`onContextCreate` 不会触发、headless 接口抛 `ERR_GL_NOT_AVAILABLE`。

## 接口说明

> [!TIP] "Platform"列表示该属性在原三方库上支持的平台。

> [!TIP] "OpenHarmony Support"列为 yes 表示 OpenHarmony平台支持 该属性；no 则表示不支持；partially 表示部分支持。使用方法跨平台一致，效果对标 iOS 或 Android 的效果。

### 组件
| 名称       | 参数类型     | 必填  | 平台  | OpenHarmony平台支持| 描述 |
|------------|------------|----------------------|------|-----|----------------|
| GLView | [GLViewProps](#glviewprops) | yes  | all | yes | OpenGL ES 渲染目标视图，挂载即创建 GL 上下文，drawingBuffer 每帧呈现为视图内容 |

### 属性
GLViewProps

| 名称                           | 参数类型          | 默认值       | 必填            | 平台   | HarmonyOS平台支持    | 描述 |
|------------------------------|--------------------|------------|---------------|------|------------------|---------------|
| onContextCreate | (gl: ExpoWebGLRenderingContext) => void | None | Yes | All | Yes | GL 上下文创建完成回调，参数 gl 为 WebGL2RenderingContext 扩展（含 contextId / endFrameEXP / flushEXP） |
| style | StyleProp\<ViewStyle\> | None | No | All | Yes | 容器样式；内部原生视图以 flex:1 铺满 |
| msaaSamples | number | 4 | No | iOS | No | iOS 多重采样；鸿蒙忽略（与 Android 一致），运行时透传为 undefined |
| enableExperimentalWorkletSupport | boolean | false | No | All | Partially | 仅保留 prop，运行时 console.warn 提示不支持变更；worklet 能力首期未开放 |
| nativeRef_EXPERIMENTAL | (ref) => void | None | No | All | Yes | 取原生 GLView 引用（hidden API，内部调试用） |
| 其余 ViewProps | ViewProps | None | No | All | Yes | 未识别的 props 原样透传给外层 View |

SnapshotOptions（`takeSnapshotAsync` 第二个参数）

| 名称       | 参数类型     | 默认值 | 必填 | 平台 | HarmonyOS平台支持 | 描述 |
|------------|------------|--------|------|------|------------------|----------------|
| format | 'jpeg' \| 'png' \| 'webp' | 'jpeg' | No | All | Partially | jpeg/png 逐项支持；webp 降级导出 `.png` 并打印告警 |
| flip | boolean | false | No | All | Yes | 垂直翻转快照 |
| rect | {x, y, width, height} | None | No | All | Yes | 按像素裁剪，直接透传 glReadPixels |
| compress | number | 1.0 | No | All | Yes | 0~1 压缩级别，1.0 不压缩 |
| framebuffer | WebGLFramebuffer | None | No | All | Yes | 指定读取的 framebuffer；缺省为视图呈现的 framebuffer（headless 场景为当前绑定 framebuffer） |

GLSnapshot（`takeSnapshotAsync` 返回值）

| 名称       | 参数类型     | 描述 |
|------------|------------|----------------------|
| uri | string \| Blob \| null | 快照文件 uri（沙箱 cache 目录，如 `file:///.../GLView/xxx.png`） |
| localUri | string | 同 uri，为兼容 `texImage2D` 保留的同义字段 |
| width / height | number | 快照像素尺寸 |

### API
| 名称     | 类型     | 参数类型 | 返回值 | 必填    | 平台 | OpenHarmony平台支持 | 描述  |
|--------|---------|-----|-------|-------|--------|-----|-----------------|
| GLView.createContextAsync | function | / | Promise\<ExpoWebGLRenderingContext\> | No | All | Yes | 创建无头（无视图）GL 上下文；导出前需自行创建 framebuffer 并调用 gl.flushEXP() |
| GLView.destroyContextAsync | function | (gl \| ctxId) | Promise\<boolean\> | No | All | Yes | 销毁上下文；上下文不存在时返回 false（与原库一致） |
| GLView.takeSnapshotAsync | function | (gl \| ctxId, options?) | Promise\<GLSnapshot\> | No | All | Yes | 快照编码为图片文件写入沙箱 cache 目录，返回 { uri, localUri, width, height } |
| GLView#takeSnapshotAsync | function | (options?) | Promise\<GLSnapshot\> | No | All | Yes | 实例方法，使用该视图关联的上下文，等价于静态方法传 this.exglCtxId |
| GLView#destroyObjectAsync | function | ({ id: number }) | Promise\<boolean\> | No | All | Yes | 销毁已注册的 GL 对象；未知 id 返回 false |
| GLView#createCameraTextureAsync | function | (cameraRefOrHandle) | Promise\<WebGLTexture\> | No | All | No | 相机画面注册为纹理，依赖 expo-camera 鸿蒙适配（未就绪）；JS 侧抛 UnavailabilityError |
| GLView.getWorkletContext | function | (contextId) | ExpoWebGLRenderingContext \| undefined | No | iOS, Android | No | 已废弃；worklet 运行时首期未开放 |
| getWorkletContext | function | (contextId) | ExpoWebGLRenderingContext \| undefined | No | iOS, Android | No | 全局导出版本，同上 |
| gl.endFrameEXP | function | / | void | No | All | Yes | 结束当前帧并上屏（GLView 内每帧需调用；headless 场景不涉及上屏） |
| gl.flushEXP | function | / | void | No | All | Yes | 提交命令队列到 GL 线程；headless 场景导出前必须手动调用 |
| gl.__expoSetLogging | function | (option: GLLoggingOption) | void | No | All | Yes | 开关 GL 调用日志，取值见 GLLoggingOption |
| gl.contextId | property | / | number | No | All | Yes | 当前上下文 id（takeSnapshotAsync 可传） |
| gl.drawingBufferWidth / Height | property | / | number | No | All | Yes | 画布像素尺寸；上下文创建时冻结 |
| WebGL 2.0 命令子集 | function | / | / | No | All | Yes | texImage2D / createProgram / bindFramebuffer 等，经 JSI 直装不经过桥；远程调试模式下不可用 |
| gl.getExtension | function | (name) | object \| null | No | All | No | 驱动扩展未按 WebGL 扩展面桥接，返回 null 并告警 |
| gl.texImage2D | function | 六参 | void | No | All | No | 无浏览器语义；请用九参形式（data / width / height） |

## 遗留问题

- `createCameraTextureAsync` 依赖 expo-camera 鸿蒙适配，暂不支持（JS 侧抛 `UnavailabilityError`）
- `takeSnapshotAsync` 的 `format: 'webp'` 降级导出 `.png`（与 iOS 行为一致）
- `msaaSamples` 为 iOS 专属能力，鸿蒙忽略
- 远程调试（Chrome Debug）模式下 GL 上下文不可用（`ERR_GL_NOT_AVAILABLE`）

## 其他
无

## 目录结构
````
/react-native-ohos-expo-gl-1.0.0-beta.1.tgz   # 本包 tgz（npm pack 产物）
└── package
    ├── src/                  # JS 端源码
    │   ├── GLView.tsx        # GLView 组件（props/静态方法/实例方法）
    │   ├── GLView.types.ts   # 类型定义（GLViewProps/SnapshotOptions/GLSnapshot/ExpoWebGLRenderingContext）
    │   ├── GLUtils.ts / GLErrors.ts / GLWorkletContextManager.ts
    │   └── specs/v1/         # NativeExpoGL TurboModule 与 ExpoGLView NativeComponent spec
    ├── harmony/
    │   ├── expo_gl.har       # 鸿蒙原生包（应用工程 entry 侧引入）
    │   └── expo_gl/          # 鸿蒙适配源码
    │       ├── index.ets     # 入口：导出 ExpoGlPackage / ExpoGLView
    │       ├── src/main/ets/ # ExpoGLView（XComponent + EGL）、ExpoAssetModule、TurboModulesFactory
    │       └── src/main/cpp/ # EXGL C++ 核心（EXWebGLRenderer/EXGLNativeContext 等）+ codegen generated
    ├── package.json          # name/version/harmony.autolinking（cmakeLibraryTargetName: expo_gl）
    ├── README.md
    └── CHANGELOG.md
````

## 贡献代码

使用过程中发现任何问题都可以提交 [Issue](https://github.com/tyBrave/expo)，当然，也非常欢迎提交 [PR](https://github.com/tyBrave/expo/pulls) 。

## 开源协议

本项目基于 [The MIT License (MIT)](https://github.com/expo/expo/blob/main/LICENSE) ，请自由地享受和参与开源。
