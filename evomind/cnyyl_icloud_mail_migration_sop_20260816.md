# 澳瑞虹 `cnyyl.com` 邮箱迁移至 iCloud+ 自定义域执行手册
## 版本：V1.0（先邮箱，网站/网页域名部署暂缓）
## 适用对象：执行 Agent / IT 运维 / 域名 DNS 操作人员 / PPT-PDF 制作 Agent

---

# 0. 项目目标

将澳瑞虹国际贸易（上海）有限公司现有 `@cnyyl.com` 邮箱体系，从当前“老板邮局”迁移到 **Apple iCloud+ Custom Email Domain（自定义电子邮件域）**。

本阶段只处理：

- `cnyyl.com` 邮件收发；
- Apple 账号与邮箱地址分配；
- DNS 邮件记录；
- 旧邮件备份/迁移；
- 收发测试；
- 回滚方案。

本阶段 **不处理**：

- `www.cnyyl.com` 网站迁移；
- Cloudflare Pages；
- 家庭服务器；
- Cloudflare Tunnel；
- 网站主机续费；
- 网站 SSL/CDN；
- 网站备案变更。

原则：**邮箱先独立打通，网站保持原状。**

---

# 1. 先做可行性判断：不要直接改 DNS

## 1.1 Apple 当前能力边界

iCloud+ 自定义邮件域允许：

- 1 个域所有者；
- 最多共享给 **5 个其他人**；
- 即同一个域最多可对应 **6 个 Apple Account 使用者**；
- 每个人在同一个域下最多 **3 个活动邮箱地址**；
- 每个参与者必须有 Apple Account；
- 必须启用双重认证（2FA）；
- 必须已经创建主要的 `@icloud.com` 邮箱；
- 域所有者负责添加/删除成员；
- 非家庭共享成员加入时，需要满足 Apple 当前的 iCloud+ 条件；
- 不支持 Managed Apple Accounts（受管 Apple 账号）。

## 1.2 最重要的限制：多个 Apple 账号 ≠ 共同访问同一个邮箱

Apple 的设计是：

> 一个具体的 `name@cnyyl.com` 地址，归属/分配给一个域成员的 Apple Account。

例如：

- `jimmy@cnyyl.com` → Apple Account A
- `sales@cnyyl.com` → Apple Account B
- `finance@cnyyl.com` → Apple Account C

这是可行的。

但下面这种“共享邮箱”需求 **不是 iCloud Custom Email Domain 的原生能力**：

- Apple Account A、B、C 三个人同时登录并共同管理同一个 `sales@cnyyl.com` 收件箱；
- 三个人共享已发送、草稿、已读/未读状态；
- 类似 Microsoft 365 Shared Mailbox / Google Group Collaborative Inbox 的企业共享邮箱。

Apple 支持把一个地址从某个成员 **转移** 给另一个成员，这进一步说明地址是按成员归属，而不是原生共享邮箱模型。

### 决策门槛

在任何 DNS 修改前，先填写下面表格：

| 现有邮箱地址 | 实际使用人 | 是否多人共用 | 是否必须保留原地址 | 旧邮件是否必须迁移 | 计划 Apple Account |
|---|---|---:|---:|---:|---|
| 例：jimmy@cnyyl.com | Jimmy | 否 | 是 | 是 | Apple Account A |
| 例：sales@cnyyl.com | 销售多人 | 是 | 是 | 是 | **需重新评估** |
|  |  |  |  |  |  |

### STOP 条件

出现以下任意情况，不要切换 MX：

1. 实际需要独立邮箱的人员超过 6 人；
2. 某个关键地址必须由多人共享同一个收件箱；
3. 公司需要离职员工邮件集中留存、审计、管理员强制接管；
4. 需要 10 个独立员工邮箱且每人独立登录；
5. 需要企业级合规、日志、保留策略、邮件归档。

此时应继续使用企业邮箱 SaaS，而不是 iCloud Custom Email Domain。

---

# 2. 当前环境基线

已知：

- 域名：`cnyyl.com`
- 当前邮箱服务：老板邮局 / Chinaemail 体系
- 当前企业邮箱由澳瑞虹自己控制
- 当前服务有效期：至 2027-03-12
- 当前合同是 10 用户邮箱套餐
- 网站暂时不迁移
- DNS 当前仍由外部人员/代理后台操作

