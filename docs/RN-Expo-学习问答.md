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
