# jovijob-oauth-keys

「就位」第三方登录用的 **Google 公钥中转**。这个仓库里只有**公开信息**（Google 本来就公开发布的登录公钥），没有任何密钥，所以可以公开。

## 为什么需要它

「就位」的服务器在北京，连不上 Google；而校验 Google 登录凭证，必须先拿到 Google 的公钥。
GitHub Actions 的机器在境外、连得上 Google，服务器又连得上 `api.github.com`，于是由 Actions 定时抓公钥存在这里，服务器来这里读。

- `google.json`：Google 的公钥集（`https://www.googleapis.com/oauth2/v3/certs` 的内容，排序、固定缩进）
- `.github/workflows/mirror-google-keys.yml`：每 6 小时抓一次；**只有公钥变了才提交**，另外每 7 天至少提交一次 `heartbeat.txt`（GitHub 会停掉 60 天没有活动的公开仓库的定时任务）
- `heartbeat.txt`：心跳，只是为了让仓库保持「有活动」

服务器怎么读、读不到怎么办、信任关系：见主仓库 `docs/第三方登录.md` 第六节。

## 建这个仓库（只做一次）

1. 在 GitHub 新建一个**公开**仓库，名字建议 `jovijob-oauth-keys`（要和服务器配置 `google_jwks_repo` 里写的一致）。
2. 把本目录的全部内容（含隐藏的 `.github` 目录）原样放进去，推到默认分支。
3. 仓库 Settings → Actions → General → Workflow permissions：选 **Read and write permissions**（工作流要往仓库里提交 `google.json`）。
4. Actions 页面手动跑一次 `mirror-google-keys`（Run workflow），确认是绿的。
5. 把 `owner/仓库名`（如 `FFanHao/jovijob-oauth-keys`）配到服务器 `config.local.php` 的 `google_jwks_repo`。

之后不用再管。Actions 失败时 GitHub 会给仓库所有者发邮件。

## 安全（这个仓库里的 google.json 就是「就位」验证 Google 登录的信任根）

谁能改 `google.json`，谁就能伪造任意 Google 用户的登录。所以：

- **GitHub 账号开两步验证（2FA）**，并且只给确实需要的人写权限；
- 工作流里**没有用任何第三方 action**（只用 `git` 和 `curl`），不引入别人的可变标签；
- 打开本仓库的 Watch → Custom → 勾选 Pushes（或在 Settings 里加通知），`google.json` 一变就收到邮件——Google 换钥本来就很少，每一次变化都值得看一眼；
- 工作流本身（`.github/workflows/`）的改动同样要留意。
