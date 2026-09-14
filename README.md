Tor 通过隐藏您的互联网地址与您使用的服务之间的连接来保护您的网络隐私。我们认为 Tor 是相对安全的，但请确保您阅读了说明并正确配置。

## 在线说明文档与配置生成

本仓库基于 Tor 0.4.9.11 的定制静态编译版，支持 SOCKS5 用户名指定出口国家（SocksExitByUser）。

- 在线说明文档（运行教程）：<https://lmq8267.github.io/tor-0.4.9.11>
- 在线配置生成器：[https://lmq8267.github.io/tor-0.4.9.11/在线配置文件生成.html](https://lmq8267.github.io/tor-0.4.9.11/%E5%9C%A8%E7%BA%BF%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%94%9F%E6%88%90.html)

## 构建

从源代码构建 Tor：

```
./configure
make
make install
```

从刚克隆的 git 仓库构建 Tor：

```
./autogen.sh
./configure
make
make install
```

## 发布版本

压缩包、校验和和签名可在此处找到：https://dist.torproject.org

- 校验和：`<tarball-name>.sha256sum`
- 签名：`<tarball-name>.sha256sum.asc`

### 发布计划

您可以在以下地址找到我们的发布计划：

- https://gitlab.torproject.org/tpo/core/team/-/wikis/NetworkTeam/CoreTorReleases

### 可以签署发布的密钥

以下密钥是本仓库的维护者。其中一个或多个密钥可以签署发布版本，请不要期望所有密钥都会签署：

- Alexander Færøy:
    [514102454D0A87DB0767A1EBBE6A0531C18A9179](https://keys.openpgp.org/vks/v1/by-fingerprint/1C1BC007A9F607AA8152C040BEA7B180B1491921)
- David Goulet:
    [B74417EDDF22AC9F9E90F49142E86A2A11F48D36](https://keys.openpgp.org/vks/v1/by-fingerprint/B74417EDDF22AC9F9E90F49142E86A2A11F48D36)
- Nick Mathewson:
    [2133BC600AB133E1D826D173FE43009C4607B1FB](https://keys.openpgp.org/vks/v1/by-fingerprint/2133BC600AB133E1D826D173FE43009C4607B1FB)

## 开发

请参阅 [doc/HACKING/](./doc/HACKING) 中的开发文档。

## 资源

主页：

- https://www.torproject.org/

下载新版本：

- https://www.torproject.org/download/tor

如何验证 Tor 源代码：

- https://support.torproject.org/little-t-tor/

文档和常见问题：

- https://support.torproject.org/

如何运行 Tor 中继：

- https://community.torproject.org/relay/
