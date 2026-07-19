此目录包含随 Tor 一起发布的配置文件。包括：

geoip
geoip6

    IPv4 和 IPv6 的 GeoIP 文件

torrc.minimal, torrc.sample:

    由 autoconf 根据 torrc.minimal.in 和 torrc.sample.in 生成。

torrc.minimal.in:

    一个非常小的 torrc，适合作为默认配置安装到
    /etc/tor/torrc。

    我们尽量少修改 torrc.minimal.in，因为修改它会让许多软件包的
    用户不得不重新构建他们的 torrc 文件。


torrc.minimal.in-staging

    这里用于暂存一段时间内对 torrc.minimal.in 的修改。这样，当某个
    变更足够大、值得生成新的 torrc.minimal.in 时，我们可以一次性
    复制所有其他修改。极简配置文件示例

torrc.sample.in:

    一个详细、说明充分、功能完整的 torrc。适合用来让人们了解如何
    设置各种选项，包括那些大多数人不应该随意调整的选项。完整配置文件示例


==============================

关于 geoip 格式：

我们的 geoip 文件是面向行的。任何空行，或以 # 开头的行，都会被忽略。

所有其他行都由三个逗号分隔的值组成：START,END,CC。对于 geoip 文件，
START 和 END 是以 32 位整数表示的 IPv4 地址（例如用 3325256709 表示
198.51.100.5）。对于 geoip6 文件，START 和 END 是不带方括号的 IPv6
地址。在这两种情况下，CC 都是两个字符的国家/地区代码。

一行 START,END,CC 的语义是：START 和 END 之间的所有地址（包含 START
和 END）都应映射到国家/地区代码 CC。

我们保证这些文件中的所有条目互不重叠，也就是说，不存在被多于一行匹配
的地址。我们还保证这些文件中的所有条目都按地址的数值升序排序。

因此，这里一种有效的查找算法是对文件中的所有条目执行二分查找。

请注意，这些数据库中确实存在“空隙”：并非每个可能的地址都能映射到国家/
地区代码。在这些情况下，Tor 会将国家/地区报告为 ??。
