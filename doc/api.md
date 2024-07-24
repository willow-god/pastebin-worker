# API 文档

## GET `/`

返回主页。

## **GET** `/<name>[.<ext>]` 或 `/<name>/<filename>[.<ext>]`

获取名称为 `<name>` 的粘贴内容。默认情况下，返回粘贴的原始内容。

`Content-Type` 头被设置为 `text/plain;charset=UTF-8`。如果提供了 `<ext>`，则工作器会根据 `<ext>` 推断 MIME 类型并更改 `Content-Type`。该方法接受以下查询字符串参数：

- `?a=`：可选。设置 `Content-Disposition` 为 `attachment`，如果存在该参数。

- `?lang=<lang>`：可选。返回带有语法高亮的网页，由 prism.js 提供支持。

- `?mime=<mime>`：可选。指定 MIME 类型，抑制 `<ext>` 的影响。如果指定了 `lang`，则没有效果（此时 MIME 类型始终为 `text/html`）。

示例：`GET /abcd?lang=js`，`GET /abcd?mime=application/json`。

如果发生错误，工作器将返回不同于 `200` 的状态码：

- `404`：未找到给定名称的粘贴。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

使用示例：

```shell
$ curl https://share.lius.me/i-p-
https://web.archive.org/web/20210328091143/https://mp.weixin.qq.com/s/5phCQP7i-JpSvzPEMGk56Q

$ curl https://share.lius.me/~panty.jpg | feh -

$ firefox 'https://share.lius.me/kf7z?lang=nix'

$ curl 'https://share.lius.me/~panty.jpg?mime=image/png' -w '%{content_type}' -o /dev/null -sS
image/png;charset=UTF-8
```

## GET `/<name>:<passwd>`

返回名称为 `<name>` 且密码为 `<passwd>` 的粘贴的编辑页面。

如果发生错误，工作器将返回不同于 `200` 的状态码：

- `404`：未找到给定名称的粘贴。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

## GET `/u/<name>`

重定向到名称为 `<name>` 的粘贴中记录的 URL。

如果发生错误，工作器将返回不同于 `302` 的状态码：

- `404`：未找到给定名称的粘贴。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

使用示例：

```shell
$ firefox https://share.lius.me/u/i-p-

$ curl -L https://share.lius.me/u/i-p-
```

## GET `/a/<name>`

