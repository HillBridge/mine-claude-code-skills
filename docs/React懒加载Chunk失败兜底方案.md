# React 懒加载 Chunk 失败兜底方案

> 用于解决 SPA 部署后,用户停留在旧页面并触发懒加载路由时,旧 chunk 加载失败导致白屏的问题。

---

## 1. 问题场景

React 项目通常会用路由懒加载:

```tsx
const AccountPage = React.lazy(() => import('@/pages/account'))
```

用户打开页面后,浏览器里运行的是当前版本的 JS。此时如果前端重新部署,旧页面不会自动切换到新版本。

当用户继续在旧页面里点击某个懒加载路由时,旧运行时代码可能会请求旧资源:

```txt
/assets/account-page.oldhash.js
```

如果这个旧 chunk:

- 浏览器本地没有缓存
- CDN 边缘节点没有缓存
- 源站部署时已经删除

那么请求会返回 404,React 懒加载会抛出加载错误,常见表现是 `ChunkLoadError` 或动态 import 失败,页面可能白屏。

---

## 2. 前端兜底目标

前端无法让一个已经被删除的旧 chunk 重新存在,但可以做到:

1. 识别 chunk 加载失败
2. 自动刷新一次页面
3. 刷新后重新校验 `index.html`,拿到新的资源地图
4. 如果刷新后仍失败,展示可恢复错误页,避免白屏
5. 上报错误,方便确认是否是部署资源断裂

一句话:

> 前端兜底不是修复旧资源,而是在发现旧资源加载失败后,让用户重新进入新版本。

---

## 3. 推荐方案

推荐组合:

| 层级 | 做法 | 作用 |
| --- | --- | --- |
| 懒加载 retry | 动态 import 失败后重试 1 次 | 处理临时网络抖动 |
| Error Boundary | 包住路由出口 | 捕获懒加载渲染阶段错误 |
| 自动刷新一次 | 识别 chunk 错误后 `window.location.reload()` | 重新获取最新入口和资源 |
| 防死循环标记 | 使用 `sessionStorage` 记录已刷新 | 避免无限刷新 |
| 兜底页面 | 第二次仍失败时显示刷新按钮 | 避免白屏 |
| 错误上报 | 记录路由、版本、错误信息 | 定位部署问题 |

---

## 4. 识别 Chunk 加载错误

不同构建工具和浏览器的错误信息不完全一致,建议按多种关键词判断。

```ts
const CHUNK_ERROR_KEY = 'app_chunk_reload_once'

export function isChunkLoadError(error: unknown) {
  const message = String(error instanceof Error ? error.message : error)
  const name = error instanceof Error ? error.name : ''

  return (
    name.includes('ChunkLoadError') ||
    message.includes('ChunkLoadError') ||
    message.includes('Loading chunk') ||
    message.includes('Failed to fetch dynamically imported module') ||
    message.includes('Importing a module script failed')
  )
}
```

常见错误包括:

| 环境 | 可能的错误 |
| --- | --- |
| Webpack | `ChunkLoadError` / `Loading chunk xxx failed` |
| Vite / 原生 ESM | `Failed to fetch dynamically imported module` |
| Safari | `Importing a module script failed` |

---

## 5. 自动刷新一次

刷新前要写入 `sessionStorage` 标记,防止资源一直异常时反复刷新。

```ts
const CHUNK_ERROR_KEY = 'app_chunk_reload_once'

export function handleChunkError(error: unknown) {
  if (!isChunkLoadError(error)) {
    return false
  }

  const hasReloaded = sessionStorage.getItem(CHUNK_ERROR_KEY)

  if (!hasReloaded) {
    sessionStorage.setItem(CHUNK_ERROR_KEY, '1')
    window.location.reload()
    return true
  }

  return false
}
```

执行效果:

1. 第一次 chunk 加载失败
2. 识别为部署资源问题或动态 import 加载问题
3. 自动刷新当前页面
4. 浏览器重新请求或校验 `index.html`
5. 如果 `index.html` 缓存策略正确,会拿到新版本资源引用

---

## 6. Error Boundary 包住路由区域

React 懒加载失败通常会冒泡到 Error Boundary。建议在应用根部或路由出口外层包一层专门的边界。

