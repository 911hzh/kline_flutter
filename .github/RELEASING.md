# 从 develop 手动发布

仅有一个发布工作流：Release from develop。合并、普通 push 和 tag push 都不发布。
从 develop 手动运行后，同一次运行完成分析、测试、发布预检、创建 tag、发布
pub.dev 和创建 GitHub Release。版本号读取根目录 pubspec.yaml，不自动递增。

## 一次性配置

pub.dev 当前拒绝 workflow_dispatch 事件的 GitHub OIDC 发布。单次手动运行
使用 Google 服务账号签发的 ID token，不能沿用 GitHub Actions tag 授权。

1. 在 Google Cloud 项目中创建专用于发布的服务账号。
2. 在 pub.dev 包管理页面的 Automated publishing 中启用 Google Cloud
   服务账号发布，填写该账号的完整邮箱。
3. 为该账号创建 JSON key；按 google-github-actions/auth 的要求，为账号自身
   授予 Service Account Token Creator（roles/iam.serviceAccountTokenCreator），
   并启用 IAM Service Account Credentials API。
4. 在 GitHub 仓库 Settings → Secrets and variables → Actions 中添加
   Repository secret：PUB_DEV_SERVICE_ACCOUNT_KEY，内容为完整 JSON key。
   密钥不要提交到仓库或粘贴到聊天中；此密钥具备该包发布能力，应定期轮换。
5. 将 release.yml 合并到默认分支及 develop；仓库策略需允许
   actions/checkout、subosito/flutter-action、google-github-actions/auth，
   并允许工作流声明的 contents: write 权限。

## 每次发布

1. 更新 pubspec.yaml 的 version 和 CHANGELOG.md，合并到 develop。
2. Actions → Release from develop → Run workflow，分支选择 develop。
3. 等待该运行全部成功。无需手动创建 tag，运行期间 develop 新增提交不会影响本次发布。

如果 v<version> 已存在且指向不同提交，流程会停止，绝不移动旧 tag。
此前失败的 v0.5.2 仍指向旧提交，合并修复后应使用新版本（例如 0.5.3）。
如果 pub.dev 已发布但 GitHub Release 创建失败，为同一个 tag 手动补建 Release，
不要再次发布相同版本。

参考：
- https://dart.dev/tools/pub/automated-publishing#publish-from-anywhere-using-a-service-account
- https://github.com/google-github-actions/auth#service-account-key-json