当前已知邮件 DNS（以现有后台截图为准）包括：

- `mail` CNAME → `m200n.chinaemail.cn`
- `pop3` CNAME → `p200n.chinaemail.cn`
- `smtp` CNAME → `s200n.chinaemail.cn`
- `@` MX → `mx200n.chinaemail.cn`
- `@` MX → `mxa200n.chinaemail.cn`
- `@` TXT → 当前 Chinaemail SPF
- `_dmarc` TXT → 当前 DMARC

> 注意：切换到 iCloud 前，必须再次导出/截图完整 DNS；不能只根据旧截图操作。

---

# 3. 推荐迁移策略

采用 **“准备 → 验证 → 备份 → DNS 切换 → 双向测试 → 观察 → 再下线旧服务”**。

不建议：

- 一上来直接删旧 MX；
- 同时迁网站；
- 同时换 NS；
- 同时迁 Cloudflare；
- 同一天停掉老板邮局；
- 没有备份就切换；
- 让 Agent 根据网上文章自行猜 DNS 值。

---

# 4. Apple Account 准备

## 4.1 域所有者账号

选择一个长期由澳瑞虹控制的 Apple Account 作为域所有者。

建议：

- 不使用临时员工账号；
- 不使用可能离职人员账号；
- 开启 2FA；
- 绑定公司可持续控制的恢复方式；
- 确认已有主要 `@icloud.com` 邮箱；
- 确认 iCloud+ 订阅有效。

记录：

- 域所有者 Apple Account：__________
- iCloud+ 套餐：__________
- 2FA：已开启 / 未开启
- `@icloud.com` 主邮箱：__________

## 4.2 其他使用者

每个计划使用 `@cnyyl.com` 的人确认：

- [ ] 有独立 Apple Account
- [ ] 已开启 2FA
- [ ] 已创建主要 `@icloud.com` 邮箱
- [ ] Apple Account 不把待迁移的 `@cnyyl.com` 地址作为另一个 Apple Account 的占用地址
- [ ] 确定该成员需要哪 1–3 个 `@cnyyl.com` 地址
- [ ] 确认成员数量总计不超过 6 人（域所有者 + 最多 5 人）

## 4.3 家庭共享与非家庭成员

如果成员处于同一个 Apple Family Sharing：

- 可以通过家庭共享使用 iCloud+ 相关能力；
- 适合家庭/固定合伙人场景。

如果成员不在家庭共享：

- Apple 允许域共享给家庭外成员；
- 按 Apple 当前规则，家庭外成员需要满足其 iCloud+ 条件；
- 不要默认“域主一个 iCloud+ 就能覆盖任意 5 个外部公司员工”。

企业实际落地前，逐个成员在 Apple 页面验证邀请资格。

---

# 5. 旧老板邮局盘点与备份

## 5.1 导出所有现有邮箱地址

从老板邮局管理员后台导出或人工记录：

- 邮箱地址；
- 姓名；
- 容量；
- 已用空间；
- 转发规则；
- 自动回复；
- 黑白名单；
- 邮箱别名；
- 邮件群组；
- 是否多人共享；
- 是否存在 Catch-all；
- 最近登录/是否仍使用。

## 5.2 旧邮件备份

每个关键邮箱至少执行一种备份：

### 方案 A：Apple 官方“Import Mail”
在 iCloud Custom Email Domain 设置完成时尝试 Apple 的导入功能。

注意：

- Apple 明确说明并非所有旧邮件服务商都受支持；
- 若老板邮局不在支持范围，不要反复试错。

### 方案 B：Mac 邮件 App 手工迁移
若旧邮箱支持 IMAP：

1. Mac Mail 同时添加“旧老板邮局邮箱”和“iCloud Mail”；
2. 等旧邮箱全部同步完成；
3. 按年份/文件夹分批复制到 iCloud Mail；
4. 每批迁移后核对数量；
5. 不要一次移动几万封；
6. 先 Copy，确认无误后再考虑删除旧端。

### 方案 C：本地归档
对财务、合同、客户等关键邮件做额外本地归档。

## 5.3 迁移前必须保存

- [ ] 完整邮箱地址清单
- [ ] 旧邮件已备份
- [ ] 联系人已导出
- [ ] 重要附件已核对
- [ ] 旧邮箱密码/管理员权限仍可使用
- [ ] 当前 DNS 完整截图/导出
- [ ] 老板邮局不做提前停机

