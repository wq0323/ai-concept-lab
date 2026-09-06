# DEVELOPMENT.md — 开发笔记

本仓库从零搭建过程中遇到的真实问题、诊断过程与最终解决方式。仅作记录，便于后续学习者避坑。

---

## 问题 1：GitHub 网页走代理打不开，但 SSH/API 直连通

**现象**：浏览器打开 `https://github.com/settings/tokens` 进不去；命令行用 `curl https://github.com` 返回 HTTP 000 超时。但 `ssh -T git@github.com` 能成功认证为 `wq0323`；`curl https://api.github.com` 返回 HTTP 200。

**诊断**：

```bash
curl -m 8 -s -o /dev/null -w "%{http_code}\n" https://github.com   # 走代理 → 000
curl -m 8 --noproxy '*' -s -o /dev/null -w "%{http_code}\n" https://github.com   # 直连 → 200
```

发现本机环境里有 `http_proxy=http://127.0.0.1:51580` 这条代理变量，**代理对 `github.com` 主站超时，但对 `api.github.com` 和 SSH 22 端口通**。

**最终解决**：

- **网页访问**：让浏览器关掉代理/梯子直连。
- **Git 操作**：继续走 SSH（22 端口，与代理无关），`git push` 从未受影响。
- **API 调用**：用 Python `urllib` 显式禁用代理：`build_opener(ProxyHandler({}))`，保证走直连。

---

## 问题 2：Git Bash 下 `curl -d` 发送中文 JSON，名字被损坏成 "-"

**现象**：第一次用 `curl -X POST https://api.github.com/user/repos -d '{"name":"大数据与人工智能", ...}'` 创建仓库，返回 `"name": "-"`、`html_url` 是 `https://github.com/wq0323/-`。

**诊断**：

- GitHub 的仓库名（`name`）实际上不支持中文字符，会被规范化/替换为 `-`。
- 进一步验证：用 Python 重新发同样请求（但仓库已被占用），报 `"name already exists on this account"`——证明中文名也被规范化为 `-`，与第一次创建的 `-` 冲突。

**最终解决**：

- 仓库 `name`（URL slug）改用 ASCII 英文（`bigdata-ai` / `ai-concept-lab`），中文放在 `description` 和 README 标题里。
- 同时改用 Python `urllib` 发请求（`json.dumps(..., ensure_ascii=False).encode('utf-8')`），避免 Windows Git Bash 下命令行参数被损坏。

---

## 问题 3：Classic Personal Access Token 没有 `delete_repo` 权限

**现象**：想删除过程中产生的垃圾空仓库 `wq0323/-`，`DELETE /repos/wq0323/-` 返回 `403 Must have admin rights to Repository`。

**诊断**：

```
$ curl -H "Authorization: token ghp_xxx" https://api.github.com/user/repos
X-OAuth-Scopes: repo
```

响应头 `X-OAuth-Scopes` 只有 `repo`，**classic token 的删除仓库权限在独立的 `delete_repo` scope**，不在 `repo` 里。Token 持有者实际是仓库 owner，但缺这个 scope 所以删不了。

**最终解决**：

- 本仓库作者后续方便时可手动到网页（`https://github.com/wq0323/-` → Settings → Danger Zone → Delete）删除；或重新生成带 `delete_repo` scope 的 Token。
- 仓库 `wq0323/-` 是否被删除不影响 `wq0323/ai-concept-lab` 的可用性。

---

## 问题 4：`git add -A` 之后 `git status --short` 输出为空，但实际文件已被加入

**现象**：执行 `git add -A && git status --short` 时，`status --short` 没打印 `A` 标记，但随后的 `git log --stat` 显示 commit 包含了所有 7 个文件。

**诊断**：用 `git ls-files` 直接查已跟踪文件清单，发现 `.gitignore`、`.workbuddy/skills/...`、`README.md`、`learning-materials/*.html` 全在；说明 `git add -A` **实际上成功**了，只是 `status --short` 在某种 Windows + Git for Windows 组合下输出有滞后/被截断的怪行为。

**最终解决**：

- 不依赖 `git status --short` 单步反馈，改用 `git ls-files` / `git log --stat -1` 来验证文件确实被跟踪。
- 推送后用 GitHub API `GET /repos/<owner>/<repo>/contents/` 二次确认远程结构。

---

## 问题 5：Windows Git 默认会改换行符，提交时弹出 CRLF 警告

**现象**：首次 commit 后弹出多条警告 "LF will be replaced by CRLF the next time Git touches it"。

**诊断**：`git config core.autocrlf` 在 Windows Git 安装时默认是 `true`（提交时 LF 转 CRLF，检出时反过来）。

**最终解决**：对纯 Markdown / HTML / 配置文件来说，这种自动转换是无害的（不会破坏内容），保持默认即可。如果以后要做跨平台协作、或者 commit hook 对字节敏感，再单独调整。

---

## 工具与命令速查（本项目实际用过）

```bash
# 1) 用 Python 调用 GitHub API（UTF-8 + 禁用代理）
python -c "
import urllib.request, json
TOKEN='ghp_xxx'
opener=urllib.request.build_opener(urllib.request.ProxyHandler({}))
data=json.dumps({'name':'ai-concept-lab','description':'...','public':True}, ensure_ascii=False).encode('utf-8')
r=urllib.request.Request('https://api.github.com/user/repos', data=data, method='POST')
r.add_header('Authorization','token '+TOKEN)
r.add_header('Accept','application/vnd.github+json')
r.add_header('Content-Type','application/json')
print(opener.open(r,timeout=25).read().decode('utf-8'))
"

# 2) 验证远程结构
python -c "
import urllib.request, json
TOKEN='ghp_xxx'
opener=urllib.request.build_opener(urllib.request.ProxyHandler({}))
r=urllib.request.Request('https://api.github.com/repos/wq0323/ai-concept-lab/contents/', method='GET')
r.add_header('Authorization','token '+TOKEN)
r.add_header('Accept','application/vnd.github+json')
for x in json.loads(opener.open(r,timeout=20).read()):
    print(x['type'], x['path'])
"

# 3) 推送
git push -u origin main
```

---

## 安全实践

- **临时脚本用完即删**：含 Token 的 Python 一次性脚本在脚本运行完成后立即 `rm`，避免 Token 残留在工作树。
- **`.gitignore`**：包含 `.env` / `*.pem` / `*.key` / `secrets.*` / `credentials.*` 等敏感文件规则，**任何时候都不要把 Token 直接写进仓库文件**。
- **Token 用完撤销**：建议每次使用完 Personal Access Token 后到 https://github.com/settings/tokens Revoke 掉，避免长期暴露。