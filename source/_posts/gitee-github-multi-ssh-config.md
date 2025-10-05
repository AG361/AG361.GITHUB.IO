---
title:  'gitee + github 多平台多用户 SSH 密钥配置教程'
date: 2022-11-07
tags:
	- git
	- ssh
---

![](../images/e7e31f4d-d268-4520-a51f-c040ae05298a.jpg)

> 适用于 GitHub / Gitee / 其他平台多账户共存场景。  
> 本文将教你如何为每个平台、每个用户独立生成 SSH 密钥，并通过配置文件实现自动识别。

---

## 一、生成单个 SSH 密钥

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/user_id_rsa
```

### 参数说明：

| 参数 | 含义 |
|------|------|
| `-t ed25519` | 使用 **Ed25519** 算法生成密钥（比 RSA 更快更安全） |
| `-C "your_email@example.com"` | 添加注释，通常为邮箱或用途标识 |
| `-f ~/.ssh/user_id_rsa` | 指定密钥保存路径与文件名（自动生成 `.pub` 公钥文件） |

生成结果：

- 私钥：`~/.ssh/user_id_rsa`  
- 公钥：`~/.ssh/user_id_rsa.pub`

👉 该方式已满足单平台单用户的使用需求。

---

## 二、多平台多用户密钥管理

当你在多个平台（如 GitHub、Gitee）或多个账户之间切换时，建议将密钥按文件夹分类管理。

### 示例目录结构

```bash
~/.ssh/
├── github/
│   ├── user1_id_rsa
│   └── user2_id_rsa
├── gitee/
│   ├── user1_id_rsa
│   └── user2_id_rsa
└── other/
    ├── user1_id_rsa
    └── user2_id_rsa
```

### 密钥生成命令示例

#### GitHub

```bash
ssh-keygen -t ed25519 -C "user1@example.com" -f ~/.ssh/github/user1_id_rsa
ssh-keygen -t ed25519 -C "user2@example.com" -f ~/.ssh/github/user2_id_rsa
```

#### Gitee

```bash
ssh-keygen -t ed25519 -C "user1@example.com" -f ~/.ssh/gitee/user1_id_rsa
ssh-keygen -t ed25519 -C "user2@example.com" -f ~/.ssh/gitee/user2_id_rsa
```

#### Other 平台

```bash
ssh-keygen -t ed25519 -C "user1@example.com" -f ~/.ssh/other/user1_id_rsa
ssh-keygen -t ed25519 -C "user2@example.com" -f ~/.ssh/other/user2_id_rsa
```

---

## 三、配置多个账户（使用 SSH Config）

### 1️⃣ 创建配置文件

```bash
cd ~/.ssh
touch config
```

### 2️⃣ 编辑 `config` 文件

```bash
# =========================
# [GitHub]
# =========================

# 默认 GitHub 账号
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github/user1_id_rsa

# 第二个 GitHub 账号（your_name 可自定义）
Host your_name.github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github/user2_id_rsa


# =========================
# [Gitee]
# =========================

# 默认 Gitee 账号
Host gitee.com
  HostName gitee.com
  User git
  IdentityFile ~/.ssh/gitee/user1_id_rsa

# 第二个 Gitee 账号
Host your_name.gitee.com
  HostName gitee.com
  User git
  IdentityFile ~/.ssh/gitee/user2_id_rsa


# =========================
# [Other 平台]
# =========================
# 按需扩展
Host other.com
  HostName other.com
  User git
  IdentityFile ~/.ssh/other/user1_id_rsa
```

---

## 四、配置原理说明

1. **SSH 连接格式：**  
   Git 默认通过 `git@github.com:user/repo.git` 识别要连接的主机与用户。  
   - `git@` 前为 **User**（通常为 git）  
   - `github.com` 为 **Host**

2. **冲突原因：**  
   多个账户都使用相同的 `User` 和 `Host`（git@github.com），默认只会使用一把私钥。

3. **解决方案：**  
   在 `config` 中给不同账户设置 **别名 Host**（如 `your_name.github.com`），  
   SSH 就能根据不同 Host 选择对应的密钥。

4. **新的仓库地址写法：**

   ```bash
   git@your_name.github.com:your_github_username/repo.git
   ```

   SSH 会自动识别 `your_name.github.com` 并使用对应的密钥。

---

## 五、验证配置是否成功

以 GitHub 为例 👇

### 默认账号验证

```bash
ssh -T git@github.com
```

### 第二个账号验证

```bash
ssh -T git@your_name.github.com
```

✅ 若出现以下提示，说明配置成功：

```
Hi your_name! You’ve successfully authenticated, but GitHub does not provide shell access.
```

---

## 六、项目使用说明

### 1️⃣ 设置项目独立用户名与邮箱

> 建议取消全局配置，针对每个仓库单独设置。

```bash
# 取消全局设置
git config --global --unset user.name
git config --global --unset user.email

# 进入项目目录后单独设置
git config user.name "your_name"
git config user.email "your_email@example.com"
```

---

### 2️⃣ 绑定远程仓库地址

#### 默认账户仓库

```bash
git remote add origin git@github.com:githubusername/repo.git
```

#### 第二账户仓库

```bash
git remote add origin git@your_name.github.com:githubusername/repo.git
```

---

## ✅ 总结

| 项目 | 默认路径 | 多账户路径 | 配置方式 |
|------|-----------|-------------|------------|
| GitHub | `~/.ssh/github/user1_id_rsa` | `~/.ssh/github/user2_id_rsa` | `config` 设置别名 Host |
| Gitee | `~/.ssh/gitee/user1_id_rsa` | `~/.ssh/gitee/user2_id_rsa` | 同上 |
| Other | `~/.ssh/other/user1_id_rsa` | `~/.ssh/other/user2_id_rsa` | 同上 |

通过以上配置，你可以：
- 同时维护多个 GitHub / Gitee 账户；
- 独立管理 SSH 密钥；
- 方便地在不同项目中切换身份；
- 避免冲突与认证错误。