```tsx
import React from 'react'

const CHUNK_ERROR_KEY = 'app_chunk_reload_once'

function isChunkLoadError(error: unknown) {
  const message = String(error instanceof Error ? error.message : error)
  const name = error instanceof Error ? error.name : ''

  return (
    name.includes('ChunkLoadError') ||
    message.includes('ChunkLoadError') ||
    message.includes('Loading chunk') ||
    message.includes('Failed to fetch dynamically imported module') ||
    message.includes('Importing a module script failed')
  )
}

export class ChunkErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false }

  static getDerivedStateFromError() {
    return { hasError: true }
  }

  componentDidCatch(error: unknown) {
    if (!isChunkLoadError(error)) {
      return
    }

    const hasReloaded = sessionStorage.getItem(CHUNK_ERROR_KEY)

    if (!hasReloaded) {
      sessionStorage.setItem(CHUNK_ERROR_KEY, '1')
      window.location.reload()
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <p>页面资源已更新,请刷新后继续使用。</p>
          <button onClick={() => window.location.reload()}>刷新</button>
        </div>
      )
    }

    return this.props.children
  }
}
```

使用方式:

```tsx
<ChunkErrorBoundary>
  <React.Suspense fallback={<Loading />}>
    <RouterProvider router={router} />
  </React.Suspense>
</ChunkErrorBoundary>
```

建议包在路由内容区域或应用根路由外层,不要只包很深的页面组件,否则有些路由懒加载错误可能捕获不到。

---

## 7. 懒加载 Retry

如果只是短暂网络波动,可以先重试一次动态 import。

```tsx
import React from 'react'

export function lazyWithRetry<T extends React.ComponentType<any>>(
  importer: () => Promise<{ default: T }>
) {
  return React.lazy(async () => {
    try {
      return await importer()
    } catch (error) {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return importer()
    }
  })
}
```

使用:

```tsx
const AccountPage = lazyWithRetry(() => import('@/pages/account'))
```

注意:

- retry 能缓解临时网络失败
- 如果旧 chunk 已经被删除,retry 仍然会失败
- 最终仍需要 Error Boundary + 自动刷新兜底

---

## 8. 全局未处理 Promise 兜底

部分动态 import 失败可能表现为未处理的 Promise rejection。可以加全局监听作为补充。

```ts
window.addEventListener('unhandledrejection', (event) => {
  if (handleChunkError(event.reason)) {
    event.preventDefault()
  }
})
```

这层不能替代 Error Boundary,但可以覆盖一部分没有进入 React 渲染边界的异步错误。

---

## 9. 错误上报建议

建议在识别到 chunk 加载失败时上报:

| 字段 | 说明 |
| --- | --- |
| `error.name` | 错误名称 |
| `error.message` | 错误信息 |
| `location.href` | 当前页面 |
| `appVersion` | 当前应用版本 |
| `buildTime` | 构建时间 |
| `userAgent` | 浏览器信息 |
| `hasReloaded` | 是否已经自动刷新过 |

示例:

```ts
function reportChunkError(error: unknown, hasReloaded: boolean) {
  // 接入 Sentry、Datadog、自研日志平台等
  console.error('[chunk-load-error]', {
    error,
    href: window.location.href,
    hasReloaded,
  })
}
```

有了上报后,可以区分:

- 是否集中发生在发版后几分钟
- 是否集中在某些 CDN 节点或区域
- 是否集中在某个懒加载路由
- 是否是旧资源被删除导致

---

## 10. 和缓存部署策略的关系

前端兜底只能降低白屏影响,不能替代正确的缓存和部署策略。

完整方案应同时满足:

```txt
index.html:
  Cache-Control: no-cache
  或 Cache-Control: max-age=0, must-revalidate

assets/*.hash.js/css:
  Cache-Control: public, max-age=31536000, immutable

部署:
  新 hash 资源先上传
  确认 CDN 可访问
  最后切换 index.html
  旧 hash 资源保留最近 3-5 个版本或保留一段时间

前端:
  捕获 ChunkLoadError
  自动刷新一次
  第二次失败展示错误页
```

如果 `index.html` 被长缓存,自动刷新后仍可能拿到旧入口,继续请求旧资源,从而刷新也无法恢复。因此前端兜底依赖入口缓存策略正确。

如果旧 hash 资源部署时被立即删除,用户停留在旧页面触发懒加载时仍会先失败一次。因此还需要旧资源只增不删来减少失败概率。

---

## 11. 最终结论

React 前端兜底推荐:

1. 路由懒加载使用 `lazyWithRetry`
2. 路由出口外层包 `ChunkErrorBoundary`
3. 识别 `ChunkLoadError` / 动态 import 失败
4. 第一次失败自动刷新页面
5. 使用 `sessionStorage` 防止无限刷新
6. 第二次仍失败展示刷新按钮或错误页
7. 同时上报错误

一句话:

> React 侧要做的是捕获懒加载资源断裂,自动刷新一次拿新入口;真正消灭问题还要配合 `index.html` 不长缓存和旧 hash 资源保留。