---

# 6. 在 iCloud 创建 Custom Email Domain

优先在浏览器使用 iCloud.com 执行。

步骤：

1. 登录域所有者 Apple Account；
2. 进入 iCloud+；
3. 打开 **Custom Email Domain / 自定义电子邮件域**；
4. 选择 **Add a domain you own / 添加你拥有的域**；
5. 选择 **You and Other People / 你和其他人**（如需要多 Apple Account）；
6. 输入：
   - `cnyyl.com`
7. 添加成员；
8. 输入成员的 Apple Account 邮箱/手机号；
9. 将现有 `@cnyyl.com` 地址分配给对应成员；
10. 每个地址完成 Apple 发出的验证邮件；
11. 进入 DNS 更新阶段；
12. **此时先截图 Apple 给出的全部 DNS 要求，不要立即让李欣删除旧 MX。**

需要保存的 Apple 页面截图：

- 域名添加成功页面；
- 成员列表；
- 邮箱地址分配列表；
- Apple 生成的 Personal TXT；
- MX 要求；
- SPF 要求；
- DKIM CNAME；
- Verify 页面。

---

# 7. Apple 官方 DNS 要求

Apple 官方当前要求完成 iCloud Mail 自定义域时更新：

- MX
- TXT
- CNAME

## 7.1 Personal TXT 验证记录

类型：

```text
TXT
```

Host：

```text
@
```

Value：

```text
使用 Apple 设置页面为 cnyyl.com 生成的“个人 TXT 验证值”
```

**不要从网上复制别人的 TXT 值。**

## 7.2 SPF

Apple 官方标准：

```text
v=spf1 include:icloud.com ~all
```

当前 `cnyyl.com` 已有 Chinaemail SPF，因此迁移期间绝对不要创建两个 SPF TXT。

正确原则：

- 一个域名只维护一个 SPF 策略；
- 迁移准备期，如旧老板邮局和 Apple 仍可能同时发信，将 `include:icloud.com` 合并进现有 SPF；
- 最终完全切至 Apple 后，再按 Apple 要求清理旧 Chinaemail SPF 授权；
- 每一次改 SPF 后都做 DNS 查询与发信验证。

## 7.3 MX

Apple 官方 MX：

```text
mx01.mail.icloud.com.
Priority: 10
```

```text
mx02.mail.icloud.com.
Priority: 10
```

最终切换时，旧老板邮局 MX：

```text
mx200n.chinaemail.cn
mxa200n.chinaemail.cn
```

应从主域 MX 中移除，避免邮件随机投递到两套完全不同且互不共享的邮箱系统。

## 7.4 DKIM

类型：

```text
CNAME
```

Host：

```text
sig1._domainkey
```

针对 `cnyyl.com`，Apple 官方模板对应值应为：

```text
sig1.dkim.cnyyl.com.at.icloudmailadmin.com.
```

最终仍以 Apple 设置页面实时显示值为准。

## 7.5 TTL

Apple 官方建议：

```text
3600 秒 / 1 小时
```

---

# 8. DMARC 处理

当前已有 `_dmarc.cnyyl.com`。

迁移原则：

- 不要因为迁 iCloud 就直接删除 DMARC；
- 先确保 Apple SPF/DKIM 验证成功；
- 再切 MX；
- 切换后用 Gmail/Outlook/QQ/163 等外部邮箱测试；
- 查看邮件头确认 SPF、DKIM、DMARC 结果；
- 如果现有 DMARC 是较严格策略（例如 quarantine），更要先完成 DKIM/SPF 验证再正式发信。

---

# 9. 给李欣/域名 DNS 操作人员的执行单

## 第一阶段：准备性 DNS

先要求对方：

1. **不要动网站记录**；
2. 不修改 `www`；
3. 不修改网站 CNAME；
4. 不修改 NS；
5. 不修改与网页有关的 A/AAAA/CNAME；
6. 仅增加 Apple 的：
   - Personal TXT；
   - DKIM CNAME；
7. SPF 按迁移方案合并，不得保留两个独立 SPF TXT；
8. 先不删除旧老板邮局 MX。

目的：先完成 Apple 域验证和签名准备。

## 第二阶段：正式切换窗口

确认：

- Apple 成员已接受；
- 地址已验证；
- 旧邮件已备份；
- iCloud Mail 可登录；
- Apple DNS 页面全部准备完成。

