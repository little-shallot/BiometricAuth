# 生物识别登录插件 BiometricAuth

## 一、插件简介

BiometricAuth 是一款适用于 uni-app App 端的原生生物识别身份验证插件，支持 Android 与 iOS 平台调用系统级生物识别能力，实现指纹识别、Face ID、Touch ID 等身份验证功能。

适用场景包括：登录验证、敏感操作二次确认、支付前校验、账号安全验证、隐私页面访问验证等。

## 二、插件信息

| 项目 | 内容 |
|---|---|
| 插件 ID | `BiometricAuth` |
| 插件名称 | 生物识别登录 |
| 当前版本 | `1.0.0` |
| 插件类型 | uni-app 原生插件 |
| Android 集成方式 | `aar` |
| iOS 集成方式 | `framework` |
| Android 最低版本 | Android 6.0 / API 23 |
| iOS 最低版本 | iOS 11.0 |
| JS 调用模块名 | `BiometricAuth` |

## 三、支持平台

### Android

- 支持指纹 / 生物识别能力检测
- 支持系统生物识别认证
- 支持取消按钮文案配置
- 基于 AndroidX Biometric 能力实现

Android 依赖：

```json
"androidx.biometric:biometric:1.1.0",
"androidx.fragment:fragment:1.6.2"
```

Android 权限：

```json
"android.permission.USE_BIOMETRIC",
"android.permission.USE_FINGERPRINT"
```

### iOS

- 支持 Face ID
- 支持 Touch ID
- 支持生物识别能力检测
- 支持生物识别锁定状态识别
- 支持锁定后引导用户使用锁屏密码验证
- 支持识别失败次数过多后主动检测锁定状态
- 支持通过参数控制是否由原生层弹出密码确认框

依赖系统框架：

```json
"LocalAuthentication.framework"
```

Face ID 权限说明：

```json
"NSFaceIDUsageDescription": "需要使用面容ID进行身份验证"
```

## 四、功能特性

### 1. 检测设备是否支持生物识别

通过 `checkSupport` 方法检测当前设备是否支持生物识别，并返回可用状态、识别类型及不可用原因。

可返回信息包括：

- 是否支持生物识别
- 是否已录入生物特征
- 当前识别类型
- 是否需要使用锁屏密码兜底
- 不可用原因

### 2. 发起生物识别认证

通过 `authenticate` 方法拉起系统生物识别认证。

支持状态包括：

- 认证成功
- 用户取消
- 认证失败
- 生物识别锁定
- 锁屏密码验证成功或失败

### 3. iOS 锁定后密码兜底

当 iOS Face ID / Touch ID 连续失败导致系统锁定时，插件支持自动识别锁定状态，并根据 `showDeviceCredentialConfirm` 参数决定是否弹出原生确认框。

开启后，插件会弹出：

```text
生物识别已锁定
是否使用锁屏密码进行验证？
[取消] [使用密码]
```

用户点击“使用密码”后，插件继续拉起 iOS 系统锁屏密码验证；用户点击“取消”则直接返回取消结果，不会继续弹出锁屏密码验证。

### 4. iOS 锁定状态主动复检

在 Face ID / Touch ID 验证失败回调后，插件会主动再次检测当前生物识别是否已被系统锁定。

这可以覆盖以下场景：

1. 用户连续多次识别失败。
2. 系统提示验证次数过多。
3. 用户关闭系统生物识别弹窗。
4. 插件主动判断是否已锁定。
5. 若已锁定，则根据 `showDeviceCredentialConfirm` 决定是否弹出“使用密码”确认框。

## 五、安装与配置

### 1. 插件目录结构

```text
BiometricAuth
├─ android
│  └─ BiometricAuth.aar
├─ ios
│  └─ BiometricPlugin.framework
│     ├─ BiometricPlugin
│     └─ Info.plist
└─ package.json
```

### 2. manifest.json 配置

在 uni-app 项目 `manifest.json` 中配置原生插件：

```json
{
  "app-plus": {
    "nativePlugins": {
      "BiometricAuth": {}
    }
  }
}
```

配置完成后，需要重新制作自定义运行基座。

## 六、使用方法

### 1. 引入插件

