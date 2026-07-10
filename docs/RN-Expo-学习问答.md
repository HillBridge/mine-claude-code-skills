# React Native / Expo 入门问答笔记

> 整理时间：2026-07-09
> 主题：跨平台 App 开发（React Native + Expo）入门、发布、环境搭建

---

## Q1：React Native 和 Flutter，哪个更适合新手开发 App？

**快速结论**

- 没有任何前端/移动端基础 → **Flutter**
- 已经会 JavaScript / React / 前端 → **React Native**

**对比**

- **Flutter（Dart 语言）**：一套代码两端 UI 高度一致，文档和组件库完整，热重载体验好，环境配置省心；缺点是要学新语言 Dart（但几天能上手）。
- **React Native（JS/TS）**：会 JS/React 几乎零成本，生态庞大、社区活跃、可复用 Web 知识；缺点是依赖原生模块时配置繁琐、两端 UI 一致性要额外处理。

**建议**：零基础选 Flutter，有前端基础选 React Native。两者都能做出商用级 App，先做出第一个完整小项目最重要。

---

## Q2：React Native 和 Flutter 发布有哪些坑？

**通用坑（两者都有）**

- iOS：需付费开发者账号（$99/年）；证书 / Provisioning Profile 配置复杂；`Info.plist` 权限用途说明漏写会被拒；隐私清单（Privacy Manifest）强制；审核严格。
- Android：**签名 keystore 必须备份**（丢了无法更新已发布 App）；需上传 App Bundle(.aab)；targetSdkVersion 要跟上；缺隐私政策会被拒。
- 共性：Debug 正常但 Release 崩溃（混淆/压缩后暴露）；release 误指向测试环境；包体积偏大。

**Flutter 特有**：release tree-shaking 裁掉反射代码（需 `--obfuscate` + keep 规则）；CocoaPods / M 芯片架构问题；assets 忘在 `pubspec.yaml` 声明；插件某端未实现；引擎带来基础体积。

**React Native 特有**：JS bundle 打包导致「debug 正常 release 白屏」；Hermes 引擎差异；原生依赖版本冲突（Gradle/Pod）；升级 RN 版本痛苦；ProGuard/R8 裁掉原生方法名；新旧架构过渡兼容问题；OTA 热更新受苹果政策约束。

**给新手建议**：尽早跑通一次完整 release 打包；证书/keystore 立刻备份；RN 新手用 Expo（EAS Build）屏蔽大部分坑；上架前对照审核指南自查。

---

## Q3：频繁更新发包会有限制或问题吗？

**走商店正常发版**

- iOS：每次更新都要重新审核（数小时~1~2 天）；无硬性次数限制但频繁提交易触发人工复审；加急审核次数有限。
- Android：审核较快，支持分阶段发布（staged rollout）。
- 共性：线上长期存在多版本 → 后端需**版本兼容**；强制更新逻辑要提前设计。

**热更新（真正的「频繁更新」方案）**

- 能做：改 JS/Dart 逻辑、UI、修 bug，不过审核，秒级下发，可灰度、可回滚。
- 限制：苹果政策红线——只能修复/改进现有功能，**不能改变主要功能**；只能改 JS/Dart 层，**动原生必须走商店**；RN 的 CodePush 已停维护（改用 Expo Updates），Flutter 用第三方 Shorebird；无灰度易全量翻车。

**实践**：小修 bug/改文案走热更新；加功能/动原生走正常发版；商店版控制节奏（如 1~2 周一版）；做灰度 + 快速回滚；后端多版本兼容。

---

## Q4：频繁上 bug（修复）和上新功能有什么区别？

核心区别：**改动性质不同，决定能否绕开商店审核、风险高低**。

| 维度 | 修 bug | 上新功能 |
|---|---|---|
| 改动层 | 多为 JS/Dart 层 | 常涉及原生层 |
| 能否热更新 | 通常可以 | 通常不行 |
| 过商店审核 | 可绕开 | 必须 |
| 苹果政策 | 允许 | 热更新会违规 |
| 发布速度 | 秒级 | 数小时~1天 |
| 风险 | 低 | 高 |
| 灰度需求 | 建议 | 必须 |

**组合打法**：日常线上 bug 用热更新即时修；新功能按版本节奏走商店发布并带上累积的修复；新功能配灰度 + feature flag；后端始终兼容多版本。

一句话：**修 bug 追求「快」靠热更新；上新功能追求「稳」靠商店审核 + 灰度**。

---

## Q5：React Native 如何快速开始？

**最省心路径 = Expo**

前置：安装 Node.js（LTS）；手机装 Expo Go App。

```bash
# 创建项目（官方脚手架）
npx create-expo-app@latest myapp
cd myapp
npx expo start
```

启动后扫码即可在真机运行（改代码实时刷新）；模拟器按 `i`(iOS) / `a`(Android)。

**为什么用 Expo**：前期不用装 Xcode/Android Studio；屏蔽原生打包和证书坑；打包用 EAS Build 一条命令；内置 OTA 热更新。

**学习顺序**：核心组件（View/Text/Image/ScrollView/TextInput）→ 样式（StyleSheet + Flexbox）→ 路由（Expo Router）→ 列表（FlatList）→ 网络请求（fetch/axios）→ 状态管理（useState/useContext → Zustand）。

**资源**：reactnative.dev（学组件）+ docs.expo.dev（学工程化）。

---

## Q6：Expo 是什么？有哪些能力/不具备哪些能力/优缺点？除了 Expo 还有哪些开发方式？

**Expo 是什么**：围绕 React Native 的一套工具链 + 云服务 + 运行时。本质还是 RN，帮你封装好原生层。组成：

- **Expo SDK**：预封装原生能力库（相机/定位/通知等）
- **Expo Go**：手机宿主 App，扫码跑代码用于调试
- **EAS**：云服务（Build 打包 / Submit 上架 / Update 热更新）
- **Expo Router**：文件路由
- **CLI 工具**

**能做什么**：一条命令创建/运行/真机预览（前期不装 Xcode/AS）；SDK 覆盖多数常见原生能力；云端打包；OTA 热更新；一套代码出 iOS/Android/Web；支持 prebuild / dev client / config plugin 接入第三方原生库。

**不具备/受限**：托管流程不能随意写自定义原生代码（需转 dev client/prebuild）；冷门原生 SDK 接入麻烦；包体积偏大；深度定制原生工程不如纯原生自由；依赖 EAS 云服务（有额度/排队）；SDK 版本升级有节奏。

**优点**：上手快、体验好、官方维护、文档质量高。
**缺点**：深度原生定制不够自由、包体积大、依赖云服务、遇未覆盖原生需求成本反而高。

**除 Expo 外的其他方式**

1. **纯 React Native（Community CLI）**：`npx @react-native-community/cli init`，完全自由但环境/打包/升级坑全自己趟。
2. **Expo 的不同档位**：Managed（全托管）/ **Dev Client（定制宿主，可加原生，目前最推荐）** / Prebuild（生成原生工程）。
3. **从 Expo 弹出（prebuild/eject）**：接管原生工程，退回接近纯 RN。
4. **集成到现有原生 App**：RN 作为模块嵌入原生项目（大厂常用）。

**结论**：主流方向是 **Expo（配合 Dev Client）**，既省心又解决了「不能用原生模块」的老问题。

---

## Q7：EAS Build / Submit / Update 三者的关系和区别？

一句话区分：