然后让 DNS 操作人员：

1. 删除旧老板邮局的主域 MX；
2. 新增 Apple 两条 MX；
3. 确认 SPF 包含 Apple；
4. 确认 DKIM CNAME；
5. 保持网站 DNS 不动；
6. TTL 3600；
7. 保存修改后的完整 DNS 截图。

---

# 10. 正式切换的推荐时间

建议选择：

- 工作日低峰；
- 尽量避开客户集中询价/合同发送时间；
- 预留 2–4 小时观察窗口；
- 老板邮局继续保持可登录至少 7–14 天；
- 不要在切 MX 同一天注销旧邮箱。

---

# 11. 验收测试

每一个迁移邮箱都执行。

## 11.1 外部 → iCloud 收件

分别使用：

- Gmail
- Outlook
- QQ 邮箱
- 163 邮箱
- 其他企业邮箱

发送至：

```text
xxx@cnyyl.com
```

确认：

- [ ] 5–10 分钟内收到
- [ ] 不进垃圾邮件
- [ ] 发件人正确
- [ ] 附件正常
- [ ] 中文主题正常

## 11.2 iCloud → 外部发信

从：

```text
xxx@cnyyl.com
```

发到上述外部邮箱。

确认：

- [ ] From 显示 `@cnyyl.com`
- [ ] Reply-To 正确
- [ ] SPF PASS
- [ ] DKIM PASS
- [ ] DMARC PASS
- [ ] Gmail/Outlook 不判垃圾
- [ ] 附件正常

## 11.3 Apple 设备

测试：

- iPhone Mail
- Mac Mail
- iCloud.com Mail

确认：

- [ ] 同步正常
- [ ] 发件地址可选择 `@cnyyl.com`
- [ ] 收件同步正常
- [ ] 已读状态正常
- [ ] Sent/已发送同步正常

## 11.4 非 Apple 客户端（如有）

Apple 官方 iCloud Mail 第三方客户端参数当前包括：

IMAP：

```text
imap.mail.me.com
Port 993
SSL/TLS
```

SMTP：

```text
smtp.mail.me.com
Port 587
SSL/TLS/STARTTLS
需要认证
```

第三方客户端应使用 Apple 的 App-Specific Password，而不是直接使用 Apple Account 主密码。

---

# 12. Catch-all 是否开启

Apple 支持域所有者启用“Allow All / 接收所有未建立地址的邮件”。

例如：

即使没有创建：

```text
abc@cnyyl.com
```

也可以由域所有者接收该域未被占用地址的邮件。

注意：

- Catch-all 只进入域所有者邮箱；
- 已分配给其他成员的地址不会被域所有者截获；
- 被删除的地址行为需要单独核实；
- 企业使用 Catch-all 可能增加垃圾邮件。

默认建议：

```text
先关闭 Catch-all
```

稳定后再按需求开启。

---

# 13. 回滚方案

任何以下情况出现时立即考虑回滚：

- Apple Verify 长时间失败；
- 大量外部邮箱收不到；
- 发 Gmail/Outlook 大面积进垃圾箱；
- 关键地址映射错误；
- 用户无法登录；
- 旧邮件未备份；
- DKIM/SPF/DMARC 异常。

回滚步骤：

1. 不删除 iCloud 账号和域配置；
2. 将主域 MX 恢复到老板邮局原 MX；
3. 恢复原 SPF；
4. 保持 DMARC；
5. 等 DNS 生效；
6. 用外部邮箱重新测试；
7. 确认老板邮局恢复收件；
8. 查明原因后再安排第二次迁移。

**不要通过删除域来“回滚”。先回滚 DNS。**

---

# 14. 企业治理风险

iCloud Custom Email Domain 本质上更接近“个人/家庭/小团队自定义域邮件”，不是完整企业邮件管理系统。

特别注意：

## 14.1 离职问题

Apple 官方说明：

- 域所有者可以移除成员；
- 被移除成员的域邮箱地址会被删除；
- 但该成员在被移除前已经收发的邮件，仍保留在其自己的 iCloud Mail 账户中。

这意味着：

> 如果将公司员工的工作邮件绑定到员工私人 Apple Account，公司无法像传统企业邮箱那样完全控制其历史邮件数据。

因此：

- 股东/长期固定人员：可接受；
- 普通员工：谨慎；
- 财务/法务/HR/客户核心邮箱：优先考虑企业邮箱。