```js
// #ifdef APP-PLUS
const biometric = uni.requireNativePlugin('BiometricAuth');
// #endif
```

Android 和 iOS 均使用同一个模块名：`BiometricAuth`。

### 2. 检测生物识别支持情况

```js
biometric.checkSupport((res) => {
  console.log('生物识别支持情况：', res);

  if (res.canAuth) {
    console.log('当前设备可进行生物识别认证');
  } else {
    uni.showToast({
      title: res.reason || '当前设备不支持生物识别',
      icon: 'none',
    });
  }
});
```

### 3. 发起 iOS 认证

```js
biometric.authenticate(
  {
    title: '验证身份',
    allowDeviceCredential: false,
    showDeviceCredentialConfirm: true,
  },
  (res) => {
    if (res.code === 0) {
      uni.showToast({
        title: '验证成功',
        icon: 'none',
      });
    } else {
      uni.showToast({
        title: res.msg || '当前验证失败',
        icon: 'none',
      });
    }
  },
);
```

### 4. 发起 Android 认证

```js
biometric.authenticate(
  {
    title: '请验证身份',
    negativeText: '取消',
  },
  (res) => {
    if (res.code === 0) {
      uni.showToast({
        title: '验证成功',
        icon: 'none',
      });
    } else {
      uni.showToast({
        title: res.msg || res.reason || '当前验证失败',
        icon: 'none',
      });
    }
  },
);
```

## 七、完整调用示例

```js
// #ifdef APP-PLUS
const systemInfo = uni.getSystemInfoSync();
const isIOS = systemInfo.platform === 'ios';
const biometric = uni.requireNativePlugin('BiometricAuth');

const startBiometricAuth = () => {
  biometric.authenticate(
    isIOS
      ? {
          title: '验证身份',
          allowDeviceCredential: false,
          showDeviceCredentialConfirm: true,
        }
      : {
          title: '请验证身份',
          negativeText: '取消',
        },
    (res) => {
      if (res.code === 0) {
        uni.showToast({
          title: '验证成功',
          icon: 'none',
        });
      } else {
        uni.showToast({
          title: res.msg || res.reason || '当前验证失败',
          icon: 'none',
        });
      }
    },
  );
};

biometric.checkSupport((res) => {
  console.log('生物识别支持情况：', res);

  if (isIOS) {
    if (res.canAuth) {
      startBiometricAuth();
      return;
    }

    uni.showToast({
      title: res.reason || '当前设备不支持生物识别',
      icon: 'none',
    });
    return;
  }

  if (res.code === 0 && res.canAuth) {
    startBiometricAuth();
    return;
  }

  uni.showToast({
    title: res.reason || '当前设备不支持生物识别',
    icon: 'none',
  });
});
// #endif
```

## 八、API 说明

### checkSupport

检测当前设备是否支持生物识别。

#### 调用方式

```js
biometric.checkSupport(callback);
```

#### 回调字段

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Number | 状态码，`0` 表示可用 |
| canAuth | Boolean | 是否可以进行认证 |
| authType | String | 生物识别类型，可能为 `facial`、`fingerPrint`、`unknown` |
| needDeviceCredential | Boolean | 是否需要锁屏密码兜底，主要用于 iOS |
| reason | String | 不可用或特殊状态说明 |

### authenticate

发起生物识别认证。

#### 调用方式

```js
biometric.authenticate(options, callback);
```

#### options 参数

| 参数 | 类型 | 必填 | 平台 | 默认值 | 说明 |
|---|---|---|---|---|---|
| title | String | 否 | Android / iOS | `验证身份` | 系统认证弹窗标题或说明文案 |
| negativeText | String | 否 | Android | - | Android 取消按钮文案 |
| allowDeviceCredential | Boolean | 否 | iOS | `false` | 是否直接允许系统锁屏密码兜底 |
| showDeviceCredentialConfirm | Boolean | 否 | iOS | `true` | 生物识别锁定后，是否由原生层弹出“使用密码”确认框 |

#### allowDeviceCredential 说明

```js
allowDeviceCredential: false
```

表示优先只拉起 Face ID / Touch ID。若生物识别被锁定，插件会根据 `showDeviceCredentialConfirm` 决定是否弹出原生确认框。

```js
allowDeviceCredential: true
```

