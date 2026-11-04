# 从 develop 手动发布

将代码合并到 `develop` 后，在 GitHub 手动运行一次 `Release from develop`。
工作流读取根目录 `pubspec.yaml` 的版本号，自动创建 tag，然后发布根包到 pub.dev
并创建 GitHub Release。合并代码、普通 push 和推送 tag 都不会自动发布。

## 一次性配置

1. 将 `release.yml` 和 `publish.yml`（均位于 `.github/workflows/`）加入 GitHub
   默认分支，使手动触发可用；`develop` 也必须包含这两个工作流。
2. 在 pub.dev 的 `kline_flutter` 管理页面启用 GitHub Actions 自动发布：
   repository 填 `911hzh/kline_flutter`，tag pattern 填 `v{{version}}`。
   不启用 Require GitHub Actions environment（当前工作流未配置 environment）。
   使用 OIDC，无需配置 Secrets 或 PAT。若包尚未发布过，需先手动发布首个版本。
3. 仓库需允许 GitHub Actions 运行。工作流已声明创建 tag、Release 和触发工作流
   所需的 GITHUB_TOKEN 权限；组织策略也需允许这些权限。

## 每次发布

1. 在待合并代码中更新根目录 `pubspec.yaml` 的 `version` 及 `CHANGELOG.md`，
   合并到 `develop`。使用尚未发布过的版本号，例如 `0.5.2`。
2. 打开 GitHub → Actions → **Release from develop** → **Run workflow**。
3. 将 **Use workflow from** 选择为 **develop**，点击 **Run workflow**。
4. 第一阶段自动为触发时的 develop 提交创建 `v0.5.2` tag；随后在 Actions 中
   查看自动启动的 **Publish tagged package**，等待分析、测试、发布预检、
   pub.dev 发布及 GitHub Release 创建完成。第一阶段成功仅表示已启动发布。

也可以使用已登录的 GitHub CLI：

```sh
gh workflow run release.yml --repo 911hzh/kline_flutter --ref develop
```

无需手动创建或推送 tag。内部第二阶段使用 tag 上下文，以满足 pub.dev OIDC
授权要求；两个工作流都仅使用 `workflow_dispatch`，没有 tag push 触发器。

若第一阶段失败后重试，只复用指向同一提交的 tag，绝不移动已有 tag。
若发布前检查失败，修复后请更新版本号再发布（旧 tag 仍指向原提交）。
若 pub.dev 发布成功但 GitHub Release 创建失败，请为同一 tag 手动补建 Release，
不要重跑发布，pub.dev 不允许重复发布同一版本。

参考：[Dart 自动发布文档](https://dart.dev/tools/pub/automated-publishing)、
[手动 tag 触发支持说明](https://github.com/dart-lang/pub-dev/issues/8507)。
