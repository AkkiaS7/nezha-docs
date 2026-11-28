---
outline: deep
---

# 登录常见问题

当前 V2 Dashboard 使用本地账户登录，并支持将本地账户绑定到 OAuth2 第三方账户。旧版 GitHub OAuth 登录问题不再适用于当前登录流程。

## 忘记管理员密码

请参考 [初始化用户密码](/guide/q13.html)，推荐只重置指定用户的密码。重置后建议立即登录后台修改为新的强密码。

如果是升级后正确的密码也提示错误，请先看下一节，很可能不需要重置密码。

## 升级到 v2.6.0 及以上后提示密码错误

从 v2.6.0 起，Dashboard 在密码登录时会检查浏览器请求的 `Origin` 是否与 Dashboard 收到的 `Host` 一致（主机名和端口都要相同，不区分大小写），用于防御跨站请求伪造（CSRF）。如果反向代理、CDN 或隧道转发给 Dashboard 的 `Host` 与浏览器地址栏不一致，即使密码正确也无法登录：

- 包含新提示的版本会显示：「登录被拒绝：请求的 Origin 与面板收到的 Host 不一致。如果使用了反向代理，请确认它原样转发了浏览器的 Host 请求头（包括端口）。」
- 更早的 v2.6.0 及以上版本会显示「用户名或密码错误」，与真正输错密码时相同。

包含新提示的版本还会把省略的端口与对应协议的默认端口视为等价（`https` 为 `443`，`http` 为 `80`）：浏览器访问 `https://dashboard.example.com` 时，`Host` 为 `dashboard.example.com:443` 也能登录。更早的 v2.6.0 及以上版本要求两者完全一致。

常见原因：

| 浏览器访问地址 | Dashboard 收到的 Host | 原因 |
| --- | --- | --- |
| `https://dashboard.example.com:8443` | `dashboard.example.com` | Nginx 使用了 `proxy_set_header Host $host;`，`$host` 不带端口 |
| `https://dashboard.example.com` | `127.0.0.1:8008` | Nginx 没有设置 `Host`，或代理、CDN、隧道把 Host 改写成了源站地址 |
| `https://dashboard.example.com` | `dashboard.example.com:443` | 代理在 Host 中加上了默认端口。包含新提示的版本视为一致，可以登录；更早的 v2.6.0 及以上版本会拒绝 |

这类失败不计入 Web 应用防火墙（WAF）的登录失败次数。但如果因此反复尝试其他密码，那些错误的密码会被计入，可能导致 IP 被封禁。

### 自查

1. **通过 SSH 本地转发绕过代理登录**：不要通过公网直接访问 `http://服务器IP:8008` 登录，密码会以明文传输。在自己的电脑上建立 SSH 本地端口转发并保持连接：

   ```bash
   ssh -N -L 8008:127.0.0.1:8008 用户名@服务器IP
   ```

   然后在本机浏览器访问 `http://127.0.0.1:8008/dashboard` 并登录，流量经 SSH 加密传到服务器。如果本机的 `8008` 端口已被占用，把第一个 `8008` 换成其他端口，访问地址随之改变；如果 Dashboard 监听的不是 `8008`，替换第二个 `8008`。直连能登录，说明密码正确，问题出在代理转发的 Host 上。
2. **对比请求头**：在浏览器开发者工具的“网络”面板中找到登录请求 `/api/v1/login`，查看请求头 `Origin`，与代理实际转发给 Dashboard 的 `Host` 对比，两者的主机名和端口应当一致。

### 修正代理配置

- **Nginx**：在转发 Dashboard 的 `location` 中使用 `proxy_set_header Host $http_host;`。`$http_host` 是浏览器发送的原始 Host，包含非标准端口。完整示例见 [反向代理配置](/guide/q3.html)。
- **Caddy**：上游为 HTTP 时（如 `to localhost:8008`），`reverse_proxy` 默认原样转发 Host，不需要额外设置。上游为 HTTPS 时，Caddy 2.11 起默认把 Host 设为上游地址，需要显式写 `header_up Host {hostport}`。显式设置 `header_up Host` 时，请使用 `{hostport}`，不要用不带端口的 `{host}`，也不要改成上游地址（如 `{upstream_hostport}`）。
- **Cloudflare Tunnel**：在 Public Hostname 的 HTTP 设置中，将 **HTTP Host Header** 留空；使用配置文件时不要设置 `originRequest.httpHostHeader`。如果必须设置，请填写浏览器实际访问的域名。
- **其他 CDN、面板软件或负载均衡**：把回源 Host（或“发送域名”）设置为浏览器实际访问的域名；通过非标准端口访问时要带上端口。

不要为了让登录通过而改写 `Origin` 请求头（如在 Nginx 中写 `proxy_set_header Origin ...`，或在 Caddy 中写 `header_up Origin ...`），这会让这项保护失效。应当让转发的 Host 与浏览器地址保持一致。

## OAuth2 登录失败

请先确认以下配置：

1. Dashboard 配置文件中的 `oauth2` 字段使用新版结构，详见 [设置 OAuth 2.0 绑定](/guide/q14.html)。
2. 第三方平台的 Callback URL 为 `https://你的面板域名/api/v1/oauth2/callback`。
3. 已先使用本地账户登录，并在个人信息页完成 OAuth2 绑定。

如果还没有绑定关系，不能直接使用 OAuth2 登录到本地账户。

## 切换网络后登录失效

Dashboard 默认把浏览器 JWT 会话绑定到登录时识别的客户端 IP。切换 Wi-Fi、移动网络、VPN 或代理出口后，可能需要重新登录；这不等同于密码错误或账号被封。

先检查[前端真实 IP 请求头](/guide/q12.html)，确保代理没有把访客 IP 错误识别为共享或不稳定的地址。确需允许会话跨 IP 使用时，可设置 [`allow_jwt_ip_change: true`](/configuration/dashboard.html#allow-jwt-ip-change)，但这会放宽会话保护；默认仍为 `false`，当前管理前端无此开关。

默认用户前端首页遇到游客 API 的 `ApiErrorUnauthorized` 会提示登录并提供 `/dashboard` 入口，不再把此情形显示成后端不可用。启用了 `force_auth` 时应先登录；不要仅凭这个提示判断 Agent 离线。

## 登录接口被频繁尝试或误封

V2 继续提供 Web 应用防火墙，用于限制登录爆破。若 Dashboard 部署在反向代理或 CDN 后面，请正确配置 [前端真实 IP 请求头](/guide/q12.html)，否则可能因为无法识别真实访问者 IP 而出现误判。

如果已经误封，可以使用管理员账户在 **系统设置 → Web 应用防火墙** 中删除对应封禁记录。
