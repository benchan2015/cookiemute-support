# CookieMute — Auto Reject Banners

自动识别并点击 Cookie 同意弹窗里的「拒绝」按钮。规则优先、多语言兜底、点击后验收。

全程本地运行：**不发任何网络请求，不收集任何数据**。

## 特性

- **CMP 专用识别**：OneTrust、Cookiebot、Didomi、Usercentrics、Osano、TrustArc、Complianz、Quantcast、Google Funding Choices、CookieYes、iubenda
- **多语言兜底**：30+ 种语言的「拒绝全部 / 仅必要 / 不同意」文案，覆盖中文、日韩、阿拉伯语、希伯来语、印地语及欧洲主要语言
- **按钮打分**：强/弱拒绝词、按钮行位置、视觉次要度综合评分，低于阈值不点击
- **点击后验收**：轮询确认弹窗是否真的消失，没消失自动换下一个候选，避免「点了但没效果」
- **shadow DOM / iframe**：递归遍历 open shadow root；`all_frames` 覆盖 iframe 内弹窗
- **持续监听**：MutationObserver 覆盖延迟弹出和 SPA 路由切换
- **兜底隐藏（可选）**：找不到拒绝按钮时隐藏弹窗，仅对确认含 Cookie 特征词的弹窗生效，不会误藏其他对话框
- **手动触发 / 白名单 / 实时开关**：白名单支持子域名，加入 `example.com` 同时覆盖其子域

## 隐私

- 无网络请求、无追踪、无分析
- API 权限仅 `storage` + `activeTab`；页面访问只用于识别 Cookie 弹窗
- 所有数据（开关、白名单、计数）只存在浏览器本地的 `chrome.storage.local`

## 安装（开发者模式）

1. 克隆仓库：

   ```bash
   git clone https://github.com/benchan2015/cookiemute.git
   ```

2. 打开 `chrome://extensions/`，开启「开发者模式」
3. 点击「加载已解压的扩展程序」，选择仓库目录

## 使用

点击工具栏图标：

- **Auto Reject**：自动拒绝开关
- **Hide banner if reject button missing**：兜底隐藏开关
- **Manual Reject Now**：对当前页面立即执行一次
- **Add current site to whitelist**：当前站点加入白名单（支持子域名）

## 工作原理

1. **CMP 选择器**：先按服务商特征确认在场，直接点击它生成的拒绝按钮
2. **按钮打分**：没有 CMP 规则时，在候选弹窗内按文本和位置给按钮打分，取最高分且超过阈值的
3. **点击验收**：点击后轮询确认弹窗消失；失败则换下一个候选
4. **兜底隐藏**：没有任何可点按钮时，按用户设置隐藏确认过的 Cookie 弹窗

## 文件结构

| 文件 | 作用 |
|---|---|
| `manifest.json` | 扩展配置（MV3） |
| `rules.js` | 识别规则：CMP 选择器 + 多语言词表，改规则只需动这里 |
| `content.js` | 调度逻辑：定位弹窗、打分、点击、验收、监听 |
| `popup.html` / `popup.js` | 工具栏弹窗界面与交互 |
| `icons/` | 扩展图标 |

## 测试

需要 Node 18+。浏览器用例会自动查找本机 Chrome/Edge；找不到时跳过，可用 `BROWSER_PATH` 指定路径。

```bash
npm test              # 规则单测 + 浏览器集成测试
npm run test:unit     # 只跑规则单测，不需要浏览器
npm run test:browser  # 只跑浏览器用例
```

每次运行结束会在 `test-results/` 生成 `report.html`（浏览器打开即可看）和 `report.json`。

覆盖内容：

- `rules.js` 的结构与多语言词表（强/弱拒绝、同意否决、危险操作、关键词）
- 21 个浏览器用例：CMP 按钮、Didomi「继续而不接受」、遮挡误判、延迟弹窗、shadow DOM、iframe、兜底隐藏、开关中途切换、白名单添加、popup 状态持久化
- `npm test` 当前全部通过；设置 `REQUIRE_BROWSER=1` 可在找不到浏览器时让测试失败

