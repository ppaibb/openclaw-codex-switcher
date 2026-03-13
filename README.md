# openclaw-codex-switcher

一个给 **OpenClaw** 用的本地 **Codex 多账号切换工具**。

它的核心思路不是把一堆账号塞进 `openclaw.json` 做“花名册”，而是：

- 把每个 Codex 账号保存成一个独立 snapshot
- 统一把当前激活槽位保持为 `openai-codex:default`
- 切换时只注入当前账号凭据
- 通过 snapshot 管理多账号、配额和续期

这样做的好处是：

- 逻辑简单
- 容易 debug
- 不容易污染 OpenClaw 其它配置
- 更适合一台机器上轮换多个 Codex 账号

---

## 项目、Skill、命令的关系

这三个名字不是一回事：

- **GitHub 项目名**：`openclaw-codex-switcher`
- **ClawHub Skill 名**：`codex-switcher`
- **本地命令名**：`cs`

也就是说：
- GitHub 上这是一个项目
- ClawHub 上这是一个 skill
- 真正在机器上使用时，命令还是 `cs`

---

## 主要功能

- `cs add`：新增 Codex 账号
- `cs switch`：切换账号
- `cs list`：查看所有快照、对应邮箱和剩余有效期
- `cs current`：查看当前激活账号
- `cs quota`：查看当前账号配额
- `cs refresh`：刷新单个账号快照
- `cs refresh-all`：自动刷新临近过期的账号

---

## 设计目标

这个项目最开始是为了替代高敏感、黑盒、第三方 relogin 脚本。

目标不是做一个花哨的账号管理器，而是做一个：

- 本地
- 可审计
- 可理解
- 可控
- 不乱改配置

的 OpenClaw Codex 多账号工具。

---

## 工作原理

活跃凭据文件：

- `~/.openclaw/agents/main/agent/auth-profiles.json`

快照目录：

- `~/.openclaw/auth-snapshots/`

每个账号只保存最小必要字段：

- `access`
- `refresh`
- `expires`
- `accountId`

切换账号时，工具只更新 `openai-codex:default` 的凭据，不碰无关配置。

---

## 依赖

运行前置：

- Python 3
- `requests`

安装依赖：

```bash
pip install requests
```

---

## 安装方式

### 方式一：直接作为本地工具使用

将脚本软链到系统路径：

```bash
ln -s /path/to/openclaw-codex-switcher/skills/codex-switcher/scripts/cs.sh /usr/local/bin/cs
```

然后就可以直接使用：

```bash
cs list
cs current
cs quota
```

### 方式二：作为 Skill 使用

本项目也提供 OpenClaw Skill：

- Skill 名：`codex-switcher`
- 打包文件：`codex-switcher.skill`

如果你使用 ClawHub 上传目录，请上传：

```text
skills/codex-switcher
```

不是整个仓库，也不是别的目录层级。

---

## 常用命令

### 查看快照列表

```bash
cs list
```

### 查看当前账号

```bash
cs current
```

### 查看当前账号配额

```bash
cs quota
```

### 切换账号

```bash
cs switch gpt01-gua.cx
```

### 新增账号（自动生成 alias）

```bash
cs add
```

### 新增账号（手动指定 alias）

```bash
cs add my-alias
```

登录完成后：

```bash
cs add --apply '<callback-url>'
```

或：

```bash
cs add --apply '<callback-url>' my-alias
```

### 刷新单个快照

```bash
cs refresh gpt01-gua.cx
```

### 自动刷新临近过期的快照

```bash
cs refresh-all
```

---

## 推荐 cron

每天跑一次或两次就够了：

```bash
0 3 * * * /usr/local/bin/cs refresh-all
```

---

## 安全说明

这是本地高敏感认证工具，请妥善保护：

- `~/.openclaw/auth-snapshots/`
- `~/.openclaw/agents/main/agent/auth-profiles.json`

建议：

- 不要把快照目录暴露给不可信用户
- 不要把 token / refresh token 输出到日志或聊天里
- 不要盲目执行第三方 `curl | bash` relogin 脚本

---

## 适合谁

适合这类用户：

- 一台 OpenClaw 宿主机上要轮换多个 Codex 账号
- 想看当前账号剩余配额
- 想定时自动续期账号快照
- 不想依赖第三方黑盒 relogin 脚本

---

## 发布渠道

- GitHub：源码、Issue、版本迭代
- ClawHub：Skill 发布

源码仓库：

- `https://github.com/ppaibb/openclaw-codex-switcher`

---

## 当前状态

当前版本已经验证通过这些链路：

- `cs add`
- `cs list`
- `cs switch`
- `cs current`
- `cs quota`
- `cs refresh`
- `cs refresh-all`

适合继续打磨并发布。
