> Document Template: v0.4.2

<p align="center">
  <h1 align="center"> <code>expo-gl</code> </h1>
</p>

This project is based on [expo-gl](https://github.com/expo/expo/tree/main/packages/expo-gl) and is the OpenHarmony adaptation of it for React Native.

The OpenHarmony-adapted packages of this library can be downloaded as tgz directly from the repository. The version mapping is as follows:

| Library Name    | Version    | Release Info     | RN Version Supported    | Autolink     | API Version     | Community Baseline    |  Source Code   |
| ------------ | ------------ | ------------------------------ | ------------- | ------------- |------------------------ | ------------- | ------------- |
| @react-native-ohos/expo-gl | [~ 1.0.0-beta.1](https://github.com/tyBrave/expo/tree/main/packages/expo-gl) | [Github Releases](https://github.com/tyBrave/expo/releases) | 0.72.* / 0.77.* / 0.82.* / 0.84.* | Partially | API13+ | 58.0.0 | [main](https://github.com/tyBrave/expo/tree/main/packages/expo-gl) |

## Introduction

expo-gl provides `GLView` (an OpenGL ES render-target view) and `ExpoWebGLRenderingContext` (a subset of the WebGL 2.0 spec, installed directly into the JS runtime via JSI), serving as the native foundation of upper-layer rendering libraries such as [gl-react](../README.md) on RNOH.

## Installation

Enter the project directory and run:

**npm**

```bash
npm install @react-native-ohos/expo-gl
```

**yarn**

```bash
yarn add @react-native-ohos/expo-gl
```

## Dependencies

Choose the library versions matching the RN framework version of your application project. The version mapping is as follows:

| RN Framework Version | OpenHarmony-adapted Library | Supported Version Range |
| --- | --- | --- |
| 0.72 / 0.77 / 0.82 / 0.84 | @react-native-ohos/expo-modules-core | >=1.0.0 <1.1.0 |
| | @react-native-ohos/expo-asset | >=1.0.0 <1.1.0 |

## Link

|                                      | Autolink Supported | RN Framework Version |
|--------------------------------------|----------------|-----------|
| ~ 1.0.0-beta.1                       | Yes            |  0.72     |
| ~ 1.0.0-beta.1                       | No             |  0.77     |
| ~ 1.0.0-beta.1                       | No             |  0.82     |
| ~ 1.0.0-beta.1                       | No             |  0.84     |

Projects using AutoLink should follow the Autolink guide: https://gitcode.com/openharmony-sig/ohos_react_native/blob/master/docs/zh-cn/Autolinking.md

If your version supports Autolink and the project has integrated Autolink, skip the ManualLink configuration.

> [!TIP] This library has been verified on RNOH 0.77 / 0.82 / 0.84 via the ManualLink approach below (manual har import + registration on both the C++ and ArkTS sides); if you use autolinking on these versions, follow the autolinking documentation of the corresponding RNOH version.

<details>
  <summary>ManualLink: guidance for manually configuring native dependencies</summary>

Open the HarmonyOS project `harmony` of your project with DevEco Studio.

### 1. Overrides RN SDK

To make the project depend on the same RN SDK version, add the `overrides` field in the root `oh-package.json5` of the project, pointing to the RN SDK version to use. The value can be a specific version number, a fuzzy version, or a local HAR package or source directory.

For how this field works, read the [official documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#zh-cn_topic_0000001792256137_overrides)

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony": "~0.72.139" // ohpm online version
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony.har" // path to a local har package
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony" // path to source code
  }
}
```

### 2. Import the Native Code

> [!TIP] The har package is located in the `harmony` folder of the installed library.

Open `entry/oh-package.json5` and add the following dependency

```json
"dependencies": {
    "@react-native-ohos/expo-gl": "file:../../node_modules/@react-native-ohos/expo-gl/harmony/expo_gl.har"
  }
```

Add the corresponding dependency in `harmony/oh-package.json5` as well (`file:../node_modules/...`), according to the actual path.

### 3. Configure CMakeLists and import ExpoGlPackage

Open `entry/src/main/cpp/CMakeLists.txt` and add:

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

Open `entry/src/main/cpp/PackageProvider.cpp` and add:

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

### 4. Import ExpoGLView on the ArkTS side

Find **function buildCustomRNComponent()**, usually located in `entry/src/main/ets/pages/Index.ets` or `entry/src/main/ets/rn/LoadBundle.ets`, and add:

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

Find the constant `arkTsComponentNames` in `entry/src/main/ets/pages/Index.ets` or `entry/src/main/ets/rn/LoadBundle.ets` and add the component name to its array

```diff
const arkTsComponentNames: Array<string> = [
+ ExpoGLView.NAME
  ];
```

### 5. Import ExpoGlPackage on the ArkTS side

Open `entry/src/main/ets/RNPackagesFactory.ets` and add:

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

### Run

Click the `sync` button in the top-right corner,

or run in the terminal:

```bash
cd entry
ohpm install
```

Then build and run.

## Constraints and Limitations

### Compatibility

The content of this document has been verified on the following versions:

1. RNOH: 0.72.139; RN: 0.72.1; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;
2. RNOH: 0.77.48; RN: 0.77.1; react: 18.3.1; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;
3. RNOH: 0.82.34; RN: 0.82.1; react: 19.1.1; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;
4. RNOH: 0.84.2; RN: 0.84.1; react: 19.2.3; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 5.0; ROM: 6.0.0;

### Permission Requirements

- If the texture source uses a network url, the application needs to apply for the network permission

  Add to `entry/src/main/module.json5`:

```json
"requestPermissions": [
  {
    "name": "ohos.permission.INTERNET",
  },
],
```

## Usage Example
The following code shows the basic usage scenario of this library:

> [!WARNING] The import name stays unchanged.

```tsx
import React from "react";
import { StyleSheet } from "react-native";
import { GLView } from "expo-gl";

// 1. View rendering: after <GLView> mounts, onContextCreate fires and provides the WebGL context
function GLViewScreen() {
  return (
    <GLView
      style={styles.gl}
      onContextCreate={(gl) => {
        gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
        gl.clearColor(0, 0, 1, 1);
        gl.clear(gl.COLOR_BUFFER_BIT);
        gl.endFrameEXP(); // called at the end of each frame to drive presentation
      }}
    />
  );
}

// 2. Headless context (viewless rendering, used with a custom framebuffer + snapshot)
const gl = await GLView.createContextAsync();
//   ...texImage2D / framebufferTexture2D / draw...
const snapshot = await GLView.takeSnapshotAsync(gl, {
  framebuffer: fbo,
  format: "png",
  compress: 1.0,
});
// snapshot = { uri, localUri, width, height }; the file is written into the application sandbox cache directory
const destroyed = await GLView.destroyContextAsync(gl);

const styles = StyleSheet.create({ gl: { flex: 1 } });
```

## Usage Notes

**Headless (offscreen) context**

```tsx
const gl = await GLView.createContextAsync();
// create your own FBO with a texture attached
const fbo = gl.createFramebuffer();
gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, tex, 0);
gl.viewport(0, 0, size, size);
// headless has no render loop; the EXGL command batch is not submitted automatically,
// so it must be flushed manually before export
gl.flushEXP();
const shot = await GLView.takeSnapshotAsync(gl, { framebuffer: fbo, format: "png" });
await GLView.destroyContextAsync(gl);
```

> [!TIP] In headless scenarios, skipping `gl.flushEXP()` raises `E_GL_NO_FRAMEBUFFER: No framebuffer bound.` (the commands are not submitted, so the fbo cannot be found in the native objects table).

**SnapshotOptions**

```tsx
// format: jpeg (default) / png / webp (on HarmonyOS webp falls back to .png export with a warning, consistent with iOS)
// flip: flip vertically; rect: crop in pixels; compress: 0~1, 1.0 means no compression
const shot = await GLView.takeSnapshotAsync(gl, {
  format: "png",
  flip: false,
  rect: { x: 200, y: 200, width: 400, height: 400 },
  compress: 1.0,
});
// shot.uri / shot.localUri can feed <Image source={{ uri }} /> directly, or be fed back as a texture input
```

**GL call logging**

```tsx
import { GLLoggingOption } from "expo-gl";