| 工具 | 干什么 | 类比 |
|---|---|---|
| **EAS Build** | 打包成安装包（.ipa/.aab） | 做出成品 |
| **EAS Submit** | 上传到 App Store / Google Play | 送货上架 |
| **EAS Update** | 给已上架 App 推热更新（JS 层） | 售后打补丁 |

**流程位置**

```
写代码
  → EAS Build   产出 .ipa/.aab          【打包】
  → EAS Submit  上传商店，走审核         【上架】
  →（App 已在用户手机上）
  → EAS Update  推送 JS bundle，秒级更新 【热更新】
```

- **Build + Submit 是一对**：改原生就得重走这套（需审核）。
- **Update 是独立补丁通道**：只改 JS/资源，免审核，但受苹果政策约束、动原生无效。
- **判断关键**：改动是否涉及原生层？只改 JS → Update；动原生 → Build + Submit。

**节奏**：新功能/加原生库 → build → submit → 等审核；日常纯 JS bug → update 秒级下发。

---

## Q8：真正开发中如何搭建测试和生产两套环境？

核心解决三件事：**变量隔离、构建区分、两个 App 能共存**。

**需要隔离**：API 地址、App 名称/图标、Bundle ID/Package Name（**必须不同**）、第三方 key、热更新 channel。

**Expo 方案**

1. `eas.json` 定义多套 profile：

```jsonc
{
  "build": {
    "staging":    { "channel": "staging",    "env": { "APP_ENV": "staging" } },
    "production": { "channel": "production", "env": { "APP_ENV": "production" } }
  }
}
```

```bash
eas build --profile staging --platform all
eas build --profile production --platform all
```

2. `app.config.js` 按环境动态改配置（名字、Bundle ID、图标、apiUrl）——Bundle ID 不同 → 两个 App 可同装一机。

3. 代码读取：`Constants.expoConfig.extra.apiUrl`，或用 `EXPO_PUBLIC_` 前缀环境变量（`process.env.EXPO_PUBLIC_API_URL`）。

4. 热更新按环境推：`eas update --branch staging` / `--branch production`，互不干扰。

**纯 RN 做法**：iOS 用 Xcode Scheme + Configuration + `.xcconfig`；Android 用 Gradle `productFlavors`（自动生成不同 applicationId）；变量注入用 `react-native-config` 读 `.env.staging`/`.env.production`。

**关键提醒**：Bundle ID/applicationId 必须区分；敏感 key 用环境变量 + EAS Secrets 不硬编码；图标/名字做区分；后端也要有对应环境；热更新 channel 严格对应。

---

## Q9：Expo 项目初始化后的架构图（各模块标识）

> 基于 `npx create-expo-app` 官方默认模板（Expo Router + TypeScript）

```
rn-demo/
├── app/                          # 【核心】页面目录 — 文件即路由（Expo Router）
│   ├── (tabs)/                   #   路由分组：底部 Tab 导航（括号名不进 URL）
│   │   ├── _layout.tsx           #     Tab 布局：几个 Tab、图标、标题
│   │   ├── index.tsx             #     第一个 Tab 页面（首页 "/"）
│   │   └── explore.tsx           #     第二个 Tab 页面（"/explore"）
│   ├── _layout.tsx               #   根布局：全局 Provider、主题、字体加载入口
│   ├── +not-found.tsx            #   404 兜底页
│   └── modal.tsx                 #   示例模态页
│
├── components/                   # 【复用】通用 UI 组件
│   ├── ui/                       #   基础 UI 原子组件（图标、按钮等）
│   │   ├── icon-symbol.tsx       #     跨平台图标封装
│   │   └── collapsible.tsx       #     可折叠面板等示例组件
│   ├── themed-text.tsx           #   跟随主题的文本组件
│   ├── themed-view.tsx           #   跟随主题的容器组件
│   ├── parallax-scroll-view.tsx  #   视差滚动示例组件
│   └── haptic-tab.tsx            #   带触感反馈的 Tab 按钮
│
├── constants/                    # 【常量】全局常量
│   └── theme.ts                  #   颜色 / 主题 / 字体常量定义
│
├── hooks/                        # 【逻辑】自定义 React Hooks
│   ├── use-color-scheme.ts       #   读取系统深色/浅色模式
│   └── use-theme-color.ts        #   根据主题返回对应颜色
│
├── assets/                       # 【静态资源】图片、字体、图标
│   ├── images/                   #   图片（含 App 图标、启动图源文件）
│   │   ├── icon.png              #     App 图标源文件
│   │   ├── splash-icon.png       #     启动屏图标
│   │   ├── adaptive-icon.png     #     Android 自适应图标
│   │   └── favicon.png           #     Web 版网站图标
│   └── fonts/                    #   自定义字体文件
│       └── SpaceMono-Regular.ttf
│
├── scripts/                      # 【工具脚本】项目维护脚本
│   └── reset-project.js          #   一键清空示例、重置为空白项目
│
├── app.json                      # 【配置★】Expo 应用配置（名称/图标/BundleID/权限/插件）
│                                 #   多环境时常换成 app.config.js 动态生成
├── package.json                  # 【依赖】npm 依赖 + 脚本命令
├── tsconfig.json                 # 【配置】TS 编译配置 + 路径别名 @/*
├── eslint.config.js              # 【规范】ESLint 代码检查配置
├── expo-env.d.ts                 # 【类型】Expo 自动生成类型声明（勿手改）
├── .gitignore                    # Git 忽略规则
└── README.md                     # 项目说明
```

> 真实创建后还会生成 `node_modules/`（依赖）、`.expo/`（本地缓存），二者不进版本库。

**各模块作用速记**

| 目录/文件 | 角色 | 是否常动 |
|---|---|---|
| `app/` | 页面路由，加文件即加页面 | ⭐ 天天动 |
| `components/` | 复用 UI 组件 | ⭐ 经常动 |
| `hooks/` | 复用逻辑 | 经常动 |
| `constants/` | 颜色、主题等常量 | 偶尔 |
| `assets/` | 图片字体资源 | 偶尔 |
| `app.json` | App 配置中心 | 打包前动 |
| `package.json` | 依赖和命令 | 装库时动 |
| `tsconfig.json` / `eslint.config.js` | 工程配置 | 很少动 |
| `scripts/reset-project.js` | 清空示例代码 | 用一次 |

**真实开发常补充**

```
rn-demo/
├── app.config.js        # 替代 app.json，按 APP_ENV 动态改名字/BundleID
├── eas.json             # EAS 构建配置：staging / production 两套 profile
├── .env.staging         # 测试环境变量
├── .env.production      # 生产环境变量
└── src/                 # 很多团队把 components/hooks/constants 收进 src/
    ├── api/             #   接口层
    ├── features/        #   业务模块（feature 分层）
    └── ...
```

**核心记忆**：`app/` 管页面路由、`components/`+`hooks/` 管复用、`app.json` 管应用配置；工程化配置大多一次配好很少动。

---

## Q10：默认 Expo 架构适合多人协作吗？有什么缺点？可以怎么优化？

**结论**：起步够用，但**不适合多人协作的中大型项目**，需要改造。默认模板是官方为「演示 + 快速上手」设计的，扁平、按技术类型分目录，人一多、功能一多就暴露问题。

**缺点**

