# glados-checkin

GLaDOS（glados.cloud / glados.rocks / glados.network）每日自动签到脚本，使用 GitHub Actions 实现。

## 2026-10-20 签到权限修复

如 GitHub Actions 日志显示 `code=-2 | message=没有权限`，说明签到请求被 GLaDOS 服务器拒绝，GitHub Action 本身能运行。

GLaDOS 自 2026 年 9 月升级了 Cookie 会话；旧版 `gld:sess` 可能无法调用新版 API。请：

1. 在登录 GLaDOS 的浏览器打开 https://glados.cloud/console/checkin，退出并重新登录。
2. 按 F12 → Network（网络）→ 刷新，找到 `/api/user/status` 请求，在 Request Headers 中复制**完整的 Cookie 值**（应包含 `gld:sess=` 和 `gld:sess.sig=`）。
3. 进入本仓库 **Settings → Secrets and variables → Actions**，修改已有的 `GLADOS` Secret，粘贴最新完整 Cookie。不要把 Cookie 放到 Issues、提交、聊天或日志中。
4. 推荐额外设置 `GLADOS_USER_AGENT` Secret：在刚才登录的浏览器按 F12 → Console 输入 `navigator.userAgent`，复制输出作为 Secret 内容，以匹配新版设备校验。
5. 打开 **Actions → GLaDOS Daily Checkin → Run workflow** 手动验证，检查日志是否显示 `Checkin! Got` 或当天已签到。

本仓库已支持新版 Cookie 透传及可选 User-Agent 配置；**脚本无法自动生成账号的 Cookie，需在浏览器登录获取**。

---

---
**注意⚠️**(2026.4.20)：
- 2026年4月20日收到glados的邮件，原账号已经转移到新的平台[Railgun](https://railgun.info)，继承了之前账号的计划和剩余天数，但没有继承之前的点数
- 如需要继续自动签到，请移步到新的仓库[railgun_checkin](https://github.com/Zirui8/railgun_checkin)

**V2026.2.4**(2026.2.4)：
- 支持**多个账号**同时签到
- 失败会让 Actions 失败并发送绑定GitHub账号的邮箱提醒，重复签到视为正常，不触发失败邮件

**参考项目**：本仓库参考/部分 fork 自 [actions-integration/checkin](https://github.com/actions-integration/checkin)，本仓库改为纯 Python 实现。

---

## 使用说明

### 1.0 更新 Cookie
GLaDOS 签到脚本使用 **Cookie** 进行登录。Cookie 和 网址Api 可能需要定期更新，否则可能会签到失败。  

更新方法：  
1. 登录 [GLaDOS 官网](https://glados.cloud/)，进入[签到面板](https://glados.cloud/console/checkin) 
2. 按 `F12` 打开浏览器开发者工具 → 切到 **网络(Network)**
3. 刷新一下网页或点击一次“签到”按钮，浏览器开发者工具**Name**(名称)中会出现`checkin`字样，点击`checkin`
4. 在 **Headers** 中下滑找到 **Cookies** ，找到以下字段并完整复制：

```txt
gld:sess=xxx; gld:sess.sig=xxx
```

---

**多账号**：每个账号重复以上步骤，分别获取不同的 Cookie 字符串。

### 1.1 设置到 GitHub Secrets

1. 打开你的 GitHub 仓库 → **Settings** → **Secrets and variables** → **Actions**
2. 点击 **New repository secret**
3. **Name**(名称)填：`GLADOS`（必须全大写）
4. **Secret**(密钥)填入所有账号的 Cookie，多账号用换行分隔
5. **单账号示例**：

```txt
gld:sess=xxx; gld:sess.sig=xxx
```

6. **多账号示例**：

```txt
gld:sess=账号1; gld:sess.sig=账号1;
gld:sess=账号2; gld:sess.sig=账号2;
gld:sess=账号3; gld:sess.sig=账号3;
```


---

### 2. 查看签到日志
1. 打开仓库首页 → 点击 **Actions** 标签页  
2. 在左侧选择 `GLaDOS Checkin` workflow（第一次可以运行检测，显示“✅”表示成功）
3. 点击最新一次运行记录（按日期排序）  
4. 点击 **Run checkin script** 步骤，可以查看所有账号的输出日志  
   - 如果签到成功，会显示类似 `Checkin! Got X Points` 的提示  
   - 如果返回 `"Checkin Repeats! Please Try Tomorrow"` 表示今天已经签过到  
   - 如果是 Cookie 过期或其他错误，会有对应的报错信息，并会发送报错信息至绑定GitHub账号的邮箱

---

### 3. 修改签到时间
当前自动签到时间为 **每天北京时间 00:37**（UTC 时间 16:37）。  

如果需要调整时间，可以修改 `.github/workflows/xxx.yml` 中的：
```yaml
schedule:
  - cron: '37 16 * * *'  # UTC 时间 16:30
```
**注意：GitHub Actions 使用 UTC 时间，需要自己换算成北京时间（+8 小时）。**
