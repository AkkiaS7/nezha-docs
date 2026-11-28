---
outline: deep
---

# Settings

**After logging in to the admin page, click the avatar → System Settings to open the settings page.**

---

## System Configuration

### Site Name

Customize the site title here for easier identification and management.

### Language Settings

Set the software language, mainly affecting notifications and error messages.

The web interface can also change language in real time.

### Custom Code

This item is used to add custom styles or scripts to the web interface, such as changing the logo, background image, theme, external links, or analytics code.

This item is split into two fields:

- **User frontend custom code**: corresponds to `custom_code` in the configuration file and is injected into the public user frontend.
- **Admin frontend custom code**: corresponds to `custom_code_dashboard` in the configuration file and is injected only into the logged-in admin frontend.

The following global variables are read only by the user frontend. They do not take effect if placed in admin frontend custom code. Set them in user frontend custom code with `<script>...</script>`; use `true` / `false` for booleans, not strings.

- `window.CustomBackgroundImage`: Desktop background image URL.
- `window.CustomMobileBackgroundImage`: Mobile background image URL. If unset, the desktop background image is used.
- `window.CustomLogo`: Logo image URL.
- `window.CustomDesc`: Homepage description text.
- `window.CustomLinks`: Custom external links. The value must be a JSON string parseable by `JSON.parse`, in the format `[{"name":"GitHub","link":"https://github.com/nezhahq/nezha"}]`.
- `window.ForceTheme`: Force the default color theme. Value is `"light"` or `"dark"`.
- `window.ShowNetTransfer`: Whether to show inbound/outbound traffic on server cards.
- `window.DisableAnimatedMan`: Whether to disable the default animated character illustration.
- `window.CustomIllustration`: Custom illustration URL. After setting it, it replaces the default illustration, and `window.DisableAnimatedMan` is usually unnecessary.
- `window.FixedTopServerName`: Whether to pin the top server name.
- `window.ForceUseSvgFlag`: Whether to force SVG flags.
- `window.ForceShowServices`: Whether to expand the homepage service monitoring panel by default.
- `window.ForceCardInline`: Whether to force the horizontal server list layout on non-mobile devices. Mobile devices still use the card layout.
- `window.ForceShowMap`: Whether to expand the homepage map by default.
- `window.ForcePeakCutEnabled`: Whether to enable network chart peak clipping by default, reducing the impact of sudden spikes on chart readability.

User frontend example:

```html
<script>
  window.CustomLogo = "https://example.com/logo.png";
  window.CustomDesc = "My Monitor";
  window.CustomLinks = JSON.stringify([
    { name: "GitHub", link: "https://github.com/nezhahq/nezha" }
  ]);
  window.ForceTheme = "dark";
  window.ForceShowServices = true;
</script>
```

- Admin frontend
    1. `window.DisableAnimatedMan`: Boolean, turns the animated character illustration on/off.

### Theme