1. **按「技术类型」分目录而非「业务」分（最大问题）**：一个功能的代码散落在 `components/`、`hooks/`、`app/` 多处；目录会膨胀成大杂烩；多人改同一目录易 Git 冲突；难以整块删除/迁移。
2. **缺少明确分层**：没有 `api/`、`services/`、`store/`，网络请求和业务逻辑容易塞进组件，导致组件臃肿、逻辑难复用。
3. **没有工程化协作约束**：缺 `eas.json`、`.env.*`、统一的 Prettier/commit 规范/CI；路径别名只有 `@/*` 粒度太粗。
4. **示例代码混在里面**：parallax、explore、modal 等演示代码干扰真实开发。
5. **测试目录缺失**：没有 `__tests__/` 约定。

**优化：改造成 Feature-Sliced（按业务分层）结构**

```
rn-demo/
├── app/                        # 只放路由页面（页面只做组装，不写业务逻辑）
├── src/
│   ├── features/               # 【核心】按业务模块分，每块自成一体
│   │   └── order/              #   订单功能
│   │       ├── components/     #     专属组件
│   │       ├── hooks/          #     专属 hook
│   │       ├── api/            #     专属接口
│   │       ├── constants/
│   │       ├── types.ts
│   │       └── __tests__/      #     该功能测试
│   ├── shared/                 # 跨功能共享（通用组件/hooks/utils/constants）
│   ├── api/                    # 基础设施（请求实例、拦截器、通用接口）
│   ├── services/               # 业务服务层
│   ├── store/                  # 全局状态（Zustand/Redux）
│   └── theme/                  # 主题、颜色、字体
├── assets/
├── app.config.js               # 替代 app.json，多环境动态配置
├── eas.json                    # staging / production 构建 profile
├── .env.staging / .env.production
├── .prettierrc / commitlint    # 团队代码/提交规范
└── tsconfig.json               # 更细的路径别名
```

**配套改造清单**

| 优化项 | 做法 | 解决什么 |
|---|---|---|
| 业务分层 | 引入 `src/features/*` | 功能内聚，减少 Git 冲突 |
| 共享层 | `src/shared/` | 复用清晰，避免重复 |
| 接口分层 | `src/api/` + feature 内 `api/` | 请求逻辑独立于 UI |
| 状态管理 | 加 `src/store/`（Zustand 轻量） | 全局状态统一去处 |
| 多环境 | `eas.json` + `app.config.js` + `.env.*` | 测试/生产隔离 |
| 路径别名 | tsconfig 配 `@/features` `@/shared` 等 | import 清爽 |
| 规范 | Prettier + ESLint + commitlint + CI | 多人风格统一 |
| 清理示例 | `npm run reset-project` | 去演示代码干扰 |
| 测试约定 | 每个 feature 内建 `__tests__/` | 测试有落脚点 |

**一句话**：默认结构「demo 友好、团队不友好」；多人协作要改成 Feature-Sliced（按业务分层）+ shared 共享层 + 清晰的 api/store 分层 + 多环境配置 + 团队规范。

---

## Q11：原生功能在哪里？如何调用原生功能？

核心：**大部分原生能力不用碰原生代码，直接 import JS 包调用**。原生功能分四个层级，从省事到硬核：

```
省事 ←──────────────────────────────────────→ 硬核
① Expo SDK      ② 第三方原生库    ③ Config Plugin    ④ 自写原生模块
（装了就用）    （社区封装好）     （改原生配置）      （写 Swift/Kotlin）
```

**① Expo SDK —— 官方封装好的原生能力（最常用，覆盖约 90% 需求）**

npm 包形式，装完直接 import，不碰原生代码。

```bash
npx expo install expo-camera expo-location
```

```js
import * as Location from 'expo-location'
const { status } = await Location.requestForegroundPermissionsAsync()
const loc = await Location.getCurrentPositionAsync()
```

**② 第三方原生库 —— 社区封装的原生能力**

Expo SDK 没覆盖的用社区库；含原生代码，**不能在 Expo Go 跑**，需 Dev Client。

```bash
npx expo install react-native-xxx
npx expo prebuild
eas build --profile development
```

**③ Config Plugin —— 通过配置修改原生工程**

需要改 `Info.plist`/`AndroidManifest.xml`/加权限时，写进 `app.config.js` 的 `plugins`，prebuild 时自动应用，仍不用手写原生。

```js
plugins: [
  'expo-camera',
  ['expo-location', { locationAlwaysAndWhenInUsePermission: '需要定位…' }],
]
```

**④ 自写原生模块 —— 真正的原生代码（最少用到）**

以上都满足不了才写，推荐用 Expo Modules API（比传统 Bridge 简单）。位置在 `modules/`（本地模块）或 `ios/`+`android/`。

```bash
npx create-expo-module my-native-module --local
```

**位置与能否在 Expo Go 运行**

| 层级 | 物理位置 | 你写什么 | Expo Go 能跑 |
|---|---|---|---|
| ① Expo SDK | node_modules（import） | 只写 JS | ✅ |
| ② 第三方原生库 | node_modules + prebuild | 只写 JS | ❌ 需 Dev Client |
| ③ Config Plugin | app.config.js 的 plugins | 写配置 | ❌ 需 Dev Client |
| ④ 自写原生模块 | modules/ 或 ios/+android/ | Swift/Kotlin + JS | ❌ 需 Dev Client |

**权限处理**：调用原生能力几乎都要「运行时请求 + 声明用途」。

```js
const { status } = await Camera.requestCameraPermissionsAsync()   // 运行时请求
```
```js
ios: { infoPlist: { NSCameraUsageDescription: '需要相机以拍摄凭证' } }  // 声明用途，iOS 必写
```

**实用建议**：从上往下选，能用高层就别下沉——先查 Expo SDK → 社区原生库 → config plugin → 最后才自写原生。新手和多数产品停在 ①②③ 层就够，几乎不用写 Swift/Kotlin。

---

## Q12：React Native 写的代码和 React 代码有什么区别和相似点？

**共享同一套语言和思想内核，区别主要在「渲染成什么」和「用什么标签/样式」。会 React 学 RN 很快（1~2 天）。**

**相似点（几乎照搬）**

- 同一套 React 核心：组件、JSX、props、state、Hooks（useState/useEffect/useContext…）、单向数据流
- 状态管理库通用：Zustand、Redux、React Query
- 非 UI 库直接复用：axios、dayjs、lodash、zod、i18n
- TypeScript / ESLint / Prettier / 函数式组件写法一致

**区别（需要重新学）**

1. **渲染目标不同**：Web 渲染成 DOM 跑在浏览器；RN 渲染成原生组件跑在 App 里，**没有 DOM**。

2. **标签不同（没有 div/span）**：

| React(Web) | React Native | 说明 |
|---|---|---|
| `<div>` | `<View>` | 容器 |
| `<span>`/`<p>` | `<Text>` | 所有文字必须包在 Text 里 |
| `<img>` | `<Image>` | 图片 |
| `<input>` | `<TextInput>` | 输入框 |
| `<button>` | `<Pressable>`/`<TouchableOpacity>` | 按钮 |
| 自动滚动 | `<ScrollView>`/`<FlatList>` | 滚动要显式组件 |
| `<a>` | 路由 `<Link>` | 无超链接概念 |

3. **样式不同（没有 CSS）**：用 `StyleSheet.create` + JS 对象；无单位数字；默认全是 Flexbox 且默认竖向（column）；属性驼峰命名；**无级联、无继承**。

```js
const styles = StyleSheet.create({
  box: { flex: 1, backgroundColor: '#fff', padding: 16 },  // 驼峰、无单位
})
```