gl.__expoSetLogging(
  GLLoggingOption.METHOD_CALLS | GLLoggingOption.GET_ERRORS, // gl.getError() after every call has a performance cost
);
gl.__expoSetLogging(GLLoggingOption.DISABLED); // disable
```

**Remote debugging limitation**

The GL context is installed directly into `global.__EXGLContexts` via JSI, consistent with the original library, and is **unavailable in remote debugging (Chrome Debug) mode**: `onContextCreate` never fires, and the headless APIs throw `ERR_GL_NOT_AVAILABLE`.

## API Description

> [!TIP] The "Platform" column indicates the platforms where the property is supported in the original library.

> [!TIP] In the "OpenHarmony Support" column, yes means the OpenHarmony platform supports the property; no means not supported; partially means partially supported. Usage is consistent across platforms, with results aligned to iOS or Android.

### Components
| Name       | Parameter Type     | Required  | Platform  | OpenHarmony Support | Description |
|------------|------------|----------------------|------|-----|----------------|
| GLView | [GLViewProps](#glviewprops) | yes  | all | yes | OpenGL ES render-target view; a GL context is created on mount, and the drawingBuffer is presented as the view content each frame |

### Props
GLViewProps

| Name                           | Parameter Type          | Default Value       | Required            | Platform   | HarmonyOS Support    | Description |
|------------------------------|--------------------|------------|---------------|------|------------------|---------------|
| onContextCreate | (gl: ExpoWebGLRenderingContext) => void | None | Yes | All | Yes | Callback fired when the GL context is created; the gl parameter is a WebGL2RenderingContext extension (including contextId / endFrameEXP / flushEXP) |
| style | StyleProp\<ViewStyle\> | None | No | All | Yes | Container style; the inner native view fills the container with flex:1 |
| msaaSamples | number | 4 | No | iOS | No | iOS multisampling; ignored on HarmonyOS (same as Android), passed through as undefined at runtime |
| enableExperimentalWorkletSupport | boolean | false | No | All | Partially | Only the prop is retained; a runtime console.warn indicates the change is not supported; the worklet capability is not available in this phase |
| nativeRef_EXPERIMENTAL | (ref) => void | None | No | All | Yes | Gets the native GLView reference (hidden API, for internal debugging) |
| Other ViewProps | ViewProps | None | No | All | Yes | Unrecognized props are passed through to the outer View as-is |

SnapshotOptions (the second parameter of `takeSnapshotAsync`)

| Name       | Parameter Type     | Default Value | Required | Platform | HarmonyOS Support | Description |
|------------|------------|--------|------|------|------------------|----------------|
| format | 'jpeg' \| 'png' \| 'webp' | 'jpeg' | No | All | Partially | jpeg/png are supported; webp falls back to `.png` export with a warning |
| flip | boolean | false | No | All | Yes | Flip the snapshot vertically |
| rect | {x, y, width, height} | None | No | All | Yes | Crop in pixels, passed through to glReadPixels directly |
| compress | number | 1.0 | No | All | Yes | Compression level 0~1; 1.0 means no compression |
| framebuffer | WebGLFramebuffer | None | No | All | Yes | The framebuffer to read from; defaults to the framebuffer presented by the view (in headless scenarios, the currently bound framebuffer) |

GLSnapshot (the return value of `takeSnapshotAsync`)

| Name       | Parameter Type     | Description |
|------------|------------|----------------------|
| uri | string \| Blob \| null | Snapshot file uri (in the sandbox cache directory, e.g. `file:///.../GLView/xxx.png`) |
| localUri | string | Same as uri; a synonymous field kept for `texImage2D` compatibility |
| width / height | number | Snapshot size in pixels |

