# 独立验证页

零依赖的静态页面：拖入凭证 JSON + 原文件，在浏览器里本地校验 SHA-256 与 Merkle Proof，并直接从公共 RPC 读取链上 `anchoredRoots`，**不经过 ProofBase 的后端服务器**。

## 文件

| 文件 | 作用 |
| --- | --- |
| `index.html` | 全部逻辑（HTML + CSS + JS 内联，无外部依赖） |
| `serve.js` | 本地起一个静态服务的极简脚本（可选） |

## 本地运行

```powershell
# 方式一：Node
node C:\Users\KQKYaiz\Desktop\proofbase-mvp\verify\serve.js 19000

# 方式二：Python
cd C:\Users\KQKYaiz\Desktop\proofbase-mvp\verify
python -m http.server 19000
```

然后浏览器打开 <http://localhost:19000>。

> **不要直接双击 `index.html`**：`crypto.subtle` 要求安全上下文，部分浏览器在 `file://` 下会禁用。用 `http://localhost` 或 https 部署即可。

## 部署到公网（给第三方用）

任选其一，都不需要后端：

1. **GitHub Pages**：把这个目录推到仓库，Settings → Pages → 选分支，得到 `https://<user>.github.io/<repo>/`
2. **Vercel / Netlify**：把 `verify` 目录拖进网页，秒出 https 地址
3. **Cloudflare Pages**：同理

部署后把链接发给对方，对方不需要装 App、不需要信任你的服务器。

## 支持的网络

内置公共 RPC（可按需在 `index.html` 的 `RPC` 常量里增删）：

| chainId | 网络 |
| --- | --- |
| 11155111 | Ethereum Sepolia |
| 97 | BNB Smart Chain 测试网 |
| 56 | BNB Smart Chain 主网 |
| 1 | Ethereum 主网 |
| 8453 | Base 主网 |

## 验证逻辑（与合约一致）

1. 文件 → `SHA-256` → 在凭证 `files[]` 里按 `sha256` 匹配
2. 用该条目的 `merkleProof` 逐层计算：每层把两个 32 字节值按字典序排序后拼接，再 `SHA-256`
3. 还原出的 Root 必须等于 `batch.merkleRoot`
4. `eth_call anchoredRoots(root)` 必须返回 `true`
5. 交易收据必须成功，且 `BatchAnchored` 事件里的 Root 与凭证一致