4. **事件不同**：`onClick`→`onPress`；`onChange`→`onChangeText`；无 hover；要处理软键盘遮挡（KeyboardAvoidingView）。

5. **平台与 API 差异**：没有 window/document；存储用 `AsyncStorage` 而非 localStorage；路由用 React Navigation/Expo Router；区分平台用 `Platform.OS` 或 `Xxx.ios.js`/`Xxx.android.js`。

6. **布局细节**：默认 Flexbox 且竖向排列；没有 grid/float/fixed（有 absolute）；长列表必须用 `FlatList`（虚拟化），不能直接 map 一堆元素。

**关系图**

```
        共享内核（几乎照搬）
   ┌─────────────────────────────┐
   │ React 组件 / JSX / Hooks     │
   │ 状态管理 / TS / 业务逻辑库    │
   └─────────────────────────────┘
              │
      ┌───────┴────────┐
      ▼                ▼
  React (Web)     React Native
  ├ div/span/img   ├ View/Text/Image
  ├ CSS 样式       ├ StyleSheet(JS对象)
  ├ onClick        ├ onPress
  ├ DOM / 浏览器   ├ 原生组件 / App
  └ 路由=URL       └ 路由=Navigation
```

**一句话**：逻辑层完全一样，视图层需要重学。React 教你「怎么思考组件和状态」100% 复用；RN 只是把渲染标签 + 样式 + 事件名 + 平台 API 换成移动端的一套。

---

## Q13：Expo SDK 官方原生能力对 iOS 和 Android 的兼容性做得怎么样？

**结论**：跨平台兼容性做得**相当扎实**，是 Expo 的核心价值——同一份 JS 代码两端大多直接跑、行为一致。少数差异来自操作系统本身，抹不平。

**兼容性好的部分**（写一套 JS 两端通用）

| 能力 | iOS | Android | 一致性 |
|---|---|---|---|
| 相机 / 定位 / 文件系统 | ✅ | ✅ | 高 |
| 图片选择 / AsyncStorage | ✅ | ✅ | 高 |
| 安全存储 secure-store | ✅ Keychain | ✅ Keystore | 高（底层不同 API 一致） |
| 生物识别 | ✅ FaceID/TouchID | ✅ 指纹/人脸 | 高 |
| 传感器 / 设备信息 / 网络 | ✅ | ✅ | 高 |
| 通知 | ✅ | ✅ | 中高 |

**需注意平台差异的部分**（系统层面差异，谁都抹不平）

1. **权限模型**：iOS 用途写 `Info.plist`（漏写拒审），Android 写 `AndroidManifest.xml`，Android 13+ 通知/精确定位有新规则。
2. **推送差异大**：iOS 走 APNs（须真机测、配证书），Android 走 FCM（配 `google-services.json`），前后台展示行为不一致。
3. **后台任务/定位**：iOS 省电策略严格，后台能力弱于 Android。
4. **系统 UI**：返回手势（iOS 侧滑 vs Android 物理键）、安全区（刘海/灵动岛，用 `react-native-safe-area-context`）、系统组件外观不同。
5. **少数模块单端支持**：Expo 文档每个 API 页顶部有平台支持标记（iOS/Android/Web），用前先看。

**应对平台差异**

```js
import { Platform } from 'react-native'
const padding = Platform.OS === 'ios' ? 20 : 16
const shadow = Platform.select({
  ios: { shadowColor: '#000', shadowOpacity: 0.1 },
  android: { elevation: 4 },
})
// 或分平台文件：Button.ios.tsx / Button.android.tsx → import './Button'
```

**对比第三方原生库**：官方 SDK 两端都测、有保障；第三方库兼容性参差不齐（可能只支持一端或某端有坑），选库要核实 star、更新频率、issue 里两端反馈。

**一句话**：官方 Expo SDK 的 iOS/Android 兼容性很扎实，常见能力一套 JS 两端通用；剩下差异主要来自操作系统本身（权限/推送/后台/系统 UI），用 `Platform` API 或分平台文件处理即可。

---

## Q14：React Native 写的代码都是原生的吗？与 React JS 通过 WebView 内嵌有什么区别？

**RN 是原生的吗？——一半是：原生渲染 + JS 逻辑**

- **UI 层是真原生**：`<View>`/`<Text>` 最终渲染成 iOS 的 `UIView`、Android 的 `android.view` 等真正的原生控件，不是网页元素。
- **逻辑层是 JS**：业务代码跑在 JS 引擎（Hermes）里，通过桥接（旧 Bridge / 新架构 JSI）和原生通信。
- 准确说法：**原生渲染 + JS 逻辑**，不是 100% 原生（那是直接写 Swift/Kotlin），也不是网页。

```
纯原生(Swift/Kotlin)   React Native        WebView 混合(Cordova/Ionic)
   全原生 UI+逻辑      原生 UI + JS 逻辑      网页 UI + JS 逻辑（套壳）
   ←──────────────── 越来越不"原生" ────────────────→
```

**RN vs WebView 内嵌 全面对比**

对象：RN（JS 逻辑 + 原生渲染） vs WebView 混合（把网页塞进 App 的 WebView 里显示，代表 Cordova/Ionic/Capacitor/内嵌 H5）。

| 维度 | React Native | WebView 内嵌 |
|---|---|---|
| **UI 渲染** | 原生控件 | 浏览器内核渲染 HTML/CSS |
| **本质** | JS 指挥原生画界面 | App 里开个浏览器显示网页 |
| **渲染性能** | 接近原生 | 受 WebView 限制，复杂页面卡 |
| **滚动/动画** | 流畅（FlatList/原生动画） | 长列表、复杂动画易掉帧 |
| **启动速度** | 较快 | WebView 初始化+网页加载，偏慢 |
| **交互手感** | 原生级 | 有「网页感」 |
| **原生能力** | 直接调用，上限高 | 经 Bridge 转发，受容器限制 |
| **技术栈** | React + RN 组件（需学差异） | 普通 Web（零迁移） |
| **Web 复用** | 逻辑复用，UI 重写 | 网页几乎原样复用 |
| **热更新** | 支持但受苹果政策约束 | 天然支持（改服务器网页即可） |
| **包体积** | 中 | 小（壳）或中 |
| **离线** | 好（代码打包本地） | 纯在线 H5 离线差 |

**怎么选**

- **RN**：要原生体验/性能、手势动画长列表多、硬件调用多、正式产品级 App。
- **WebView 混合**：已有成熟 Web 想快速套壳、内容型/营销页（变化频繁要随时更新）、团队只有 Web 能力、局部嵌 H5。

**三者关系**

```
                 UI 渲染         逻辑        体验    原生能力   更新灵活度
纯原生           原生控件        原生        ★★★★★   ★★★★★     低（必发版）
React Native     原生控件        JS(Hermes)  ★★★★☆   ★★★★☆     中（有限热更）
WebView 混合     浏览器渲染网页   JS          ★★☆☆☆   ★★☆☆☆     高（改网页即可）
```

**一句话**：RN 不是纯原生，但 UI 是真原生渲染、逻辑用 JS，性能体验远好于 WebView 套壳，同时保留 JS 效率；WebView 内嵌本质是「App 里开浏览器显示网页」，胜在 Web 复用和更新灵活，输在性能、原生感和能力上限。

---

## Q15：可以 RN + WebView 结合开发吗？如何区分场景？好处和坏处？

