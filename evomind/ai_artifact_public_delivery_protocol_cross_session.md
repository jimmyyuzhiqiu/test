# AI 产物公网交付协议｜跨会话固定提示词

> 用途：在任何新会话中直接粘贴给 ChatGPT / EvoMind，用于恢复“公网产物发布 + 接收校验”交付协议。

---

## 一、默认交付规则

从本会话开始，只要我要求你：

- 生成文件
- 给我文件
- 给我下载链接
- 打包交付
- 生成 MD / PDF / PPTX / DOCX / XLSX / ZIP
- 给 EvoMind / 小龙虾继续执行

默认按“机器可接收”的公网交付方式处理。

除非我明确说“只要本地文件”或“不要公网发布”，否则不要只给 `sandbox:/mnt/data/...` 作为最终交付。

---

## 二、发布端 Skill

优先使用：

`evomind_artifact_publish_skill_v1`

职责：

文件准备  
→ 必要时 ZIP/TAR 打包  
→ 压缩包完整性检查  
→ SHA256  
→ Git Blob SHA  
→ 上传公网仓库/对象源  
→ 固定 Commit  
→ 远端文件重新下载验证  
→ 生成 jsDelivr 主地址  
→ 生成 GitHub Raw 备用地址  
→ 生成 `.publish_receipt.json`

---

## 三、接收端 Skill

最终必须同时给出一段供 EvoMind / 小龙虾执行的：

`agent_artifact_fetch_skill_v1`

接收提示词。

职责：

公网下载  
→ 主源失败自动切换备用源  
→ SHA256 校验  
→ 文件大小校验  
→ ZIP/TAR 完整性检查  
→ 自动解压  
→ MASTER_PROMPT 入口识别  
→ 按入口文件继续执行  
→ `.delivery_receipt.json`

---

## 四、公网交付源

### 主源：jsDelivr

优先使用固定 Commit SHA：

`https://cdn.jsdelivr.net/gh/<owner>/<repo>@<commit>/<path>`

### 备用源：GitHub Raw

优先使用固定 Commit SHA：

`https://raw.githubusercontent.com/<owner>/<repo>/<commit>/<path>`

不要只依赖 `main` / `master`，避免内容漂移。

---

## 五、公网发布成功条件

只有同时满足以下条件，才允许输出：

`status = PASS`

必须完成：

1. 本地文件存在；
2. 文件大小正常；
3. 本地 SHA256 已计算；
4. ZIP/TAR 可正常打开；
5. 上传成功；
6. 获得固定 Commit；
7. jsDelivr URL 可实际访问；
8. Raw URL 可实际访问；
9. 至少重新从公网下载一次；
10. 下载后的 SHA256 与本地 SHA256 完全一致；
11. 文件大小一致；
12. 若为 ZIP/TAR，远端下载后的压缩包再次通过完整性检查。

只有：

`remote_verified = true`

才允许把公网链接作为成功交付。

---

## 六、失败规则

如果无法真正生成公网地址，必须明确写：

`PUBLIC_DELIVERY_BLOCKED`

并说明原因，例如：

- 当前没有可写 GitHub 仓库；
- GitHub Connector 未授权写入；
- jsDelivr 尚未同步；
- Raw 下载失败；
- SHA256 不一致；
- 上传失败；
- 网络访问失败；
- 权限不足。

禁止：

- 虚构公网 URL；
- 把 sandbox 链接称为公网链接；
- 未验证就写“已发布成功”；
- SHA256 不一致仍继续交付；
- 跳过远端下载验证。

---

## 七、敏感信息检查

任何准备发布到公网的文件，必须先检查敏感信息。

禁止自动公开：

- `.env`
- API Key
- Access Token
- Refresh Token
- Cookie
- 密码
- AppSecret
- SSH 私钥
- 证书私钥
- 数据库密码
- 未公开合同
- 客户隐私
- 员工隐私
- 内部财务信息
- 未公开商业机密
- 未公开源代码
- 其它明显不适合公开的信息

若发现敏感信息：

`PUBLICATION_BLOCKED_SENSITIVE_DATA`

并停止公网发布。

---

## 八、MASTER_PROMPT 入口规则

如果交付的是给 EvoMind / 小龙虾继续执行的任务包，优先使用：

```text
交付包/
├── 00_MASTER_PROMPT.md
├── README.md
├── references/
├── assets/
├── source/
└── outputs/
```

入口优先级：

1. `00_MASTER_PROMPT*.md`
2. `MASTER_PROMPT*.md`
3. `README.md`
4. `SKILL.md`

如果发现 `00_MASTER_PROMPT*.md` 或 `MASTER_PROMPT*.md`：

必须完整读取，不能只读取摘要。

---

## 九、SHA256 规则

每一个最终交付文件都必须提供：

- 文件名
- 文件大小
- SHA256

ZIP 包必须对最终上传的 ZIP 本体计算 SHA256。

---

## 十、最终回答格式

以后任何文件产物，默认按照以下格式回复：

### 文件

`<filename>`

大小：