表示直接使用 iOS 系统的设备所有者认证策略，系统可能自动允许锁屏密码兜底。该模式下，锁屏密码弹窗可能由 iOS 系统直接接管，不一定经过插件的自定义确认框。

如需由插件控制“是否使用密码”的确认流程，建议使用：

```js
{
  allowDeviceCredential: false,
  showDeviceCredentialConfirm: true
}
```

#### showDeviceCredentialConfirm 说明

```js
showDeviceCredentialConfirm: true
```

表示生物识别锁定后，由插件原生层弹出确认框，引导用户选择是否使用锁屏密码。

```js
showDeviceCredentialConfirm: false
```

表示生物识别锁定后，不弹原生确认框，直接回调锁定状态，由前端自行处理。

#### callback 回调字段

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Number | 认证状态码，`0` 表示成功 |
| msg | String | 认证结果说明 |
| reason | String | 失败原因，部分场景返回 |

## 九、常见返回结果

### 认证成功

```js
{
  code: 0,
  msg: '验证成功'
}
```

### 用户取消

```js
{
  code: -2,
  msg: '用户取消'
}
```

### 生物识别锁定

```js
{
  code: -8,
  msg: '验证次数过多已锁定'
}
```

### iOS 生物识别已锁定，可使用密码验证

```js
{
  canAuth: true,
  needDeviceCredential: true,
  reason: '生物识别已锁定，可使用锁屏密码验证'
}
```

### 设备不支持

```js
{
  canAuth: false,
  reason: '设备不支持生物识别'
}
```

### 未录入生物识别

```js
{
  canAuth: false,
  reason: '未录入生物特征'
}
```

## 十、iOS 锁定场景说明

### 场景一：进入认证前已锁定

1. 调用 `checkSupport`。
2. 插件返回 `canAuth: true`、`needDeviceCredential: true`。
3. 前端继续调用 `authenticate`。
4. 插件根据 `showDeviceCredentialConfirm` 决定是否弹出“使用密码”确认框。

### 场景二：本次认证连续失败后锁定

1. 用户拉起 Face ID / Touch ID。
2. 用户连续多次识别失败。
3. 系统提示验证次数过多。
4. 用户关闭系统生物识别弹窗。
5. 插件主动复检当前是否已锁定。
6. 若已锁定，则根据 `showDeviceCredentialConfirm` 决定是否弹出“使用密码”确认框。

### 场景三：用户点击“使用密码”

插件拉起系统锁屏密码验证。

- 密码验证成功：返回 `code: 0`
- 用户取消密码验证：返回 `用户取消`
- 密码验证不可用：返回 `设备密码不可用`

## 十一、注意事项

1. 插件仅支持 App 端，不支持 H5、小程序端。
2. Android 最低支持 API 23。
3. iOS 最低支持 iOS 11.0。
4. 使用原生插件后，必须重新制作自定义运行基座。
5. iOS 模拟器可能无法完整模拟真实 Face ID / Touch ID 场景，建议使用真机测试。
6. iOS Face ID / Touch ID 锁定属于系统机制，插件只能根据系统返回状态进行处理。
7. 如果基座未包含插件，调用时会提示当前运行基座不包含原生插件，需要重新制作自定义基座。
8. Android 和 iOS 均使用 `BiometricAuth` 作为 JS 调用模块名。
9. 如需由插件控制锁屏密码确认流程，iOS 推荐传入 `allowDeviceCredential: false` 与 `showDeviceCredentialConfirm: true`。

## 十二、版本说明

### v1.0.0

- 支持 Android 生物识别认证。
- 支持 iOS Face ID / Touch ID 认证。
- 支持检测生物识别可用状态。
- 支持 iOS 生物识别锁定后使用锁屏密码兜底。
- 支持 Face ID / Touch ID 失败次数过多后的锁定状态主动复检。
- 支持通过 `showDeviceCredentialConfirm` 控制 iOS 原生密码确认弹窗。
- 支持 Android / iOS 统一模块名调用：`BiometricAuth`。

## 十三、文件编码说明

本文档及插件相关源码文件建议统一使用 **UTF-8 无 BOM** 编码保存，避免中文说明、中文提示文案在不同编辑器或打包环境中出现乱码。