**可以，而且是业界主流做法**。RN 提供 `react-native-webview` 组件，很多大厂 App 都是「RN/原生外壳 + 局部 H5」。

**怎么结合**

```jsx
import { WebView } from 'react-native-webview'
<WebView source={{ uri: 'https://h5.myapp.com/activity/618' }} style={{ flex: 1 }} />
```

架构：RN 负责骨架和核心页面，WebView 承载某些页面/区块。

```
┌─────────────────────────────────┐
│   App 外壳（React Native）        │
│   ├ Tab 导航 / 路由（RN）         │
│   ├ 首页、个人中心（RN 原生渲染）  │
│   ├ 支付、扫码（RN + 原生模块）    │
│   └ [ WebView: 营销活动/文章/帮助中心（H5） ] │
└─────────────────────────────────┘
```

**如何划分场景**——判断标准：「变化频率」和「体验要求」哪个更高。

- **用 RN（原生渲染）**：首页/Tab 主框架、列表 Feed 流、登录/支付/扫码、相机/地图/复杂交互、个人中心。特征：高频交互、要原生体验、要调硬件、稳定不常改。
- **用 WebView（H5）**：营销活动页/大促会场、文章资讯/帮助中心、用户协议/隐私政策、运营配置/问卷、已有成熟 Web 业务。特征：内容型、变化快、要随时更新、体验要求不高、可复用现有网页。

> 划分原则：**核心和高频交互用 RN；边缘的、内容型的、变化快的用 H5。**「用户天天用的」用原生，「运营天天改的」用 H5。

**好处**

| 方面 | 好处 |
|---|---|
| 更新灵活 | H5 改服务器即可，活动页秒级上线不用审核 |
| 体验保底 | 核心页面 RN 保证门面和高频操作原生手感 |
| 开发效率 | 活动页交给 Web 团队快速产出 |
| 复用资产 | 已有 Web 业务直接嵌入不重写 |
| 分工清晰 | 原生/RN 团队 + H5 团队并行 |
| 降低发版压力 | 大量易变内容走 H5 |

**坏处 / 代价**

| 方面 | 问题 |
|---|---|
| 体验割裂 | H5 与 RN 手感不一致，用户能感知 |
| 通信复杂 | RN↔WebView 双向通信要搭 Bridge |
| 性能落差 | H5 加载慢、复杂交互卡，低端机/弱网明显 |
| 调试困难 | 出问题要判断 RN 层还是 H5 层 |
| 登录态同步 | token/用户信息要在两层间传递，易出 bug |
| 离线白屏 | 纯在线 H5 弱网白屏，需离线包兜底 |
| 审核风险 | H5 占比过高可能被判「不像原生 App」拒审 |
| 安全 | WebView 有 XSS/任意 URL 风险，要域名白名单 |

**RN ↔ WebView 通信（结合的核心难点）**

```jsx
// RN → H5：注入 JS / 传登录态；接收 H5 消息
<WebView
  source={{ uri: url }}
  injectedJavaScript={`window.APP_TOKEN='${token}';true;`}
  onMessage={(e) => {
    const data = JSON.parse(e.nativeEvent.data)
    if (data.type === 'pay') startNativePay(data.order)   // H5 请求调起原生支付
  }}
/>
```
```js
// H5 → RN：调用原生能力
window.ReactNativeWebView.postMessage(JSON.stringify({ type: 'pay', order: '123' }))
```

典型模式：H5 负责展示，需要原生能力（支付/扫码/分享）时通过 Bridge 通知 RN 执行。

**落地建议**：① 先分层（页面标注高频核心 vs 易变内容）；② 统一 Bridge 通信协议；③ 设计好登录态注入与过期同步；④ 重要 H5 做离线包兜底；⑤ 控制 H5 占比避免拒审、守住体验；⑥ WebView 加域名白名单。

**三种形态对比**

| 形态 | 描述 | 适合 |
|---|---|---|
| 纯 RN | 全部原生渲染 | 追求极致体验、交互复杂 |
| 纯 WebView 套壳 | 整个 App 就是网页 | 极简、纯展示、预算紧 |
| **RN + WebView 混合** | 核心 RN + 边缘 H5 | **绝大多数中大型 App 的现实选择** |

**一句话**：能结合且是主流最优解——RN 做核心骨架和高频/硬件页面保体验，WebView 承载易变/内容型/可复用页面换灵活和效率；代价是体验割裂、通信复杂、调试和登录态同步成本。关键是按「高频核心 vs 易变内容」清晰分层、统一 Bridge 规范、控制 H5 占比守住体验和审核底线。

---

## Q16：RN 的「原生组件 + JS」在热更新时能全部热更、不受商店审核吗？

**关键误区澄清：热更新只能更新 JS 和资源，原生组件（原生代码）无法热更新，不是「全部都能热更」。**

```
RN App = JS 层  +  原生层
          ↓          ↓
       ✅ 可热更   ❌ 不可热更（必须走商店审核）
```

热更新本质是**替换 App 里的 JS bundle**，原生二进制部分动不了。

| 改动内容 | 能否热更新 |
|---|---|
| JS 业务逻辑 / RN 组件 UI 样式 | ✅ 能 |
| 图片/字体等 JS 引用的资源 | ✅ 能 |
| 调用**已有**原生模块的 JS 代码 | ✅ 能（原生能力已在包里） |
| 新增原生模块/第三方原生库 | ❌ 不能 |
| 改原生配置（权限/Info.plist/Manifest） | ❌ 不能 |
| 升级 Expo SDK / RN 版本 | ❌ 不能 |
| 改 App 图标/名称/启动图 | ❌ 不能 |

**为什么原生组件不能热更**：`<View>`/`<Text>` 的原生实现早已编译进 App 二进制；热更新只换 JS，调用已有原生实现没问题；但新增原生能力的代码不在用户已装的包里，热更推过去 JS 调用它会崩溃，所以必须重新 build 发版。

**苹果 vs 安卓热更新政策差异**

| | 苹果 App Store | 安卓 Google Play |
|---|---|---|
| 允许 JS 热更新 | ✅ 但有红线 | ✅ 更宽松 |
| 红线 | 不能改变 App 主要功能/用途 | 相对宽松 |
| 违规后果 | 可能下架 | 相对少见 |

**判断口诀**：改 JS → 热更；动原生 → 发版。（对应「修 bug 多为 JS 用热更，上新功能常涉及原生走商店」）

---

## Q17：上生产动原生要重新打包走审核，那测试环境苹果/安卓也会审核吗，还是只有生产？

**结论：真正严格的审核只在生产上架。测试环境大多数分发方式不走或只走极简审核。触发审核的是「往哪个渠道分发」，不是「是否改了原生」。**

**iOS 测试分发**

| 方式 | 是否审核 |
|---|---|
| Development / Ad-hoc（注册 UDID 设备直装，上限 100） | ❌ 不审核 |
| Enterprise 分发 | ❌ 不审核 |
| TestFlight 内部测试（最多 100 人） | ❌ 基本不审核 |
| TestFlight 外部测试（最多 10000 人） | ⚠️ 首个 build 过一次轻量 Beta 审核，后续多免审 |
| App Store 生产 | ✅ 完整严格审核 |

**Android 测试分发**

| 方式 | 是否审核 |
|---|---|
| 直接装 APK（侧载） | ❌ 不审核 |
| Play 内部测试轨道（最多 100 人） | ⚠️ 极简/几乎即时 |
| 封闭测试 | ⚠️ 有审核但较快 |
| 开放测试 | ⚠️ 接近生产标准 |
| 生产轨道 | ✅ 完整审核 |