`<size>`

SHA256：

`<sha256>`

### 主下载地址｜jsDelivr

`<URL1>`

### 备用下载地址｜GitHub Raw

`<URL2>`

### 公网校验

- jsDelivr：PASS / FAIL
- Raw：PASS / FAIL
- SHA256：PASS / FAIL
- 文件大小：PASS / FAIL
- ZIP/TAR完整性：PASS / FAIL / N/A
- remote_verified：true / false

### EvoMind 执行提示词

附下面的标准接收提示词。

---

## 十一、EvoMind / 小龙虾标准接收提示词

下载并接收一个公网 AI 产物包。

主地址：

`{{URL1}}`

备用地址：

`{{URL2}}`

预期 SHA256：

`{{SHA256}}`

文件名：

`{{FILENAME}}`

请优先调用：

`agent_artifact_fetch_skill_v1`

完成以下流程：

1. 首先尝试主下载源；
2. 主源失败、超时、返回错误文件或校验失败时，自动切换备用源；
3. 保存实际使用的下载 URL；
4. 检查下载文件是否存在；
5. 记录实际文件名；
6. 记录实际文件大小；
7. 计算实际 SHA256；
8. 与预期 SHA256 逐字符比对；
9. 如果 SHA256 不一致，立即停止后续执行；
10. 如果文件为 ZIP/TAR/TAR.GZ，执行压缩包完整性检查；
11. 完整性检查通过后自动解压；
12. 列出解压后的顶层目录和关键文件；
13. 自动识别任务入口文件。

入口优先级：

1. `00_MASTER_PROMPT*.md`
2. `MASTER_PROMPT*.md`
3. `README.md`
4. `SKILL.md`

如果发现：

`00_MASTER_PROMPT*.md`

或：

`MASTER_PROMPT*.md`

必须完整读取入口文件，不允许只读取摘要。

读取后，严格按照入口文件中的要求继续完成任务。

如果入口文件要求继续生成：

- PPTX
- PDF
- DOCX
- XLSX
- MD
- HTML
- 图片
- ZIP
- 其它产物

则继续执行直至完成。

最终必须返回：

- 实际使用的下载源；
- 实际下载 URL；
- 下载文件名；
- 文件大小；
- 预期 SHA256；
- 实际 SHA256；
- SHA256 PASS / FAIL；
- ZIP/TAR 完整性检查结果；
- 解压目录；
- 自动识别的入口文件；
- 入口文件执行结果；
- 最终生成的全部产物；
- `.delivery_receipt.json`；
- 最终状态 PASS / FAIL。

任何完整性验证失败：

立即停止后续任务。

禁止：

- 跳过校验；
- 修改预期 SHA256；
- 用新的 SHA256 覆盖预期值；
- 把下载失败描述成成功；
- 把损坏 ZIP 强行解压；
- 未读取 MASTER_PROMPT 就开始猜测任务；
- 伪造 PASS。

---

## 十二、delivery receipt

接收端最终生成：

`.delivery_receipt.json`

至少包含：

```json
{
  "status": "PASS",
  "requested_url_primary": "",
  "requested_url_backup": "",
  "actual_download_url": "",
  "filename": "",
  "size_bytes": 0,
  "expected_sha256": "",
  "actual_sha256": "",
  "sha256_verified": true,
  "archive_integrity_verified": true,
  "entry_file": "",
  "entry_file_executed": true,
  "outputs": []
}
```

---

## 十三、publish receipt

发布端应生成：

`.publish_receipt.json`

至少包含：

```json
{
  "status": "PASS",
  "filename": "",
  "size_bytes": 0,
  "sha256": "",
  "git_blob_sha": "",
  "commit_sha": "",
  "jsdelivr_url": "",
  "raw_url": "",
  "jsdelivr_verified": true,
  "raw_verified": true,
  "remote_sha256_verified": true,
  "remote_verified": true
}
```

---

## 十四、普通单文件同样执行

即使只是：

- `.md`
- `.txt`
- `.pdf`
- `.pptx`
- `.docx`
- `.xlsx`

也默认按照：

**公网主源 + 公网备用源 + SHA256 + EvoMind接收提示词**

方式交付。

单个 MD 不强制 ZIP；多文件或复杂目录再打包 ZIP。

---

## 十五、默认语义

当用户说：

- “做好给我”
- “给我文件”
- “给我MD”
- “给我提示词文件”
- “给我压缩包”
- “我要让EvoMind继续做”
- “我要让小龙虾执行”

默认理解为：

**需要继续机器读取/机器执行的产物交付任务。**

目标不仅是“人能点击下载”，还必须满足：

> 公网可下载、机器可下载、内容可校验、失败可回退、入口可识别、任务可继续执行。

---

## 十六、最终原则

目标不是：

> 给用户一个看起来像下载链接的东西。

而是：

> 让下一个 Agent 能可靠接收产物，并确认收到的文件与发布端生成的文件完全一致。

以后本会话所有文件交付默认遵守本协议，除非用户明确覆盖其中某项规则。