### API
| Name     | Type     | Parameter Type | Return Value | Required    | Platform | OpenHarmony Support | Description  |
|--------|---------|-----|-------|-------|--------|-----|-----------------|
| GLView.createContextAsync | function | / | Promise\<ExpoWebGLRenderingContext\> | No | All | Yes | Creates a headless (viewless) GL context; create a framebuffer yourself and call gl.flushEXP() before export |
| GLView.destroyContextAsync | function | (gl \| ctxId) | Promise\<boolean\> | No | All | Yes | Destroys a context; returns false if the context does not exist (consistent with the original library) |
| GLView.takeSnapshotAsync | function | (gl \| ctxId, options?) | Promise\<GLSnapshot\> | No | All | Yes | Encodes a snapshot into an image file written into the sandbox cache directory, returning { uri, localUri, width, height } |
| GLView#takeSnapshotAsync | function | (options?) | Promise\<GLSnapshot\> | No | All | Yes | Instance method; uses the context associated with this view, equivalent to the static method with this.exglCtxId |
| GLView#destroyObjectAsync | function | ({ id: number }) | Promise\<boolean\> | No | All | Yes | Destroys a registered GL object; returns false for unknown ids |
| GLView#createCameraTextureAsync | function | (cameraRefOrHandle) | Promise\<WebGLTexture\> | No | All | No | Registers the camera feed as a texture; depends on the HarmonyOS adaptation of expo-camera (not available yet); throws UnavailabilityError on the JS side |
| GLView.getWorkletContext | function | (contextId) | ExpoWebGLRenderingContext \| undefined | No | iOS, Android | No | Deprecated; the worklet runtime is not available in this phase |
| getWorkletContext | function | (contextId) | ExpoWebGLRenderingContext \| undefined | No | iOS, Android | No | The globally exported version; same as above |
| gl.endFrameEXP | function | / | void | No | All | Yes | Ends the current frame and presents it (must be called each frame inside GLView; not involved in presentation in headless scenarios) |
| gl.flushEXP | function | / | void | No | All | Yes | Submits the command queue to the GL thread; must be called manually before export in headless scenarios |
| gl.__expoSetLogging | function | (option: GLLoggingOption) | void | No | All | Yes | Enables/disables GL call logging; see GLLoggingOption for the values |
| gl.contextId | property | / | number | No | All | Yes | The current context id (can be passed to takeSnapshotAsync) |
| gl.drawingBufferWidth / Height | property | / | number | No | All | Yes | Canvas size in pixels; frozen at context creation |
| WebGL 2.0 command subset | function | / | / | No | All | Yes | texImage2D / createProgram / bindFramebuffer, etc.; installed directly via JSI without going through the bridge; unavailable in remote debugging mode |
| gl.getExtension | function | (name) | object \| null | No | All | No | Driver extensions are not bridged as WebGL extensions; returns null with a warning |
| gl.texImage2D | function | six args | void | No | All | No | No browser semantics; use the nine-argument form (data / width / height) |