**EAS 对应做法**

```jsonc
{
  "build": {
    "development": { "developmentClient": true },  // 设备直装，不审核
    "preview":     { "distribution": "internal" }, // 内部测试包，不审核
    "production":  {}                              // 生产包，走审核
  }
}
```

**实务提醒**：测试包不审核，但每次改原生仍要重新 build（有耗时和 EAS 排队），测试期建议把原生改动攒批再打包，别改一行 build 一次。

---

## Q18：生产审核时安卓通过、iOS 没通过，会导致双端功能不一致吗？如何同步更新？

**是的，不加控制就会因审核时间差出现不一致**：安卓几小时通过、用户用上新功能，iOS 被拒/审核慢、用户停在旧版，差异可能持续几小时到几天。根因是两商店审核独立、时长不一（iOS 通常更慢更严）。

**解决方案（推荐→兜底）**

| 方案 | 能否根治时间差 | 改代码 | 灵活度 | 场景 |
|---|---|---|---|---|
| **Feature Flag + 远程配置** | ✅ 彻底 | 要 | 高（灰度/回滚） | 重要功能 ⭐首选 |
| 协调发布（手动放量） | ✅ 可以 | 否 | 中（人工盯） | 不想改代码 |
| 后端版本兼容控制 | ✅ 部分 | 后端要 | 高 | 功能依赖接口 |
| 接受差异+强制更新 | ❌ 不根治 | 少 | 低 | 非核心功能 |

**方案一（首选）Feature Flag**：新功能代码两端都发（开关默认关、功能隐藏）→ 等 iOS/Android 都审核通过发布 → 服务端一键打开开关 → 两端用户同时看到新功能。可灰度、可一键回滚。

**方案二 协调发布**：iOS 选「手动发布」（通过后不自动上线），Android 用分阶段发布/先 0% 放量，等两端都通过再一起放。不用改代码，但要人工盯，iOS 一直被拒会拖住 Android。

**组合打法**：Flag 包新功能两端提交（默认关）→ 后端做版本兼容兜底 → 两端通过发布 → 服务端统一开关灰度 10%→全量 → 出问题一键关。

---

## Q19：Feature Flag 的机制细节（开关依据 / 拉取方式 / 新旧代码 / 清理时机 / 是否需更新 App）

**① 服务端根据什么加开关配置**

存一套规则，客户端拉取时带上自身信息，服务端据此判断：

```jsonc
{
  "newPaymentFlow": {
    "enabled": true,
    "platform": ["ios", "android"],   // 平台：可只对某端开
    "minVersion": "2.0.0",            // 版本：只对 >=2.0 开
    "rolloutPercent": 10,             // 灰度比例
    "whitelist": ["userId_123"],      // 白名单（内测）
    "region": ["BR"],                 // 地区
    "startTime": "2026-07-15T00:00"   // 定时开启
  }
}
```

灰度选「哪 10%」用 `hash(userId) % 100 < rolloutPercent`，保证同一用户结果稳定。双端时间差就靠 `platform + minVersion` 控制。

**② 客户端拉配置是 HTTP 请求吗**

是，普通 HTTP 请求，通常 App 启动时拉一次：

```js
const res = await fetch('https://api.myapp.com/config', {
  headers: { 'X-Platform': Platform.OS, 'X-App-Version': appVersion, 'X-User-Id': userId },
})
```

优化：启动拉一次存内存；请求失败用本地缓存兜底；没拉到时新开关默认「关」退回旧功能。
```js
const config = await loadRemoteConfig().catch(() => getCachedConfig() ?? DEFAULT_CONFIG)
```

**③ 要同时保留新旧两套代码吗**

要。开关存续期内新旧代码共存，靠 `if/else` 切换，以支持随时回滚：
```jsx
if (flags.newPaymentFlow) return <NewPaymentFlow />
else return <OldPaymentFlow />   // 旧代码暂留
```
代价是包体积略增和分支代码——这是安全发布的固有成本。

**④ 什么时机清除旧代码**

新功能「全量放开 + 稳定 + 老版本淘汰」后清理：灰度 10%→50%→100% → 全量观察几天~两周确认稳定不回滚 → 开关写死 true 或删判断 → 删旧代码 → 服务端清 flag。最大坏味道是 flag 用完不清、越积越多，团队要建立例行清理机制，每个 flag 建立时就规划「死期」。

**⑤ 更新新版本，用户要手动更新还是打开自动同步**

取决于新功能在 JS 层还是原生层（回到热更新边界）：

| 情况 | 用户要做什么 | 生效方式 |
|---|---|---|
| 开关控制已随 App 发出的 JS 功能 | 什么都不用做 | 下次开 App 拉到新配置，自动显示 ✅ |
| 新功能是新写的纯 JS 代码 | 走热更新 | 下次开 App 自动拉新 bundle，免去商店 ✅ |
| 新功能涉及原生层 | 必须去商店更新 App | 下载新版本才有新功能 ❌ |

「下次打开自动同步」的前提是内容在 JS 层（开关或热更新）；一旦动原生，绕不开用户手动去商店更新。

**完整图景**

```
服务端配置中心(按 平台/版本/灰度/白名单 定规则)
        │ HTTP 请求(启动拉一次，带 平台+版本+userId，失败用缓存兜底)
        ▼
客户端(新旧两套代码共存，if(flag) 新 : 旧)
        ├─ 开关控制已发布的 JS 功能 → 下次开 App 自动同步，免更新 ✅
        ├─ 新 JS 代码 → 热更新推送，下次开 App 自动生效，免去商店 ✅
        └─ 动了原生 → 必须去商店更新 App ❌
        ▼
功能稳定 + 全量 + 老版本淘汰 → 清理旧代码和 flag
```

---

## Q20：React Native 打包后产物和 React 打包后产物有什么异同？

**核心区别**：React 打包产物是「网页静态资源」给浏览器跑；RN 打包产物是「原生安装包」，里面塞了一个 JS bundle，给手机装。**相同点**：两者内部都有一个打包压缩后的 JS bundle。

**React(Web) 打包产物**（`npm run build` → `dist/`）

```
dist/
├── index.html              # 入口 HTML
├── assets/
│   ├── index-a1b2c3.js     # 打包压缩后的 JS bundle(带 hash)
│   ├── vendor-d4e5f6.js     # 第三方库拆出的 chunk
│   ├── index-x7y8z9.css     # 提取的 CSS
│   └── logo-1234.png
└── favicon.ico
```

面向浏览器，通过 URL 访问；可 code splitting 按路由拆多个 JS chunk 懒加载；部署 = 传服务器/CDN。

**React Native 打包产物**（`eas build` → `.ipa`/`.aab`/`.apk`）

```
MyApp.apk(解压后)
├── classes.dex                  # 编译后的原生代码
├── lib/                          # 原生 .so 库(RN引擎、Hermes引擎等)
├── assets/
│   └── index.android.bundle      # ★ JS 代码打包成的 bundle
├── res/                          # 原生资源(图标/启动图)
└── AndroidManifest.xml
```

面向手机操作系统，要安装；部署 = 上架商店。

**异同对比**

