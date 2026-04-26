# cogra.xyz 最低成本上线方案（Cloudflare Pages 免费版）

你现在的情况是：

- Pages 部署成功（Cloudflare 后台显示 `成功`）。
- 但浏览器访问出现：
  - `cogra.xyz` -> `DNS_PROBE_FINISHED_NXDOMAIN`
  - `*.pages.dev` 预览域名 -> `ERR_CONNECTION_CLOSED`

这说明**不是代码问题**，而是「域名解析 / 本地网络访问」问题。

---

## 1) 先确认：代码已经没问题

本仓库是纯静态站，`index.html` 可直接由 Pages 发布，无需构建命令。  
只要 DNS 正常，页面就会打开。

---

## 2) 必做排查（按顺序）

> 建议按顺序做，前一项不通过就先别做下一项。

### A. 检查注册商 NS 是否真的托管到 Cloudflare

在终端运行：

```bash
dig NS cogra.xyz +short
```

期望结果：返回 Cloudflare 分配给你域名的那两条 NS（例如 `xxx.ns.cloudflare.com`）。

如果不是 Cloudflare NS：

1. 去你买域名的注册商控制台（阿里云域名控制台）。
2. 把 NS 改为 Cloudflare 给你的两条。
3. 等待生效（通常几分钟到数小时，最慢可到 24~48 小时）。

> 你截图里的 `NXDOMAIN` 最常见原因就是这一步没完全生效。

### B. 检查 Cloudflare DNS 记录是否存在且无冲突

在 Cloudflare -> `cogra.xyz` -> DNS，至少要有：

- `CNAME` 记录
  - 名称：`@`
  - 目标：`my-page-1bg.pages.dev`
  - 代理状态：已代理（橙云）

可选再加：

- `CNAME` 记录
  - 名称：`www`
  - 目标：`my-page-1bg.pages.dev`
  - 代理状态：已代理（橙云）

同时确认：

- 不要再有冲突的 `A/AAAA @` 记录（指向别的 IP）。
- 不要有把 `@` 指到错误目标的旧记录。

### C. 在 Pages 项目里确认自定义域是 Active

Cloudflare -> Workers & Pages -> `my-page` -> `自定义域`：

- `cogra.xyz` 状态应为 `Active` / `有效`。
- 若不是，删除后重新添加一次 `cogra.xyz`。

### D. 清理本地缓存后再测

本地 DNS 可能缓存旧结果。可做：

- 换手机 4G/5G 网络直接访问 `https://cogra.xyz`
- 或切换公共 DNS（如 1.1.1.1 / 8.8.8.8）后再试

快速检查：

```bash
nslookup cogra.xyz 1.1.1.1
nslookup cogra.xyz 8.8.8.8
```

如果两边都能解析到 Cloudflare 结果，说明解析已正常。

---

## 3) 关于 `*.pages.dev` 打不开（ERR_CONNECTION_CLOSED）

这通常是本地网络、运营商、代理或安全软件导致的临时连接问题，不影响你的站点是否已部署成功。  
**以 `cogra.xyz` 能否访问作为最终验收标准**。

---

## 4) 你可以直接照着做的“最短修复路径”

1. `dig NS cogra.xyz +short` 看 NS 是否是 Cloudflare。  
2. Cloudflare DNS 里只保留 `@ -> my-page-1bg.pages.dev`（CNAME, 橙云）。  
3. Pages 的 `自定义域` 里确认 `cogra.xyz` 显示 Active。  
4. 切手机流量访问 `https://cogra.xyz` 验证。

---

## 5) 后续发布

以后每次更新网页：

```bash
git add .
git commit -m "update site"
git push
```

Cloudflare Pages 会自动重新部署。

---

## 当前仓库内容

- `index.html`：上线首页。
- `README.md`：包含本次故障（NXDOMAIN / 连接关闭）的排查手册。
