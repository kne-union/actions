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
│  publish-node-workflow              可选测试 + 检出 + npm + 静态数据 + Release │
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
│  node-test                          Node 多版本 npm test             │
│  publish-npm-workflow               npm发布 + cnpm同步 + 开发者文档同步  │
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

远程组件发布：构建 → 部署 GitHub Pages → 发布 npm（含 `README.md`）→ 同步静态数据 → 同步 UC → 创建 Release。

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

Node 项目发布：可选测试 → 检出代码 → 发布 npm → 同步静态数据 → 创建 Release。

| 输入参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `package_name` | string | 是 | - | npm 包名 |
| `run_test` | boolean | 否 | `false` | 是否在发布前真正执行测试（默认关闭；关闭时 `node-test` 仍返回 success，下游正常发布） |
| `node-versions` | string | 否 | `'["18","20","22"]'` | 传给 `node-test` 的版本矩阵 |
| `install-command` | string | 否 | `npm install` | 传给 `node-test`（无 lock 勿用 `npm ci`） |
| `test-command` | string | 否 | `npm test` | 传给 `node-test` |

开启测试的调用示例：

```yaml
jobs:
  node-npm:
    uses: kne-union/actions/.github/workflows/publish-node-workflow.yml@master
    secrets: inherit
    with:
      package_name: '@kne/your-package'
      run_test: true
```

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

#### `node-test`

Node 项目测试：checkout → setup-node（版本矩阵）→ 安装依赖 → 跑测试。供各业务仓库以 `uses` 引用，无需在业务仓写完整测试步骤。

| 输入参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `run_test` | boolean | 否 | `true` | 为 `false` 时跑空成功 job（不执行测试），供 `publish-node` 跳过测试且不触发 skip 传染 |
| `node-versions` | string | 否 | `'["18","20","22"]'` | JSON 数组，Node 版本矩阵 |
| `install-command` | string | 否 | `npm install` | 依赖安装命令（默认不要求 lock；有 lock 可传 `npm ci`） |
| `test-command` | string | 否 | `npm test` | 测试命令 |

业务仓库引用示例：

```yaml
name: CI
on:
  push:
    branches: [master, main]
  pull_request:

jobs:
  test:
    uses: kne-union/actions/.github/workflows/node-test.yml@master
```

#### `publish-npm-workflow`

发布到 npm，同步 cnpm，并在配置了 Open API 密钥时触发 [developer-document](https://develop.leapin-ai.com/) 同步。

| 输入参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `package_name` | string | 是 | - | npm 包名 |
| `artifact` | string | 否 | `build-dist` | 要下载的 artifact 名称 |
| `developer_document_sync_type` | string | 否 | `npm-package` | `npm-package`：普通库，调 `POST /open-api/sync/npm-package`；`remote-component`：远程组件/远程项目，调 `POST /open-api/sync/remote-component`（`remote` 由 `packageName` 去 scope 推导，如 `@kne-components/components-core` → `components-core`） |

**Developer Document 同步（可选）**：在组织/仓库 Secrets 中配置 `DEVELOPER_DOCUMENT_OPENAPI_APP_ID` 与 `DEVELOPER_DOCUMENT_OPENAPI_APP_SECRET`（在 develop.leapin-ai.com 管理后台「密钥管理」创建）后自动启用；未配置则跳过。签名由 `npx @kne/npm-tools generateOpenApiSignature` 生成。API 根地址默认 `https://develop.leapin-ai.com/api/v1`，可通过仓库 Variable `DEVELOPER_DOCUMENT_SYNC_URL` 或 Secret `DEVELOPER_DOCUMENT_SYNC_URL` 覆盖；签名有效期秒数可通过 Variable `DEVELOPER_DOCUMENT_OPENAPI_EXPIRE_SECONDS` 调整（默认 `300`）。

远程组件编排（`publish-remote-components-workflow`、`publish-remote-project-workflow`）应传 `developer_document_sync_type: remote-component`。

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
| `DEVELOPER_DOCUMENT_OPENAPI_APP_ID` | developer-document Open API App ID | `publish-npm-workflow`（可选，与 App Secret 同时配置才触发同步） |
| `DEVELOPER_DOCUMENT_OPENAPI_APP_SECRET` | developer-document Open API App Secret | `publish-npm-workflow`（可选） |
| `DEVELOPER_DOCUMENT_SYNC_URL` | developer-document API 根地址（如 `https://develop.leapin-ai.com/api/v1`） | `publish-npm-workflow`（可选 Secret；也可用同名 Repository Variable） |
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