| 维度 | React(Web) | React Native |
|---|---|---|
| 产物形态 | 静态文件目录 | 原生安装包 |
| 入口 | index.html | 原生 App 启动→加载 JS bundle |
| JS bundle | ✅ 有 | ✅ 有 |
| CSS | ✅ 独立文件 | ❌ 无，样式编进 JS(StyleSheet) |
| HTML | ✅ 有 | ❌ 无 |
| 原生代码/引擎 | ❌ 无 | ✅ 有(.so/.dex + Hermes) |
| 运行环境 | 浏览器 | 手机OS + RN 运行时 |
| 交付方式 | 传服务器/CDN | 上架应用商店 |
| 更新方式 | 传新文件即最新 | 商店发版；JS 部分可热更 |
| 体积 | 小 | 大(含引擎) |
| 拆包/懒加载 | ✅ 成熟 | 有限(单一 bundle 为主) |

**最关键的相同点**：都有 JS bundle。Web 用 Vite/Webpack 打包，RN 用 **Metro**（RN 专属打包器），都做模块合并、压缩、Tree-shaking。**热更新的本质就是替换这个 JS bundle**——这正是为什么 RN 只有 JS 层能热更：原生部分(.so/.dex)换不了。

**一句话**：相同是内部都有打包器生成的 JS bundle；不同是 React 产物是给浏览器的静态网页文件，RN 产物是给手机的原生安装包（原生引擎+JS bundle+原生资源），没有 HTML/CSS，体积大得多，更新要走商店（仅 JS bundle 可热更）。

---

## Q21：React Native 更新 JS bundle 有缓存问题需要处理吗？

**结论**：Web 前端那套「浏览器缓存/CDN缓存/chunk hash/强缓存白屏」的痛点在 RN 基本不存在，热更新框架（EAS Update）自动管好了大部分缓存；但有几个「更新时机」和「一致性」问题需要手动处理。

**对比 Web 痛点**

| Web 前端缓存问题 | RN 有没有 |
|---|---|
| 浏览器强缓存旧 JS 导致白屏 | ❌ 没有(不经浏览器) |
| 文件名要加 hash 防缓存 | ❌ 不用管(框架内部处理版本) |
| CDN 缓存需刷新 | ❌ 没有 CDN 那层 |
| chunk 懒加载失败 | ❌ 基本没有(单一 bundle) |

原因：RN bundle 不是浏览器按 URL 请求的静态文件，而是**下载到设备本地存起来、由热更新框架管理版本**。

**bundle 更新机制**

```
App 启动
  ├─ 加载本地已缓存的 bundle(先跑起来,不卡启动)
  ├─ 后台静默检查服务器有没有新 bundle → 有则下载到本地缓存(不影响当次运行)
  └─ 下次启动 → 加载刚下载的新 bundle 生效
```

关键点：bundle 缓存在设备本地，框架自己管新旧版本；**默认「下次启动生效」**，这次运行还是旧的；框架做了原子切换+回滚，新 bundle 下载不完整/加载失败会自动回退。

**真正需要处理的三件事**

1. **更新生效时机**：默认下次冷启动才生效；想立即生效要手动调用：
```js
const update = await Updates.checkForUpdateAsync()
if (update.isAvailable) {
  await Updates.fetchUpdateAsync()
  await Updates.reloadAsync()   // 立即重启生效
}
```
2. **runtimeVersion 匹配**：JS bundle 依赖对应版本的原生代码，用错会崩溃（详见 Q22）。
3. **多版本兼容**：灰度期不同用户 bundle 版本不同，后端要兼容多版本。

远程图片等资源的缓存与 bundle 更新无关，走各自的图片缓存策略（如 expo-image）。

**一句话**：RN 基本没有 Web 那种缓存导致白屏的痛点，bundle 由 EAS Update 自动管版本、原子切换、回滚；真正要处理的是更新生效时机、runtimeVersion 匹配、多版本兼容这三件事，不是"清缓存"。

---

## Q22：手动执行 fetchUpdateAsync + reloadAsync 后用户端表现如何？runtimeVersion 匹配如何规避崩溃？

### reloadAsync 的用户端表现

三个动作区分：`checkForUpdateAsync()` 只问有没有新版本；`fetchUpdateAsync()` 下载到本地（用户无感）；`reloadAsync()` **用新 bundle 重启 JS**（用户有明显感知）。

`reloadAsync` **不退出整个 App 进程**，而是**重新加载 JS 层**：

```
调用 reloadAsync()
   ↓
当前界面消失 → 短暂闪一下(白屏/启动图) → App 从头重新加载 → 回到首页
```

用户感受：**画面闪一下、像重开了一次，停在首页（不是原来那个页面）**；会打断当前操作（表单/浏览位置丢失）；重载耗时几百毫秒到 1~2 秒；state 和导航栈全部重置。

**不能无脑静默 reload**，正确做法：

- **弹窗让用户决定**：
```js
await Updates.fetchUpdateAsync()
Alert.alert('更新提示', '有新版本，是否立即重启应用？', [
  { text: '稍后', style: 'cancel' },
  { text: '立即更新', onPress: () => Updates.reloadAsync() },
])
```
- **挑无感时机**：App 从后台切回前台时检查+reload；或干脆只下载不 reload，靠用户下次自然冷启动生效
- **启动 splash 阶段更新**：启动图还没消失时 check+fetch+reload，用户只觉得启动稍久

**一句话**：reloadAsync = JS 层热重启，画面闪一下回首页、丢失当前状态；要么弹窗让用户选，要么挑切前台/启动这类无感时机，不要在用户操作中途静默触发。

### runtimeVersion 匹配的规避方法

**核心原则**：JS bundle 和原生包必须版本对得上，改了原生就升 runtimeVersion，让 EAS Update 只把 bundle 推给 runtimeVersion 一致的 App。

**做法一：policy 自动管理（推荐）**

```jsonc
// app.json
{ "expo": { "runtimeVersion": { "policy": "fingerprint" } } }
```

| policy | 含义 | 适用 |
|---|---|---|
| appVersion | 跟随 App 版本号 | 简单 |
| nativeVersion | 跟随原生版本号 | 类似 |
| **fingerprint** | 自动扫描原生依赖生成指纹，原生一变指纹就变 | ⭐最精准推荐 |

fingerprint 最省心：原生没变指纹不变，JS 可自由热更；一旦加原生库/改原生配置，指纹自动变化，老 App 拉不到新 bundle，自动防错配，不用手动判断"这次算不算动原生"。

**做法二：手动管理**（固定字符串，靠人记，容易漏判，不推荐）

**完整规避流程**

```
只改了 JS？ → runtimeVersion 不变 → eas update 热更新推送(老App安全拉到) ✅
动了原生？ → runtimeVersion 变化(fingerprint自动变/手动升)
           → 必须 eas build 重新打包走商店发版
           → 老App(旧runtimeVersion)不会拉到新bundle，不会崩 ✅
           → 用户装新原生包后runtimeVersion匹配，才开始接收对应bundle
```

**一句话**：用 `runtimeVersion: { policy: "fingerprint" }` 让工具自动追踪原生变化——原生没变时 JS 随便热更，原生一变指纹自动变、老 App 收不到新 bundle 只能走商店更新，新旧配对由 EAS Update 自动隔离，从根上杜绝"新 JS 配旧原生"的崩溃。

---

## Q23：checkUpdate 逻辑的执行时机是什么？是路由切换时吗？"新 JS 配旧原生"崩溃的详细流程是怎样的？

### checkUpdate 执行时机——不是路由切换，是几个固定生命周期节点

