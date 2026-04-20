# 部署指南 — 中国 / 海外双可用

总目标：海外访问走 **Cloudflare Pages**，中国访问走 **腾讯云 CDN 反代**。
用户在两边的加载体验都应该在 1s 内。

---

## 时间轴

| 阶段 | 时长 | 代价 |
|------|------|------|
| Cloudflare Pages 上线（海外就绪） | ~10 分钟 | 免费 |
| 买域名（国内商用 + ICP 备案） | ~¥55/年 + 15–20 天等待 | 备案需要身份证 + 手持照 |
| 腾讯云 CDN 配置 | ~30 分钟 | ~¥20–50/月 流量费 |
| **合计：从零到双地区可用** | **~3 周（卡在备案）** | **~¥300/年** |

---

## 阶段 1：Cloudflare Pages 部署（10 分钟，立刻海外可用）

### 1.1 准备 GitHub repo

```bash
# 本地新建文件夹，把项目文件全部放进去（保持结构）
cd ~/Desktop
mkdir vernon-site && cd vernon-site
# 把这些文件复制进来：
#   index.html
#   papers/
#   resume/
#   _headers
#   _redirects
#   robots.txt
#   README.md
#   .gitignore
# （不要复制 uploads/ 和 Vernon Ouyang.html / Vernon Ouyang v1.html）

git init
git add .
git commit -m "init: deploy-ready site"

# 去 github.com 新建 public repo：vernon-site
# 然后：
git branch -M main
git remote add origin git@github.com:VernonOY/vernon-site.git
git push -u origin main
```

### 1.2 Cloudflare Pages

1. 打开 <https://dash.cloudflare.com> → 左侧 **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. 授权 GitHub，选 `vernon-site` repo
3. 构建设置：
   - **Production branch**: `main`
   - **Framework preset**: `None`
   - **Build command**: 留空
   - **Build output directory**: `/`（根目录）
4. **Save and Deploy**
5. 2 分钟后会给你一个 URL：`vernon-site.pages.dev`

✅ 海外访问已就绪。全球 Cloudflare 节点会自动缓存。

---

## 阶段 2：买域名 + ICP 备案（15–20 天）

**为什么必须备案**：没有 ICP 备案的域名，国内 CDN（阿里云/腾讯云）**拒绝解析**。这是法规，无解。

### 2.1 买域名

- 推荐：**腾讯云** <https://dnspod.cloud.tencent.com>（国内商用、备案便捷）
- 建议买 `.com`（国际通用）
- 价格：约 ¥55–80 / 年
- 常见选择：`vernon-ouyang.com` / `wendiouyang.com` / `vernonoy.com`

**重要**：域名必须在**国内运营商**（腾讯云/阿里云）注册或**过户到国内**，才能备案。海外注册商（Namecheap/GoDaddy）买的 `.com` **不能直接备案**。

### 2.2 ICP 备案（首次备案）

1. 腾讯云控制台 → **备案** → **首次备案**
2. 选择"个人备案"（学生身份可做）
3. 需要准备：
   - 身份证正反面照片
   - 手持身份证照片
   - 域名证书（腾讯云自动生成）
   - 一台腾讯云国内服务器 **或** 腾讯云 CDN 产品（不用真的部署，过了备案就行；或直接开 CDN 试用）
4. 提交后：腾讯云初审 1–3 天 → 工信部审核 7–20 天（看省份，广东较快）
5. 备案通过后会收到短信 + 邮件，拿到**备案号**（例如 `粤ICP备2024xxxxxx号`）

**备案期间可以做什么**：继续用 Cloudflare Pages。国内用户勉强能打开（不稳但能用）。

### 2.3 备案号上墙

备案通过后，**法规要求**网站底部显示备案号。在 `index.html` 的 `<footer>` 里添加：

```html
<a href="https://beian.miit.gov.cn/" target="_blank" rel="noopener">
  粤ICP备2024xxxxxx号
</a>
```

