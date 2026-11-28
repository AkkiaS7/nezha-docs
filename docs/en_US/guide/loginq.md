---
outline: deep
---

# Login FAQ

The current V2 Dashboard uses local account login and supports binding local accounts to third-party OAuth2 accounts. Legacy GitHub OAuth login issues no longer apply to the current login flow.

## Forgot Administrator Password

See [Initialize User Password](/en_US/guide/q13.html); resetting only the affected user's password is recommended. After resetting it, log in to the admin frontend and change it to a new strong password as soon as possible.

If the correct password is rejected after an upgrade, read the next section first. You probably do not need to reset the password.

## Incorrect Password After Upgrading to v2.6.0 or Later

Since v2.6.0, Dashboard checks during password login that the browser's `Origin` header matches the `Host` header Dashboard receives (both the host name and the port, case-insensitive), to defend against cross-site request forgery (CSRF). If a reverse proxy, CDN, or tunnel forwards a `Host` that differs from the browser's address bar, login fails even with the correct password:

- Versions with the dedicated message show: "Login was refused because the request Origin does not match the Host received by the dashboard. If you use a reverse proxy, make sure it forwards the original Host header, including the port."
- Earlier v2.6.0+ versions show "Invalid Username Or Password", the same message as a real wrong password.

Versions with the dedicated message also treat an omitted port as equal to the scheme's default port (`443` for `https`, `80` for `http`): when the browser visits `https://dashboard.example.com`, a `Host` of `dashboard.example.com:443` can still log in. Earlier v2.6.0+ versions require an exact match.

Common causes:

| Browser address | Host received by Dashboard | Cause |
| --- | --- | --- |
| `https://dashboard.example.com:8443` | `dashboard.example.com` | Nginx uses `proxy_set_header Host $host;`, and `$host` has no port |
| `https://dashboard.example.com` | `127.0.0.1:8008` | Nginx does not set `Host`, or a proxy, CDN, or tunnel rewrites Host to the origin address |
| `https://dashboard.example.com` | `dashboard.example.com:443` | The proxy adds the default port to Host. Versions with the dedicated message treat it as a match and allow login; earlier v2.6.0+ versions reject it |

These failures are not counted by the Web Application Firewall (WAF) as failed logins. However, if you then try other passwords, those wrong passwords are counted and may get your IP blocked.

### Self-Check

1. **Bypass the proxy through an SSH local forward**: Do not log in through `http://server-ip:8008` over the public internet, because the password would be sent in plaintext. On your own computer, create an SSH local port forward and keep it open:

   ```bash
   ssh -N -L 8008:127.0.0.1:8008 user@server-ip
   ```

   Then open `http://127.0.0.1:8008/dashboard` in your local browser and log in; the traffic reaches the server encrypted through SSH. If local port `8008` is already in use, replace the first `8008` with another port and adjust the address accordingly; if Dashboard does not listen on `8008`, replace the second `8008`. If the direct login works, the password is correct and the problem is the Host forwarded by the proxy.
2. **Compare the headers**: In the browser developer tools Network panel, find the login request `/api/v1/login` and check its `Origin` request header. Compare it with the `Host` the proxy actually forwards to Dashboard; the host name and port must match.

### Fix the Proxy Configuration

- **Nginx**: In the `location` blocks that proxy Dashboard, use `proxy_set_header Host $http_host;`. `$http_host` is the original Host sent by the browser, including any non-standard port. See [Reverse Proxy Configuration](/en_US/guide/q3.html) for a full example.
- **Caddy**: With an HTTP upstream (such as `to localhost:8008`), `reverse_proxy` forwards Host unchanged by default, so no extra setting is needed. With an HTTPS upstream, Caddy 2.11 and later set Host to the upstream address by default, so you must set `header_up Host {hostport}` explicitly. When setting `header_up Host` explicitly, use `{hostport}`, not `{host}` (which has no port), and do not replace it with the upstream address (such as `{upstream_hostport}`).
- **Cloudflare Tunnel**: In the Public Hostname HTTP settings, leave **HTTP Host Header** empty; in a configuration file, do not set `originRequest.httpHostHeader`. If you must set it, use the domain the browser actually visits.
- **Other CDNs, hosting panels, or load balancers**: Set the origin Host (or "origin domain") to the domain the browser actually visits, and include the port when accessing through a non-standard port.

Do not rewrite the `Origin` header to make login pass (such as `proxy_set_header Origin ...` in Nginx, or `header_up Origin ...` in Caddy). That defeats this protection. Make the forwarded Host match the browser address instead.

## OAuth2 Login Failed

Check the following first:

1. The `oauth2` field in the Dashboard configuration uses the new structure. See [Setting Up OAuth 2.0 Binding](/en_US/guide/q14.html).
2. The Callback URL on the third-party platform is `https://your-dashboard-domain/api/v1/oauth2/callback`.
3. You have already logged in with a local account and completed OAuth2 binding on the profile page.

If no binding relationship exists yet, OAuth2 cannot directly log in to a local account.

## Session Lost After Switching Networks

Dashboard binds browser JWT sessions to the client IP identified at login by default. Switching Wi-Fi, mobile networks, VPNs, or proxy exits may require signing in again. This does not necessarily mean an incorrect password or a blocked account.

First verify the [Frontend Real IP Request Header](/en_US/guide/q12.html), so the proxy does not misidentify visitors as a shared or unstable address. If sessions must survive IP changes, set [`allow_jwt_ip_change: true`](/en_US/configuration/dashboard.html#allow-jwt-ip-change). This relaxes session protection; the default remains `false`, and the current admin frontend has no switch for it.

The default user frontend homepage shows a login prompt with a `/dashboard` link when a guest API returns `ApiErrorUnauthorized`, rather than reporting backend unavailability. Sign in first when `force_auth` is enabled; this prompt alone does not indicate offline Agents.

## Login Endpoint Is Frequently Tried or Incorrectly Blocked

V2 includes a Web Application Firewall to limit login brute-force attempts. If Dashboard is deployed behind a reverse proxy or CDN, configure the [Frontend Real IP Request Header](/en_US/guide/q12.html) correctly; otherwise, Dashboard may misidentify visitor IPs and block incorrectly.

If an IP has already been blocked by mistake, use an administrator account to remove the corresponding block record under **System Settings → Web Application Firewall**.