## 14.2 共享邮箱问题

如果 `sales@`、`service@`、`finance@` 需要多人共同访问，不要强行用“共享 Apple Account 密码”。

不建议：

- 多人共享一个 Apple Account 密码；
- 多人共享 2FA；
- 公司核心邮箱绑定某个人私人 Apple Account 后共同使用。

这会导致权限、2FA、离职、隐私、审计问题。

---

# 15. 针对澳瑞虹的上线决策表

## 可以迁 iCloud 的情况

满足全部：

- [ ] 实际使用人 ≤ 6
- [ ] 每人需要地址 ≤ 3
- [ ] 没有必须多人共同管理的 Shared Mailbox
- [ ] 主要由老板/股东/固定人员使用
- [ ] 接受每个人邮件保存在各自 iCloud
- [ ] 不需要复杂企业审计和邮件保留
- [ ] DNS 可以按要求修改

→ 可以执行 iCloud 迁移。

## 不建议迁 iCloud 的情况

任一项成立：

- [ ] 10 个员工都需要独立邮箱
- [ ] `sales@` 等必须多人共用
- [ ] 财务/法务邮件必须公司集中留存
- [ ] 普通员工流动较大
- [ ] 需要管理员查看/接管员工历史邮箱
- [ ] 需要邮件归档/审计/合规

→ 保留老板邮局或换企业邮箱 SaaS。

---

# 16. 实际执行任务清单

## Phase 0：盘点

- [ ] 导出现有全部邮箱地址
- [ ] 确认实际活跃人数
- [ ] 标记多人共享邮箱
- [ ] 确认每人 Apple Account
- [ ] 确认 2FA
- [ ] 确认主要 `@icloud.com`
- [ ] 确认 iCloud+ 条件
- [ ] 备份旧邮件
- [ ] 保存完整 DNS

## Phase 1：Apple 预配置

- [ ] 域所有者进入 iCloud+
- [ ] Add a domain you own
- [ ] 输入 cnyyl.com
- [ ] 添加成员
- [ ] 分配邮箱地址
- [ ] 完成旧地址验证
- [ ] 保存 Apple DNS 要求截图

## Phase 2：DNS 预处理

- [ ] 添加 Personal TXT
- [ ] 添加 DKIM CNAME
- [ ] 合并 SPF
- [ ] 不改网站 DNS
- [ ] 不改 NS
- [ ] 暂不删旧 MX

## Phase 3：切换

- [ ] 切换 MX 到 Apple
- [ ] 完成 Apple Verify
- [ ] 确认域状态正常
- [ ] 收信测试
- [ ] 发信测试
- [ ] SPF PASS
- [ ] DKIM PASS
- [ ] DMARC PASS

## Phase 4：观察

- [ ] 观察 24 小时
- [ ] 观察 72 小时
- [ ] 检查退信
- [ ] 检查垃圾邮件
- [ ] 检查漏信
- [ ] 老板邮局继续保留

## Phase 5：旧邮件迁移

- [ ] Apple Import Mail 尝试
- [ ] 不支持则 Mac Mail 手工复制
- [ ] 核对文件夹
- [ ] 核对邮件数量
- [ ] 核对附件

## Phase 6：稳定后清理

- [ ] 清理旧 Chinaemail SPF 授权
- [ ] 确认旧 MX 已删除
- [ ] 保留历史 DNS 记录截图
- [ ] 暂不注销老板邮局，至少观察 7–14 天
- [ ] 网站项目另开第二阶段

---

# 17. PPT 制作要求

Agent 基于本 Markdown 生成一套 **16:9 企业技术方案 PPT**。

建议 14–16 页：

1. 封面：cnyyl.com 邮箱迁移至 iCloud+ 技术方案
2. 当前系统与目标
3. 为什么先邮箱、后网站
4. Apple Custom Email Domain 能力边界
5. “多个 Apple Account”正确理解
6. iCloud 适配性 / STOP 条件
7. 当前老板邮局 DNS 架构
8. 新架构图
9. Apple Account 准备
10. 邮箱地址映射
11. DNS 修改表
12. 迁移时序图
13. 测试与验收
14. 回滚方案
15. 企业治理风险
16. 最终执行清单

视觉要求：