（拿到备案号后我可以帮你加。）

---

## 阶段 3：腾讯云 CDN 反代（30 分钟，中国加速）

备案通过后操作。

### 3.1 把自定义域名接到 Cloudflare Pages

1. Cloudflare dashboard → Pages → 你的项目 → **Custom domains** → **Set up a custom domain**
2. 输入你的域名（例如 `vernon-ouyang.com`）
3. 按提示：**把域名的 NS（name server）改到 Cloudflare** 或保留现有 DNS 商
4. **建议：把主域名 `vernon-ouyang.com` 交给 Cloudflare 管理，海外直连**

### 3.2 在腾讯云 CDN 开子域名给国内用户

1. 腾讯云控制台 → **内容分发 CDN** → **域名管理** → **添加域名**
2. 加速域名：`cn.vernon-ouyang.com`（子域名，国内用户走这个）
3. 业务类型：静态加速
4. **源站配置**：
   - 源站类型：**域名**
   - 源站：`vernon-site.pages.dev`
   - 回源 Host：`vernon-site.pages.dev`
   - 协议：HTTPS
5. 申请 HTTPS 证书（腾讯云免费 DV 证书，或 Let's Encrypt）
6. DNS：`cn.vernon-ouyang.com` → CNAME → `xxx.cdn.dnsv1.com`（腾讯云会给你）

### 3.3 智能分流（DNS 层面自动分国内/海外）

推荐用 **DNSPod（腾讯云）**，它支持"境内/境外线路"分流：

- `vernon-ouyang.com` 境外线路 → CNAME 到 `vernon-site.pages.dev`
- `vernon-ouyang.com` 境内线路 → CNAME 到 `xxx.cdn.dnsv1.com`（腾讯云 CDN）

这样所有用户访问**同一个域名**，DNS 自动按地区分流，体验无缝。

---

## 阶段 4：验证

```bash
# 测国内解析
dig @119.29.29.29 vernon-ouyang.com
# 应该看到腾讯云 CDN IP

# 测海外解析
dig @1.1.1.1 vernon-ouyang.com
# 应该看到 Cloudflare IP
```

用手机 4G（国内）+ VPN（海外）分别打开，加载时间应 < 1s。

---

## 日常更新流程

```bash
# 本地改完 index.html
git add index.html
git commit -m "update: xxx"
git push
# Cloudflare Pages 自动重新部署（1–2 分钟）
# 腾讯云 CDN 会自动回源拿新内容，或手动刷新缓存：
# 腾讯云 CDN → 缓存配置 → URL 刷新 → 输入 https://cn.vernon-ouyang.com/index.html
```

---

## 常见问题

**Q: 我现在人在海外，也能做 ICP 备案吗？**
A: 可以，但需要回国内或找人协助拍手持照、视频核验。部分省份支持 App 远程核验。

**Q: 能不能不备案、纯海外部署？**
A: 可以，就走 Cloudflare Pages + 主域名。国内用户大部分能打开，但不稳定，加载可能 3–10 秒，有时完全打不开。对个人简历站够用，对求职面试官展示就不够稳。

**Q: Cloudflare Pages 免费额度够吗？**
A: 免费 500 build/月 + 无限带宽 + 无限请求。对个人站完全够。

**Q: 腾讯云 CDN 月费大概多少？**
A: 按流量算，简历站月流量 ~5 GB，约 ¥10–30/月。

---

## 我能帮你做的

告诉我你的进度，我可以：

- [ ] 备案通过后，帮你把备案号加到 footer
- [ ] 生成 `sitemap.xml` 和 OG 图（社交分享预览）
- [ ] 写一份 `manifest.webmanifest` 让手机主屏可添加
- [ ] 写 Cloudflare Worker 做更精细的分流（A/B 测试、地区重定向）
- [ ] 帮你调 `_headers` 的 CSP / security headers
