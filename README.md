# mesh-peers

wfmon 的 EasyTier **引导节点（bootstrap peers）公开发现清单**。

wfmon 的 client / server 启动与运行期间会从多个公开镜像拉取本仓库的
[`peers.json`](peers.json)，校验签名后用其中的 peers 引导接入 EasyTier 虚拟网络。
这样当公共节点地址变更时，只需更新本仓库、签名、推送，全网 wfmon 即可自动跟随，
**无需重新编译或逐台修改**。

## 文件说明

| 文件 | 说明 |
|---|---|
| [`peers.json`](peers.json) | 当前引导节点清单（只含端点，**不含网络密钥/token**） |
| [`peers.json.sig`](peers.json.sig) | 对 `peers.json` 原文的 Ed25519 签名（base64，单行） |

`peers.json` 字段：

```json
{
  "version": 1,
  "updated_at": "2026-09-23T09:36:00+08:00",
  "network": "andaoai-network",
  "peers": ["tcp://225284.xyz:11010"]
}
```

- `version`：单调递增，每次修改 +1。
- `peers`：引导节点 URI 列表。

## 拉取地址（多镜像）

wfmon 会依次尝试以下来源（任一可达即可）：

- `https://raw.githubusercontent.com/andaoai/mesh-peers/main/peers.json`
- `https://cdn.jsdelivr.net/gh/andaoai/mesh-peers@main/peers.json`
- `https://ghproxy.net/https://raw.githubusercontent.com/andaoai/mesh-peers/main/peers.json`
- `https://gh-proxy.com/https://raw.githubusercontent.com/andaoai/mesh-peers/main/peers.json`

## 更新清单（维护者）

修改 `peers.json`（`version` +1）后，用离线保存的 Ed25519 私钥重新签名并推送：

```bash
go run /path/to/sign-peers <private-key-file> peers.json
git add peers.json peers.json.sig
git commit -m "peers: 更新引导节点"
git push
```

> ⚠️ 签名**私钥离线保管、绝不进本仓库**。wfmon 二进制内置对应的公钥，未签名或
> 验签失败的清单会被拒绝（fail-closed）。