## Known Issues

- `createCameraTextureAsync` depends on the HarmonyOS adaptation of expo-camera and is not supported yet (throws `UnavailabilityError` on the JS side)
- `format: 'webp'` of `takeSnapshotAsync` falls back to `.png` export (consistent with iOS)
- `msaaSamples` is an iOS-only capability and is ignored on HarmonyOS
- The GL context is unavailable in remote debugging (Chrome Debug) mode (`ERR_GL_NOT_AVAILABLE`)

## Others
None

## Directory Structure
````
/react-native-ohos-expo-gl-1.0.0-beta.1.tgz   # tgz of this package (npm pack output)
└── package
    ├── src/                  # JS sources
    │   ├── GLView.tsx        # GLView component (props/static methods/instance methods)
    │   ├── GLView.types.ts   # Type definitions (GLViewProps/SnapshotOptions/GLSnapshot/ExpoWebGLRenderingContext)
    │   ├── GLUtils.ts / GLErrors.ts / GLWorkletContextManager.ts
    │   └── specs/v1/         # NativeExpoGL TurboModule and ExpoGLView NativeComponent specs
    ├── harmony/
    │   ├── expo_gl.har       # HarmonyOS native package (imported on the entry side of the application project)
    │   └── expo_gl/          # HarmonyOS adaptation sources
    │       ├── index.ets     # Entry point: exports ExpoGlPackage / ExpoGLView
    │       ├── src/main/ets/ # ExpoGLView (XComponent + EGL), ExpoAssetModule, TurboModulesFactory
    │       └── src/main/cpp/ # EXGL C++ core (EXWebGLRenderer/EXGLNativeContext, etc.) + codegen generated
    ├── package.json          # name/version/harmony.autolinking (cmakeLibraryTargetName: expo_gl)
    ├── README.md
    └── CHANGELOG.md
````

## Contributing

If you encounter any problems during usage, feel free to submit an [Issue](https://github.com/tyBrave/expo). Pull requests are also highly welcome via [PR](https://github.com/tyBrave/expo/pulls).

## License

This project is based on [The MIT License (MIT)](https://github.com/expo/expo/blob/main/LICENSE). Enjoy and contribute to open source freely.