- 苹果式极简技术风格；
- 白色/浅灰背景；
- 蓝灰色信息层级；
- 不做花哨营销；
- 架构图优先；
- DNS 记录使用等宽字体代码块；
- 风险页使用醒目警示标签；
- 所有截图位置用占位框并标明“待执行截图”；
- 不伪造 Apple 后台截图；
- 不伪造 DNS 生效结果；
- 不伪造测试 PASS。

---

# 18. PDF 制作要求

输出与 PPT 同内容的技术执行 PDF，适合作为内部 SOP。

PDF 必须包含：

- 项目范围；
- 可行性门槛；
- Apple 限制；
- DNS 变更明细；
- 逐步 SOP；
- 回滚方案；
- 验收表；
- 风险；
- 最终 Checklist；
- 资料留档区。

禁止将“Apple 域共享”误写成“多个 Apple Account 可以共享同一个邮箱收件箱”。

---

# 19. Agent 执行提示词

你是企业 IT 邮件迁移与 DNS 运维 Agent。

目标：基于本文件，为“澳瑞虹国际贸易（上海）有限公司”的 `cnyyl.com` 域执行 **老板邮局 → Apple iCloud+ Custom Email Domain** 的迁移设计，并制作 PPT 与 PDF。

严格规则：

1. 第一阶段仅做邮箱，网站全部保持原状。
2. 不修改 `www`、网页 CNAME、网站 A/AAAA、NS。
3. 任何 MX 删除前必须完成旧邮箱盘点与备份。
4. 任何 DNS 值必须以 Apple 当次 Custom Email Domain 页面显示内容和 Apple 官方文档为准。
5. Personal TXT 绝对不可猜。
6. 不允许存在两条独立 SPF 记录。
7. DKIM 必须验证。
8. DMARC 不得随意删除。
9. 必须提供回滚方案。
10. 必须明确：域最多共享给 5 个其他人；总计最多 6 个成员；每人每域最多 3 个活动地址。
11. 必须明确：一个具体地址不是 Apple 原生 Shared Mailbox，不能把“域共享”写成“多人共同访问同一个收件箱”。
12. 如果发现实际独立邮箱用户 > 6，立即标记为 STOP，不执行切换。
13. 如果发现关键邮箱必须多人共享，立即标记为 STOP/需替代方案。
14. 不得为了完成任务而伪造已成功迁移、已验证、已测试结果。
15. 所有执行结果必须有截图或 DNS 查询证据。

最终输出：

- `cnyyl_iCloud_Mail_Migration_SOP.pdf`
- `cnyyl_iCloud_Mail_Migration_Presentation.pptx`
- `cnyyl_iCloud_Mail_Migration_Execution_Log.md`

---

# 20. 官方依据（供 Agent 核验）

制作 PPT/PDF 时优先核验 Apple 官方支持文档：

- Use Custom Email Domain with iCloud Mail
- Add an email domain you already own to iCloud Mail on iCloud.com
- Add or remove people sharing a custom email domain on iCloud.com
- Transfer custom email domain addresses on iCloud.com
- Set up an existing domain with iCloud Mail
- Import existing email messages to iCloud Mail from your custom email domain
- Allow all incoming emails to a custom email domain on iCloud.com
- iCloud Mail server settings for other email client apps

任何界面、人数限制、DNS 参数若与本文件不一致，以 **执行当日 Apple 官方页面** 为准。

---

# 21. 本阶段成功标准

只有同时满足以下条件才判定“邮箱已打通”：

- [ ] `cnyyl.com` 已在 iCloud Custom Email Domain 验证成功
- [ ] 所有计划成员正常加入
- [ ] 所有计划邮箱地址分配正确
- [ ] Apple MX 生效
- [ ] Apple SPF 生效
- [ ] Apple DKIM 生效
- [ ] DMARC 验证通过
- [ ] 外部 → cnyyl.com 收件正常
- [ ] cnyyl.com → 外部发信正常
- [ ] Gmail 测试正常
- [ ] Outlook 测试正常
- [ ] QQ/163 至少一项测试正常
- [ ] iPhone Mail 正常
- [ ] Mac Mail 或 iCloud.com 正常
- [ ] 旧老板邮局邮件有备份
- [ ] 回滚记录完整
- [ ] 网站完全未受影响

完成后，再启动第二阶段：

> `cnyyl.com` 网站迁移 / Cloudflare / Pages / Tunnel / 家庭服务器方案。