## 手动测试

自动测试是本地模拟；上架前建议用真实扩展再跑一遍。仓库自带一套手动测试页，页面本身不加载 `content.js`，所以点按钮的只会是你安装的扩展。

```bash
npm run serve
```

浏览器打开 **http://127.0.0.1:8899/**，按首页清单逐项检查：

| 页面 | 检查点 |
|---|---|
| `/keyword.html` | 自动点击 Reject all，横幅消失，状态条显示 `["reject"]` |
| `/onetrust.html` | 走 OneTrust 专用选择器，不靠关键字 |
| `/didomi.html` | 点中「Continuer sans accepter」，不点 Accept |
| `/delayed.html` | 3 秒后出现的横幅能被 MutationObserver 抓住 |
| `/shadow.html` | open shadow root 内的横幅能被点击 |
| `/iframe.html` | iframe 子页面内的横幅能被点击 |
| `/hide.html` | 先打开 popup 的 Hide 开关，无拒绝按钮的横幅应被隐藏 |
| `/no-banner.html` | 订阅框/客服框不应被点击或隐藏 |

Popup 与设置项：

1. 关闭 Auto Reject，刷新 `/keyword.html`，横幅应保留；再点 Manual Reject Now，横幅应被拒绝。
2. 对 `127.0.0.1` 点 Add current site to whitelist，刷新后横幅保留；打开 `http://localhost:8899/keyword.html`，应恢复自动拒绝（hostname 不同）。
3. 每次成功拒绝后，popup 里的 Total rejected 应 +1。

真实网站抽查（是否弹出取决于地区、语言和是否已有同意记录，建议用无痕窗口并先清除站点数据）：

- CMP 厂商站：<https://www.onetrust.com/>、<https://www.cookiebot.com/>、<https://www.didomi.io/>、<https://usercentrics.com/>、<https://www.osano.com/>、<https://trustarc.com/>、<https://complianz.io/>、<https://www.cookieyes.com/>、<https://www.iubenda.com/>、<https://www.quantcast.com/>
- 常见内容站：<https://www.spiegel.de/>、<https://www.lemonde.fr/>、<https://www.bbc.com/>、<https://www.heise.de/>、<https://www.zalando.de/>、<https://www.booking.com/>

真实网站的失败样本最有价值：把页面 URL、地区/语言、截图和 `chrome://extensions` 里的错误信息一起记录，方便补规则。

## 打包上架

```bash
npm run build
```

生成 `dist/CookieMute-<version>.zip`，只包含运行时文件：

```text
manifest.json
rules.js
content.js
popup.html
popup.js
icons/icon16.png
icons/icon32.png
icons/icon48.png
icons/icon128.png
```

`tests/`、`scripts/`、`README.md`、`package.json` 等不会进入 ZIP；`manifest.json` 位于压缩包根目录。打包前会校验 manifest 与 `package.json` 的版本号一致，并检查 manifest 引用到的运行文件是否都在打包白名单里。

## 商店素材

```bash
npm run assets
```

生成 `store-assets/` 下的 1280×800 商店截图和可选推广图。脚本会用本机 Chrome/Edge
加载扩展，实际跑一遍 Didomi 弹窗拒绝，再截取 popup 和前后对比。

Chrome/Edge 137+ 已忽略 `--load-extension`，需要指定 Chrome for Testing 136 或更早版本：

```bash
BROWSER_PATH="C:/path/to/chrome-136/chrome-win64/chrome.exe" npm run assets
```

素材清单和重新生成说明见 [store-assets/README.md](store-assets/README.md)。

## 隐私政策

见 [PRIVACY.md](PRIVACY.md)。公开副本与支持页：<https://github.com/benchan2015/cookiemute-support>。上架资料与文案见 [STORE.md](STORE.md)。

## 已知限制

- closed shadow root 内的弹窗无法访问
- 跨域 iframe 读不到顶层 URL，白名单在 frame 内按 frame 自身域名判断
- 页面加载后持续监听 30 秒；更晚出现的弹窗需要手动触发
