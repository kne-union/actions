# kne-union/actions

kne-union 组织的 GitHub Actions 可复用工作流集合，用于统一管理库发布、小程序发布、内容部署等 CI/CD 流程。

## 工作流架构

工作流分为三层，形成清晰的调用链：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        编排层（由外部仓库调用）                        │
├─────────────────────────────────────────────────────────────────────┤
│  publish-libs-workflow              库 + 示例 + 静态数据 + UC同步 + Release  │
│  publish-libs-no-example-workflow   库 + 静态数据 + Release                    │
│  publish-remote-components-workflow 构建 + GitHub Pages + npm + UC同步         │
│  publish-remote-project-workflow    构建 + npm + 静态数据 + UC同步 + Release   │
│  publish-miniprogram-libs-workflow  构建 + 小程序发布 + npm + 二维码 + Release │
│  publish-node-workflow              检出 + npm + 静态数据 + Release            │
│  publish-content-workflow           内容生成 + 静态部署 + 可选npm              │
│  complete-issue-workflow            PR合并后关闭Issue + 删除分支               │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │ 调用
┌──────────────────────────────────▼──────────────────────────────────┐
│                          中间层                                     │
├─────────────────────────────────────────────────────────────────────┤
│  build-and-publish-lib-workflow     构建库 + npm发布                 │
│  build-and-publish-example-workflow 构建示例 + GitHub Pages + npm发布│
└──────────────────────────────────┬──────────────────────────────────┘
                                   │ 调用
┌──────────────────────────────────▼──────────────────────────────────┐
│                          基础层                                     │
├─────────────────────────────────────────────────────────────────────┤
│  publish-npm-workflow               npm发布 + cnpm同步              │
│  npm-release-workflow               GitHub Release创建              │
│  publish-miniprogram-workflow       微信小程序上传                   │
│  publish-miniprogram-qrcode         小程序二维码生成 + 静态部署      │
└─────────────────────────────────────────────────────────────────────┘
```

## 工作流详细说明

### 编排层

#### `publish-libs-workflow`

完整的库发布流程：构建库 → 发布到 npm → 构建示例 → 部署到 GitHub Pages → 同步静态数据 → 同步 UC → 创建 Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |
| `example_package_name` | string | 是 | 示例项目的 npm 包名 |

#### `publish-libs-no-example-workflow`

简化版库发布流程（无示例）：构建库 → 发布到 npm → 同步静态数据 → 创建 Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |

#### `publish-remote-components-workflow`

远程组件发布：构建 → 部署 GitHub Pages → 发布 npm → 同步静态数据 → 同步 UC → 创建 Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |

#### `publish-remote-project-workflow`

远程项目发布：构建 → 发布 npm → 同步静态数据 → 同步 UC → 创建 Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |

#### `publish-miniprogram-libs-workflow`

小程序库发布：构建 → 发布小程序 → 发布 npm → 同步静态数据 → 生成二维码 → 创建 Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |
| `APP_ID` | string | 是 | 微信小程序 AppID |
| `PRIVATE_KEY_NAME` | string | 是 | 小程序上传私钥的 Secret 名称 |
| `TOKEN_SECRET_NAME` | string | 是 | 小程序 Token 的 Secret 名称 |

#### `publish-node-workflow`

Node 项目发布：检出代码 → 发布 npm → 同步静态数据 → 创建 Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |

#### `publish-content-workflow`

内容发布：使用 `@kne/blog-maker` 生成内容 → 部署静态数据 → 可选发布到 npm。

| 输入参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `publish-to-npm` | boolean | 否 | `false` | 是否同时发布到 npm |

#### `complete-issue-workflow`

PR 合并后自动处理：从分支名（格式 `issue-{数字}`）提取 Issue 编号 → 删除源分支 → 关闭 Issue 并评论。

无输入参数。

### 中间层

#### `build-and-publish-lib-workflow`

构建库并发布到 npm。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名 |

#### `build-and-publish-example-workflow`

构建示例项目、部署到 GitHub Pages 并发布到 npm。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | 示例项目 npm 包名 |

### 基础层

#### `publish-npm-workflow`

发布到 npm 并同步 cnpm。

| 输入参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `package_name` | string | 是 | - | npm 包名 |
| `artifact` | string | 否 | `build-dist` | 要下载的 artifact 名称 |

#### `npm-release-workflow`

创建 GitHub Release。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `package_name` | string | 是 | npm 包名（用于获取版本号） |

#### `publish-miniprogram-workflow`

上传微信小程序代码。

| 输入参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `artifact` | string | 否 | `build-dist` | 要下载的 artifact 名称 |
| `PROJECT_DIR` | string | 否 | `./` | 小程序项目目录 |
| `APP_ID` | string | 是 | - | 微信小程序 AppID |
| `PRIVATE_KEY_NAME` | string | 是 | - | 小程序上传私钥的 Secret 名称 |

#### `publish-miniprogram-qrcode`

生成小程序二维码并部署。

| 输入参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `APP_ID` | string | 是 | 微信小程序 AppID |
| `TOKEN_SECRET_NAME` | string | 是 | 小程序 Token 的 Secret 名称 |

## 使用方式

在其他仓库的 workflow 中通过 `uses` 引用可复用工作流：

```yaml
jobs:
  publish:
    uses: kne-union/actions/.github/workflows/publish-libs-workflow.yml@master
    secrets: inherit
    with:
      package_name: '@kne/your-package'
      example_package_name: '@kne/your-package-example'
```

## 所需 Secrets

| Secret | 说明 | 使用的工作流 |
|---|---|---|
| `KNE_PACKAGE_PUBLISH` | npm 发布 Token | `publish-npm-workflow` |
| `ADMIN_TOKEN` | 静态数据部署 Token | 涉及 manifest 同步的多个工作流 |
| `SYNC_WEB_HOOK` | UC 同步 Webhook URL | `publish-libs-workflow`, `publish-remote-project-workflow` |
| `{PRIVATE_KEY_NAME}` | 小程序上传私钥（动态命名） | `publish-miniprogram-workflow` |
| `{TOKEN_SECRET_NAME}` | 小程序 Token（动态命名） | `publish-miniprogram-qrcode` |
| `GITHUB_TOKEN` | GitHub 自动提供 | GitHub Pages 部署 |

## 依赖的 CLI 工具

| 工具 | 说明 |
|---|---|
| `@kne/npm-tools` | 版本号管理（latestVersion / nextPatchVersion） |
| `@kne/blog-maker` | 内容/manifest 生成 |
| `@kne/miniprogram-tools` | 微信小程序上传与二维码生成 |
| `cnpm` | cnpm 源同步 |

## 依赖的外部 Actions

| Action | 版本 | 说明 |
|---|---|---|
| `actions/checkout` | v6 | 代码检出 |
| `actions/setup-node` | v6 | Node.js 环境配置 |
| `actions/upload-artifact` | v6 | 构建产物上传 |
| `actions/download-artifact` | v7 | 构建产物下载 |
| `peaceiris/actions-gh-pages` | v4 | GitHub Pages 部署 |
| `peter-evans/close-issue` | v3 | 关闭 GitHub Issue |
| `ncipollo/release-action` | v1 | 创建 GitHub Release |
| `kne-union/actions-manifest` | master | Manifest 同步 |
| `kne-union/actions-deploy-static-data` | master | 静态数据部署 |