The theme setting selects the user frontend template and corresponds to [`user_template`](/en_US/configuration/dashboard.html#user-template). The admin frontend template corresponds to [`admin_template`](/en_US/configuration/dashboard.html#admin-template) and usually should stay at the default value.

Built-in themes are maintained by the official project. Community themes are contributed by the community and may not be updated together with Dashboard releases. Before using a community theme, make sure the source is trusted, it is compatible with your Dashboard version, and you understand the risks from custom code, external assets, or style changes.

### Agent Connecting Address

- This is required when installing Agents with the one-click script.
- Set it to the Agent connection address you want to use, for example: `data.example.com:8008`.
- If Agents connect through HTTPS, Dashboard built-in HTTPS, or a reverse proxy/CDN that terminates TLS, enable **Agent uses TLS connection**. The generated installation command will include the TLS option.
- If Agents connect directly to a plaintext `host:port`, do not enable TLS. Otherwise the installation command will make Agents connect with TLS and they may fail to come online.
- See [Agent Installation Prerequisites](/en_US/guide/agent.html#prerequisites) for details.

### Dashboard OAuth2 Callback Host

Specifies the public Dashboard address that receives OAuth2 callbacks, in the format `domain/IP[:port]`. Do not include a scheme or path; omit the port when using the default HTTP/HTTPS port. It is independent of **Agent Connecting Address**: the former is used for browser login callbacks, while the latter is used for Agent connections and installation commands.

Set it explicitly when using OAuth2, especially when Dashboard is behind a reverse proxy or its public hostname differs from the Agent connection address. When empty, Dashboard trusts the request Host. The reverse proxy should also preserve the original Host and pass `X-Forwarded-Proto: https` for an HTTPS public endpoint. The configuration field is [`dashboard_host`](/en_US/configuration/dashboard.html#dashboard-host).

### Reserved Dashboard Hosts

Declares other public hostnames used to access Dashboard, preventing normal members from creating NAT domains that collide with Dashboard entry hosts. These hostnames are also trusted as OAuth2 callback Hosts. Separate multiple hostnames with commas and register the corresponding callback URLs with the OAuth2 provider. The configuration field is [`reserved_hosts`](/en_US/configuration/dashboard.html#reserved-hosts).

### Custom Public DNS Nameservers for DDNS

Used by DDNS to query domain SOA records. If empty, the built-in list is used.

### Frontend Real IP Request Header

Used to identify the real client IP of visitors accessing the admin frontend and user frontend. Online users, audit records, login brute-force protection, and the Web Application Firewall all depend on it.

For direct public access, select **Use Direct IP**. For Cloudflare, usually use `CF-Connecting-IP`. For Nginx, Caddy, Apache, or another reverse proxy, the proxy must pass `X-Real-IP`, `nz-realip`, or another matching header. For complete scenarios, reverse proxy examples, verification, and recovery, see [Frontend Real IP Header Configuration](/en_US/guide/q12.html).

### Agent Real IP Request Header

- `CF-Connecting-IP` is a request header used to get the Agent's real IP.
- When Agents connect to the Dashboard through Cloudflare CDN proxy, enabling this feature lets the origin server identify the Agent's real IP correctly.

### MCP Toggle

Enables or disables the Dashboard MCP endpoint. After enabling it, clients can call `POST /mcp` with a PAT. When disabled, Dashboard rejects new MCP requests and interrupts related transfers. The configuration field is [`enable_mcp`](/en_US/configuration/dashboard.html#enable-mcp). For authentication and permission details, see [API Interface - MCP Access](/en_US/guide/api.html#mcp-access).

### Online Stats Range (days)

This field maps to [`online_stats_days`](/en_US/configuration/dashboard.html#online-stats-days). It defaults to 7 days and accepts integers from 7 to 3650, controlling online-history retention and the maximum query window. It does not change TSDB metric retention or expand the default user frontend's 1/7/30-day options to arbitrary durations.

Reducing the range causes closed sessions outside the window to be periodically deleted. Increasing it later cannot restore deleted history. See [Online history](/en_US/guide/servers.html#online-history) for display behavior.

### IP Change Notification

This feature sends notifications when a server IP address changes. Configure it as follows:

#### Configuration Options

1. **Cover**
   Select a rule to determine which servers need monitoring.

2. **Specific Servers**
   Used together with Cover. When Cover is **Cover all**, this means **exclude specific servers**. When Cover is **Ignore all**, this means **only specific servers**.

3. **Send notification to notification group**
   Select notification methods. Notification methods must be configured in advance on the Notifications page.

4. **Enable**
   After configuration, check **Enable** to make notifications take effect.

5. **Show full IP address in notification**
   By default, IP change notifications hide the full IP address.
   Check this option if you need to show the complete IP address.

---

## API Tokens

This tab creates and revokes Personal Access Tokens (PATs). PATs are suitable for automation scripts, CI, LLM tools, and MCP clients.

Normal members only see the **API Tokens** tab on the settings page. Other tabs such as system configuration, user management, online users, and Web Application Firewall are visible only to administrators.

When creating a token, select scopes, optional server ID whitelist, and expiration. The plaintext token is shown only once after creation. Later lists only show metadata such as scopes, server whitelist, expiration time, last used time, and last used IP. For authentication format and scopes, see [API Interface - Personal Access Token](/en_US/guide/api.html#personal-access-token-pat).

---

## Database

This tab shows how much space the SQLite database takes and lets you clean up old service monitoring history by hand, to reduce the database and [backup](/en_US/guide/backup.html) size and shorten restores. It is visible only to administrators.

Without TSDB, service monitoring history is stored in the SQLite table `service_histories`. It is never cleaned up automatically, so the database keeps growing as the dashboard runs.

### Usage and Estimate

The top of the page shows the current state:

- **Database size**: the size of the database file, including the WAL file that has not been checkpointed yet.
- **Reclaimable by compaction now**: space already freed inside the database but not yet returned to the disk; a compaction returns it.
- **Service history records** and **Oldest record**.
- **Free disk space**: the free space of the disk holding the database, shown as unknown when it cannot be determined.

After you choose a retention period, the page estimates the cleanup: the records to delete, the space it would free, and the database size after compaction. A cleanup cannot run when there is nothing to delete. The space to free is the history's share of the database scaled by the share of records to delete, plus the space already freed. It is an estimate, usually within 10% of the result and slightly high. When the database is large and the exact count times out, the record count is approximated and marked "(record count is approximate)".

![Database tab after choosing a retention period](/images/database/1_en_US.png)

::: tip One-time change after upgrading
The first start of a version with this feature drops a redundant single-column `created_at` index of `service_histories` (a composite index already covers that column), which saves about a fifth of the table's size. The freed space then shows as **Reclaimable by compaction now**; one compaction returns it to the disk.
:::

### What Is Deleted

Keep history for 30 days (default), 90 days, 180 days, or a custom 1–3650 days. A cleanup deletes:

- per-server service history older than the retention period (the data behind the 1-day, 7-day and 30-day service charts);
- 15-minute summaries (the data behind the 30-day status bar) older than the retention period. **Summaries are always kept for at least 30 days**: with a retention under 30 days they are still kept for 30 days;
- all history of deleted services, however recent.

With less than 30 days kept, the 30-day charts are incomplete; with less than 7 days, the 7-day charts are incomplete as well. The 30-day status bar is not affected.

Records are deleted in the background in small batches with pauses in between, so agent reports and page requests carry on meanwhile. The page shows the current stage (deleting, compacting, restarting) and, when the cleanup finishes, the records deleted and the space freed.

### Compaction (VACUUM)

Deleting records alone does not shrink the database file: the space is returned to the disk only by a compaction (SQLite `VACUUM`). Choose how to compact:

- **Compact now**: compact right after deleting. The whole database is locked while compacting; dashboard requests wait for it for up to about 5 seconds and then fail, so the dashboard may stall or ask you to sign in again, and some monitoring data reported by agents meanwhile is lost. Everything recovers when compaction ends.
- **Compact while restarting the dashboard**: after deleting, the dashboard restarts and compacts the database while starting, before it serves anything. It is unavailable meanwhile; signed-in sessions and agents are disconnected and reconnect automatically. This needs the dashboard to run under Docker or systemd with automatic restart (such as Docker's `restart: always` or systemd's `Restart=always`). The dashboard can only detect the environment, not the restart policy: without automatic restart it has to be started by hand after it exits. When no such environment is detected, this option is unavailable.
- **Do not compact**: only delete the records; the file size does not change. The freed space is reused by new data, and choosing a compaction in the next cleanup returns it.

The page estimates how long a compaction takes (from the data left after the cleanup, about 15 seconds per GiB) and recommends a mode: compact now when it is expected to take no more than 3 seconds, otherwise compact while restarting; when the dashboard cannot restart itself, compact now at a quiet time or do not compact. The recommended mode is selected by default.

A compaction needs temporary disk space: SQLite first writes the compacted database to its temporary directory, then copies it back while keeping the overwritten content in a rollback journal next to the database. The page shows the space needed, about the current database size plus the size after compaction. When the free disk space is short, the page only allows deleting records; if the space turns out short when the cleanup runs, the dashboard skips the compaction and says so in the result.

If a compaction while restarting fails, or the dashboard stops during it, the dashboard still starts normally, does not try that compaction again, and shows the reason in the result. The deleted records are not affected.

### Back Up First

Deleted history cannot be recovered. If you may need it later, take a backup in the **Backup** tab first (the confirmation dialog links there as **Go to backups**). A cleanup asks for confirmation twice: first for the retention period, the estimate and the compaction mode, then for the deletion itself.

![First confirmation](/images/database/2_en_US.png)

![Second confirmation](/images/database/3_en_US.png)

Only one database cleanup, compaction or restore runs at a time: starting another meanwhile is refused with "another database cleanup or maintenance is in progress".

### With TSDB Enabled

With [TSDB](/en_US/guide/q15.html) enabled, service monitoring history is stored in TSDB and the `service_histories` table is no longer used. The Database tab then only shows the usage, without the cleanup, and the history's retention is set by [`tsdb.retention_days`](/en_US/configuration/dashboard.html#tsdb).

---

## User Management

This tab allows adding multiple users to the Dashboard.

See [Multi-User](/en_US/guide/user.html) for details.

---

## Online Users

View current guests / users connected to the frontend, including **IP** and **connection time**, and manually block users. Administrators can also batch block selected online users.

---

## Web Application Firewall

The Web Application Firewall depends on **Frontend Real IP Request Header** to identify source IPs. Before configuring a real-IP header, read [Frontend Real IP Header Configuration](/en_US/guide/q12.html) and confirm Dashboard reads the real client IP.

You can view current block entries and related useful information:

- **IP**: The blocked user's IP.
- **Block ID**: The resource identifier the blocked user attempted to attack. Together with IP, it identifies a block entry. Usually this is a user ID, but there are these special cases:
  1. Attempting to brute force a gRPC connection secret;
  2. Attempting to brute force an API Token;
  3. Attempting to log in to a nonexistent user;
  4. Manual block through the Online Users page.
- **Count**: Number of times this entry has been blocked. Higher counts result in longer block times.
- **Reason**: The reason for the latest block on this entry.
- **Time**: The latest block time for this entry.

If the current user is an administrator, block entries can be deleted manually or in batches.