**核心原则**：检查更新是「App 级别」的动作，和路由切换（页面导航）无关——路由切换太频繁、太细，不适合挂检查更新逻辑。

**常见挂载时机**

1. **App 启动时**（最常用）：
```jsx
useEffect(() => { checkForUpdates() }, [])   // 只在 App 挂载时跑一次
```

2. **App 从后台切回前台时**：
```jsx
AppState.addEventListener('change', (nextState) => {
  if (nextState === 'active') checkForUpdates()
})
```

3. **定时轮询**（补充，不常作主力）：每 30 分钟等，用于长时间挂前台不退出的兜底。

4. **Expo 自动检查配置**（不用手写）：
```jsonc
{ "expo": { "updates": { "checkAutomatically": "ON_LOAD" } } }
```

**为什么不挂路由切换**：一次 App 生命周期路由可能切几十次，检查更新是网络请求没必要这么频繁；更新检查关心的是"进程级别"该不该用新代码，不是"页面级别"的事；挂路由上会浪费流量电量且与体验无关。

**一句话**：checkUpdate 挂在 App 启动 + 切前台 这两个生命周期节点，和路由切换无关。

### "新 JS 配旧原生"崩溃的完整流程

**场景**：做"人脸识别登录"，需要调用新原生模块 `expo-face-detector`（之前没装过）。

**第一步——老用户手机现状**：用户 A 装的 App，原生层是上次发版时固定的二进制，**没有** `expo-face-detector` 的原生实现；JS 层调用的都是已有模块。原生二进制装进手机后不会再变，除非用户去商店重新下载新版本。

**第二步——开发环境加新原生模块**：
```bash
npx expo install expo-face-detector
npx expo prebuild   # 把新模块的原生代码编译进去
```
新构建的 App 原生层多了这个模块。

**第三步——危险操作**：如果误把这次改动当纯 JS 热更新，直接 `eas update` 推给所有存量用户（包括原生层没有该模块的用户 A）：

```
用户A手机: 加载"新"JS bundle → 执行 expo-face-detector.detectFace()
  → JS桥接层去原生找这个模块 → 原生层"没有,不存在" → 💥 崩溃/白屏
```

"新 JS" = 假设用户已有新原生能力的 JS 代码；"旧原生" = 用户手机里还停留在上次发版、没有新能力的原生二进制；JS 调用原生层根本不存在的东西 → 找不到对应原生方法 → 崩溃。

**第四步——runtimeVersion 如何挡住这件事**：runtimeVersion 给"原生层的能力版本"打标签：

```
用户A的App: 原生层标签="abc123"(对应"没有人脸识别")
新JS bundle: 要求的标签="xyz789"(对应"有人脸识别")
热更新服务器: 只把标签"abc123"的bundle推给标签是"abc123"的用户
  → 用户A标签"abc123" ≠ 新bundle标签"xyz789" → ❌不匹配
  → 服务器根本不会把这个新bundle推给用户A
  → 用户A继续用标签匹配的旧bundle(没有人脸识别，但能正常跑) ✅不崩溃
```

用 `fingerprint` 策略时，标签是自动计算的指纹：prebuild 前 fingerprint="abc123"，装了新模块后 prebuild 后自动变成"xyz789"，不用手动记。

**第五步——用户 A 到底怎样才能用上新功能**：必须走商店更新，把新原生二进制装到手机上：
```
eas build(打包含新原生代码的新版本) → eas submit(上架审核) → 审核通过
→ 用户A去商店更新 → 原生层真的有了expo-face-detector → 标签变成"xyz789"
→ 服务器才把要求"xyz789"的JS bundle推给用户A → 正常工作 ✅
```

**一句话**：checkUpdate 挂在 App 启动和切前台，与路由无关；"新 JS 配旧原生"崩溃的本质是 JS 调用了用户手机原生层里根本不存在的东西（因为该用户还没通过商店更新获得这个新原生能力），而 runtimeVersion（尤其自动化的 fingerprint 策略）给"JS 需要的原生能力版本"和"用户手机实际拥有的版本"分别打标签，只有标签一致服务器才推送，从根源防止错配，把崩溃变成"静默跳过，留给下次真正发版触达"。

---

## Q24：是 runtimeVersion 匹配 bundle，还是 bundle 匹配 runtimeVersion？内部如何匹配？谁来做这个工作？

**方向澄清**：严格说，runtimeVersion 不是"主动匹配"的一方，bundle 也不是——它们都只是**被贴上标签的对象**。真正执行"比对"这个动作的是**服务器**。

更准确的说法：**客户端带着自己的 runtimeVersion 去问服务器，服务器拿这个值去"筛选出"标签相同的 bundle 返回**。即服务端根据 runtimeVersion 去匹配/筛选候选 bundle，而不是二者互相主动匹配。

**标签来源 1——原生 App 内嵌的 runtimeVersion**

```
eas build 打包时
  → 读取 app.json 的 runtimeVersion policy(如 fingerprint)
  → 根据当前原生依赖/配置计算出具体值(如 "a1b2c3")
  → 写死编译进这个原生包(iOS Info.plist/Expo.plist、Android对应资源)
  → 此值固定不变,除非重新 build
```

**标签来源 2——每次发布的 JS bundle 也带 runtimeVersion 标签**

```
eas update 发布时
  → CLI 用同样的 policy 重新计算一次 runtimeVersion
  → 把算出的值作为标签,和这次 JS bundle 一起记录到服务端(manifest 里)
```

两边用**同一套计算规则**（如 fingerprint），原生没变则算出的值一样；原生变了则新旧包算出的值不同。

**匹配的完整请求流程**

```
① App启动,expo-updates模块读取自己内嵌的runtimeVersion(如"a1b2c3")
② 向EAS Update服务器发HTTP请求,带上:
     expo-runtime-version: a1b2c3
     expo-channel-name: production
     expo-platform: ios/android
③ 服务器收到请求:找到该channel下所有发布过的bundle
     → 筛选出 runtimeVersion == "a1b2c3" 的
     → 在筛选结果里挑最新发布的那一个
④ 服务器把匹配上的bundle信息(manifest+下载地址)返回给客户端
⑤ 客户端下载这个bundle
⑥ 客户端下载后本地二次校验:下载回来的bundle标签是否与自己内嵌的一致
     → 一致才允许应用(下次reload/启动生效)；不一致则拒绝,防止服务端配置出错误装
```

**谁执行了这项工作**

| 步骤 | 谁执行 | 做什么 |
|---|---|---|
| 计算并贴标签(build时) | 本地/EAS 构建服务 | 给原生包算 runtimeVersion,写死进去 |
| 计算并贴标签(publish时) | EAS CLI | 给这次 JS bundle 算 runtimeVersion,存到服务端 |
| **真正的筛选/比对** | **EAS Update 服务器** | 拿客户端的 runtimeVersion,在候选 bundle 里找标签相同的 |
| 二次校验(下载后) | 客户端本地(expo-updates模块) | 确认下载回来的 bundle 标签与自己一致,双重保险 |

**一句话**：runtimeVersion 是贴在「原生 App」和「每次发布的 JS bundle」两边的同一种标签（用同一套规则算出）；"匹配"不是它俩自己做的，而是由 EAS Update 服务器在客户端请求更新时，拿客户端携带的 runtimeVersion 去候选 bundle 池里筛选出标签相同的那个返回，客户端下载后再本地二次核对一遍作为兜底。