返回从名称为 `<name>` 的粘贴中存储的 Markdown 文件转换而来的 HTML。Markdown 转换遵循 GitHub 风格的 Markdown（GFM）规范，由 [remark-gfm](https://github.com/remarkjs/remark-gfm) 支持。

语法高亮由 [prism.js](https://prismjs.com/) 支持。LaTeX 数学公式由 [MathJax](https://www.mathjax.org) 支持。

如果发生错误，工作器将返回不同于 `200` 的状态码：

- `404`：未找到给定名称的粘贴。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

使用示例：

```md
# Header 1
这是 `test.md` 的内容

<script>
alert("脚本应被删除")
</script>

## Header 2

| abc | defghi |
:-: | -----------:
bar | baz

**粗体**, `等宽字体`, *斜体*, ~~删除线~~, [URL](https://github.com)

- A
 - A1
 - A2
- B

![Panty](https://share.lius.me/~panty.jpg)

1. 第一
2. 第二

> 引用

$$
\int_{-\infty}^{\infty} e^{-x^2} = \sqrt{\pi}
$$

```

```shell
$ curl -Fc=@test.md -Fn=test-md https://share.lius.me

$ firefox https://share.lius.me/a/~test-md
```

## **POST** `/`

上传您的粘贴。它接受表单数据中的参数：

- `c`：必需。您的粘贴内容，可以是文本或二进制文件。它不应大于 10 MB。在获取粘贴时，其 `Content-Disposition` 中的 `filename` 将存在。

- `e`：可选。粘贴的**过期**时间。经过这个时间段后，粘贴将被永久删除。应为整数或浮点数，可以带有可选的单位（默认为秒）。支持的单位有：`s`（秒）、`m`（分钟）、`h`（小时）、`d`（天）、`M`（月）。例如，`360.24` 表示 360.24 秒；`25d` 表示 25 天。由于 Cloudflare KV 存储的限制，时间不应小于 60 秒。

- `s`：可选。允许您修改和删除粘贴的**密码**。如果未指定，工作器将生成一个随机字符串作为密码。

- `n`：可选。粘贴的**自定义名称**。如果未指定，工作器将生成一个随机字符串（默认为 4 个字符）作为名称。在获取自定义名称的粘贴时，您需要在名称前加上 `~`。名称至少为 3 个字符，由字母、数字和字符 `+_-[]*$=@,;/` 组成。

- `p`：可选。**私人模式**标志。如果指定了任何值，粘贴的名称长度为 24 个字符。如果使用了 `n`，则没有效果。

`POST` 方法默认返回 JSON 字符串，例如：

```json
{
    "url": "https://share.lius.me/abcd",
    "admin": "https://share.lius.me/abcd:w2eHqyZGc@CQzWLN=BiJiQxZ",
    "expire": 100,
    "isPrivate": false
}
```

字段解释：

- `url`：字符串。获取粘贴的 URL。当使用自定义名称时，类似于 `https://share.lius.me/~myname`。
- `suggestUrl`：字符串或 null。可能带有文件名或 URL 重定向的 URL。
- `admin`：字符串。更新和删除粘贴的 URL，它是 `url` 后缀加上密码。
- `expire`：字符串或 null。过期时间（秒）。
- `isPrivate`：布尔值。粘贴是否为私人模式。

如果发生错误，工作器将返回不同于 `200` 的状态码：

- `400`：请求格式错误。
- `409`：名称已被使用。
- `413`：内容过大。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

使用示例：

```shell
$ curl -Fc="kawaii" -Fe=300 -Fn=hitagi https://share.lius.me  # 上传一些文本
{
  "url": "https://share.lius.me/~hitagi",
  "admin": "https://share.lius.me/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false,
  "expire": 300
}

$ curl -Fc=@panty.jpg -Fn=panty -Fs=12345678 https://share.lius.me   # 上传文件
{
  "url": "https://share.lius.me/~panty",
  "admin": "https://share.lius.me/~panty:12345678",
  "isPrivate": false
}

# 由于 `curl` 将某些字符视为字段分隔符，如果字段包含分号或逗号，则应使用双引号引住字段
$ curl -Fc=@panty.jpg -Fn='"hi/hello;g,ood"' -Fs=12345678 https://share.lius.me
{
  "url": "https://share.lius.me/~hi/hello;g,ood",
  "admin": "https://share.lius.me/~hi/hello;g,ood:QJhMKh5WR6z36QRAAn5Q5GZh",
  "isPrivate": false
}
```

## **PUT** `/<name>:<passwd>`

更新名称为 `<name>` 且密码为 `<passwd>` 的粘贴。它接受表单数据中的参数：

- `c`：必需。与 `POST` 方法相同。
- `e`：可选。与 `POST` 方法相同。请注意，现在重新计算删除时间。
- `s`：可选。与 `POST` 方法相同。

`PUT` 方法的返回与 `POST` 方法相同。

如果发生错误，工作器将返回不同于 `200` 的状态码：

- `400`：请求格式错误。
- `403`：密码不正确。
- `404`：未找到给定名称的粘贴。
- `413`：

内容过大。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

使用示例：

```shell
$ curl -Fc="arigatou" https://share.lius.me/~hitagi:22@-OJWcTOH2jprTJWYadmDv
{
  "url": "https://share.lius.me/~hitagi",
  "admin": "https://share.lius.me/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "expire": null,
  "isPrivate": false
}

$ curl -Fc=@Stocking.jpg https://share.lius.me/~panty:12345678
{
  "url": "https://share.lius.me/~panty",
  "admin": "https://share.lius.me/~panty:12345678",
  "isPrivate": false
}
```

## **DELETE** `/<name>:<passwd>`

删除名称为 `<name>` 且密码为 `<passwd>` 的粘贴。

如果发生错误，工作器将返回不同于 `200` 的状态码：

- `403`：密码不正确。
- `404`：未找到给定名称的粘贴。
- `500`：意外异常。您可以将此情况报告给作者以进行修复。

使用示例：

```shell
$ curl -X DELETE https://share.lius.me/~panty:12345678
```