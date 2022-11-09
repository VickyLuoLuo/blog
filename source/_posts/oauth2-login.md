---
title: 第三方应用/网站使用微信、企业微信、钉钉登录身份验证
catalog: true
tags:
  - OAuth2
categories:
  - work
abbrlink: c179
date: 2022-11-09 15:53:37
subtitle:
header-img:
---

# 微信

https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html

## 网站应用微信登录开发指南

### 准备工作

网站应用微信登录是基于OAuth2.0协议标准构建的微信OAuth2.0授权登录系统。 在进行微信OAuth2.0授权登录接入之前，在微信开放平台注册开发者帐号，并拥有一个已审核通过的网站应用，并获得相应的 AppID 和AppSecret，申请微信登录且通过审核后，可开始接入流程。

### 授权流程说明

微信OAuth2.0授权登录让微信用户使用微信身份安全登录第三方应用或网站，在微信用户授权登录已接入微信OAuth2.0的第三方应用后，第三方可以获取到用户的接口调用凭证（access_token），通过access_token可以进行微信开放平台授权关系接口调用，从而可实现获取微信用户基本开放信息和帮助用户实现基础开放功能等。 微信OAuth2.0授权登录目前支持authorization_code模式，适用于拥有 server 端的应用授权。该模式整体流程为：

```text
1. 第三方发起微信授权登录请求，微信用户允许授权第三方应用后，微信会拉起应用或重定向到第三方网站，并且带上授权临时票据 code 参数；
2. 通过 code 参数加上 AppID 和AppSecret等，通过 API 换取access_token；
3. 通过access_token进行接口调用，获取用户基本数据资源或帮助用户实现基本操作。
```

### 获取access_token时序图：

![image.png](https://s2.loli.net/2022/11/08/gWrBvk14UhRaFc7.png)

**第一步：请求CODE**

第三方使用网站应用授权登录前请注意已获取相应网页授权作用域（scope=snsapi_login），则可以通过在 PC 端打开以下链接： https://open.weixin.qq.com/connect/qrconnect?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect 若提示“该链接无法访问”，请检查参数是否填写错误，如redirect_uri的域名与审核时填写的授权域名不一致或 scope 不为snsapi_login。

**参数说明**

| 参数          | 是否必须 | 说明                                                         |
| ------------- | -------- | ------------------------------------------------------------ |
| appid         | 是       | 应用唯一标识                                                 |
| redirect_uri  | 是       | 请使用 urlEncode 对链接进行处理                              |
| response_type | 是       | 填code                                                       |
| scope         | 是       | 应用授权作用域，拥有多个作用域用逗号（,）分隔，网页应用目前仅填写snsapi_login |
| state         | 否       | 用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止 csrf 攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数加 session 进行校验 |
| lang          | 否       | 界面语言，支持cn（中文简体）与en（英文），默认为cn           |

**返回说明**

用户允许授权后，将会重定向到redirect_uri的网址上，并且带上 code 和state参数

```text
redirect_uri?code=CODE&state=STATE
```

若用户禁止授权，则不会发生重定向。

**请求示例**

登录一号店网站应用 https://test.yhd.com/wechat/login.do 打开后，一号店会生成 state 参数，跳转到https://open.weixin.qq.com/connect/qrconnect?appid=wxbdc5610cc59c1631&redirect_uri=https%3A%2F%2Fpassport.yhd.com%2Fwechat%2Fcallback.do&response_type=code&scope=snsapi_login&state=3d6be0a4035d839573b04816624a415e#wechat_redirect
微信用户使用微信扫描二维码并且确认登录后，PC端会跳转到https://test.yhd.com/wechat/callback.do?code=CODE&state=3d6be0a40sssssxxxxx6624a415e
为了满足网站更定制化的需求，我们还提供了第二种获取 code 的方式，支持网站将微信登录二维码内嵌到自己页面中，用户使用微信扫码授权后通过 JS 将code返回给网站。 JS微信登录主要用途：网站希望用户在网站内就能完成登录，无需跳转到微信域下登录后再返回，提升微信登录的流畅性与成功率。 网站内嵌二维码微信登录 JS 实现办法：

步骤1：在页面中先引入如下 JS 文件（支持https）：

```text
http://res.wx.qq.com/connect/zh_CN/htmledition/js/wxLogin.js
```

步骤2：在需要使用微信登录的地方实例以下 JS 对象：

```text
 var obj = new WxLogin({
 self_redirect:true,
 id:"login_container", 
 appid: "", 
 scope: "", 
 redirect_uri: "",
  state: "",
 style: "",
 href: ""
 });
```

**参数说明**

| 参数          | 是否必须 | 说明                                                         |
| ------------- | -------- | ------------------------------------------------------------ |
| self_redirect | 否       | true：手机点击确认登录后可以在 iframe 内跳转到 redirect_uri，false：手机点击确认登录后可以在 top window 跳转到 redirect_uri。默认为 false。 |
| id            | 是       | 第三方页面显示二维码的容器id                                 |
| appid         | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得           |
| scope         | 是       | 应用授权作用域，拥有多个作用域用逗号（,）分隔，网页应用目前仅填写snsapi_login即可 |
| redirect_uri  | 是       | 重定向地址，需要进行UrlEncode                                |
| state         | 否       | 用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止 csrf 攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数加 session 进行校验 |
| style         | 否       | 提供"black"、"white"可选，默认为黑色文字描述。详见文档底部FAQ |
| href          | 否       | 自定义样式链接，第三方可根据实际需求覆盖默认样式。详见文档底部FAQ |

**第二步：通过 code 获取access_token**

通过 code 获取access_token

```text
https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
```

**参数说明**

| 参数       | 是否必须 | 说明                                                    |
| ---------- | -------- | ------------------------------------------------------- |
| appid      | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得      |
| secret     | 是       | 应用密钥AppSecret，在微信开放平台提交应用审核通过后获得 |
| code       | 是       | 填写第一步获取的 code 参数                              |
| grant_type | 是       | 填authorization_code                                    |

**返回说明**

正确的返回：

```text
{ 
"access_token":"ACCESS_TOKEN", 
"expires_in":7200, 
"refresh_token":"REFRESH_TOKEN",
"openid":"OPENID", 
"scope":"SCOPE",
"unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
```

**参数说明**

| 参数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| access_token  | 接口调用凭证                                                 |
| expires_in    | access_token接口调用凭证超时时间，单位（秒）                 |
| refresh_token | 用户刷新access_token                                         |
| openid        | 授权用户唯一标识                                             |
| scope         | 用户授权的作用域，使用逗号（,）分隔                          |
| unionid       | 当且仅当该网站应用已获得该用户的 userinfo 授权时，才会出现该字段。 |

错误返回样例：

```text
{"errcode":40029,"errmsg":"invalid code"}
```

**刷新access_token有效期**

access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新，access_token刷新结果有两种：

```text
1. 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间；
2. 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。
```

refresh_token拥有较长的有效期（30天），当refresh_token失效的后，需要用户重新授权。

**请求方法**

获取第一步的 code 后，请求以下链接进行refresh_token：

```text
https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
```

**参数说明**

| 参数          | 是否必须 | 说明                                          |
| ------------- | -------- | --------------------------------------------- |
| appid         | 是       | 应用唯一标识                                  |
| grant_type    | 是       | 填refresh_token                               |
| refresh_token | 是       | 填写通过access_token获取到的refresh_token参数 |

**返回说明**

正确的返回：

```text
{ 
"access_token":"ACCESS_TOKEN", 
"expires_in":7200, 
"refresh_token":"REFRESH_TOKEN", 
"openid":"OPENID", 
"scope":"SCOPE" 
}
```

**参数说明**

| 参数          | 说明                                         |
| ------------- | -------------------------------------------- |
| access_token  | 接口调用凭证                                 |
| expires_in    | access_token接口调用凭证超时时间，单位（秒） |
| refresh_token | 用户刷新access_token                         |
| openid        | 授权用户唯一标识                             |
| scope         | 用户授权的作用域，使用逗号（,）分隔          |

错误返回样例：

```text
{"errcode":40030,"errmsg":"invalid refresh_token"}
```

注意：

```text
1、Appsecret 是应用接口使用密钥，泄漏后将可能导致应用数据泄漏、应用的用户数据泄漏等高风险后果；存储在客户端，极有可能被恶意窃取（如反编译获取Appsecret）；
2、access_token 为用户授权第三方应用发起接口调用的凭证（相当于用户登录态），存储在客户端，可能出现恶意获取access_token 后导致的用户数据泄漏、用户微信相关接口功能被恶意发起等行为；
3、refresh_token 为用户授权第三方应用的长效凭证，仅用于刷新access_token，但泄漏后相当于access_token 泄漏，风险同上。

建议将secret、用户数据（如access_token）放在 App 云端服务器，由云端中转接口调用请求。
```

**第三步：通过access_token调用接口**

获取access_token后，进行接口调用，有以下前提：

```text
1. access_token有效且未超时；
2. 微信用户已授权给第三方应用帐号相应接口作用域（scope）。
```

对于接口作用域（scope），能调用的接口有以下：

| 授权作用域（scope） | 接口                      | 接口说明                                               |
| ------------------- | ------------------------- | ------------------------------------------------------ |
| snsapi_base         | /sns/oauth2/access_token  | 通过 code 换取access_token、refresh_token和已授权scope |
| snsapi_base         | /sns/oauth2/refresh_token | 刷新或续期access_token使用                             |
| snsapi_base         | /sns/auth                 | 检查access_token有效性                                 |
| snsapi_userinfo     | /sns/userinfo             | 获取用户个人信息                                       |

其中snsapi_base属于基础接口，若应用已拥有其它 scope 权限，则默认拥有snsapi_base的权限。使用snsapi_base可以让移动端网页授权绕过跳转授权登录页请求用户授权的动作，直接跳转第三方网页带上授权临时票据（code），但会使得用户已授权作用域（scope）仅为snsapi_base，从而导致无法获取到需要用户授权才允许获得的数据和基础功能。 接口调用方法可查阅[《微信授权关系接口调用指南》](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Authorized_Interface_Calling_UnionID.html)

F.A.Q

1. 什么是授权临时票据（code）？ 答：第三方通过 code 进行获取access_token的时候需要用到，code的超时时间为10分钟，一个 code 只能成功换取一次access_token即失效。code的临时性和一次保障了微信授权登录的安全性。第三方可通过使用 https 和state参数，进一步加强自身授权登录的安全性。
2. 什么是授权作用域（scope）？ 答：授权作用域（scope）代表用户授权给第三方的接口权限，第三方应用需要向微信开放平台申请使用相应 scope 的权限后，使用文档所述方式让用户进行授权，经过用户授权，获取到相应access_token后方可对接口进行调用。
3. 网站内嵌二维码微信登录 JS 代码中 style 字段作用？ 答：第三方页面颜色风格可能为浅色调或者深色调，若第三方页面为浅色背景，style字段应提供"black"值（或者不提供，black为默认值），则对应的微信登录文字样式为黑色。相关效果如下：

![image.png](https://s2.loli.net/2022/11/08/AtbkXpTJQwhg9qD.png)

若提供"white"值，则对应的文字描述将显示为白色，适合深色背景。相关效果如下：

![image.png](https://s2.loli.net/2022/11/08/7OidPUhSJeGuqxk.png)

4.网站内嵌二维码微信登录 JS 代码中 href 字段作用？ 答：如果第三方觉得微信团队提供的默认样式与自己的页面样式不匹配，可以自己提供样式文件来覆盖默认样式。举个例子，如第三方觉得默认二维码过大，可以提供相关 css 样式文件，并把链接地址填入 href 字段

```text
.impowerBox .qrcode {width: 200px;}
.impowerBox .title {display: none;}
.impowerBox .info {width: 200px;}
.status_icon {display: none}
.impowerBox .status {text-align: center;} 
```

相关效果如下：

![image.png](https://s2.loli.net/2022/11/08/pCvUA9ewFmOSbTG.png)

## 授权后接口调用（UnionID） 

### **通过 code 获取access_token**

**接口说明**

通过 code 获取access_token的接口。

**请求说明**

```text
http请求方式: GET
https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
```

**参数说明**

| 参数       | 是否必须 | 说明                                                    |
| ---------- | -------- | ------------------------------------------------------- |
| appid      | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得      |
| secret     | 是       | 应用密钥AppSecret，在微信开放平台提交应用审核通过后获得 |
| code       | 是       | 填写第一步获取的 code 参数                              |
| grant_type | 是       | 填authorization_code                                    |

**返回说明**

正确的返回：

```text
{
"access_token":"ACCESS_TOKEN",
"expires_in":7200,
"refresh_token":"REFRESH_TOKEN","openid":"OPENID",
"scope":"SCOPE"
}
```

| 参数          | 说明                                         |
| ------------- | -------------------------------------------- |
| access_token  | 接口调用凭证                                 |
| expires_in    | access_token接口调用凭证超时时间，单位（秒） |
| refresh_token | 用户刷新access_token                         |
| openid        | 授权用户唯一标识                             |
| scope         | 用户授权的作用域，使用逗号（,）分隔          |

错误返回样例：

```text
{
"errcode":40029,"errmsg":"invalid code"
}
```

### 刷新或续期access_token使用

**接口说明**

access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新，access_token刷新结果有两种：

\1. 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间；

\2. 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。

refresh_token拥有较长的有效期（30天），当refresh_token失效的后，需要用户重新授权，所以，请开发者在refresh_token即将过期时（如第29天时），进行定时的自动刷新并保存好它。

**请求方法**

使用/sns/oauth2/access_token接口获取到的refresh_token进行以下接口调用：

```text
http请求方式: GET
https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
```

**参数说明**

| 参数          | 是否必须 | 说明                                          |
| ------------- | -------- | --------------------------------------------- |
| appid         | 是       | 应用唯一标识                                  |
| grant_type    | 是       | 填refresh_token                               |
| refresh_token | 是       | 填写通过access_token获取到的refresh_token参数 |

**返回说明**

正确的返回：

```text
{
"access_token":"ACCESS_TOKEN",
"expires_in":7200,
"refresh_token":"REFRESH_TOKEN",
"openid":"OPENID",
"scope":"SCOPE"
}
```

| 参数          | 说明                                         |
| ------------- | -------------------------------------------- |
| access_token  | 接口调用凭证                                 |
| expires_in    | access_token接口调用凭证超时时间，单位（秒） |
| refresh_token | 用户刷新access_token                         |
| openid        | 授权用户唯一标识                             |
| scope         | 用户授权的作用域，使用逗号（,）分隔          |

错误返回样例：

```text
{
"errcode":40030,"errmsg":"invalid refresh_token"
}
```

**接口说明**

### 检验授权凭证（access_token）是否有效

**请求说明**

```text
http请求方式: GET
https://api.weixin.qq.com/sns/auth?access_token=ACCESS_TOKEN&openid=OPENID
```

**参数说明**

| 参数         | 是否必须 | 说明                           |
| ------------ | -------- | ------------------------------ |
| access_token | 是       | 调用接口凭证                   |
| openid       | 是       | 普通用户标识，对该公众帐号唯一 |

**返回说明**

正确的 Json 返回结果：

```text
{
"errcode":0,"errmsg":"ok"
}
```

错误的 Json 返回示例:

```text
{
"errcode":40003,"errmsg":"invalid openid"
}
```

### **获取用户个人信息（UnionID机制）**

**接口说明**

此接口用于获取用户个人信息。开发者可通过 OpenID 来获取用户基本信息。特别需要注意的是，如果开发者拥有多个移动应用、网站应用和公众帐号，可通过获取用户基本信息中的 unionid 来区分用户的唯一性，因为只要是同一个微信开放平台帐号下的移动应用、网站应用和公众帐号，用户的 unionid 是唯一的。换句话说，同一用户，对同一个微信开放平台下的不同应用，unionid是相同的。请注意，在用户修改微信头像后，旧的微信头像 URL 将会失效，因此开发者应该自己在获取用户信息后，将头像图片保存下来，避免微信头像 URL 失效后的异常情况。

**请求说明**

```text
http请求方式: GET
https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID
```

**参数说明**

| 参数         | 是否必须 | 说明                                                        |
| ------------ | -------- | ----------------------------------------------------------- |
| access_token | 是       | 调用凭证                                                    |
| openid       | 是       | 普通用户的标识，对当前开发者帐号唯一                        |
| lang         | 否       | 国家地区语言版本，zh_CN 简体，zh_TW 繁体，en 英语，默认为en |

**返回说明**

正确的 Json 返回结果：

```text
{
"openid":"OPENID",
"nickname":"NICKNAME",
"sex":1,
"province":"PROVINCE",
"city":"CITY",
"country":"COUNTRY",
"headimgurl": "https://thirdwx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/0",
"privilege":[
"PRIVILEGE1",
"PRIVILEGE2"
],
"unionid": " o6_bmasdasdsad6_2sgVt7hMZOPfL"

}
```

| 参数       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| openid     | 普通用户的标识，对当前开发者帐号唯一                         |
| nickname   | 普通用户昵称                                                 |
| sex        | 普通用户性别，1为男性，2为女性                               |
| province   | 普通用户个人资料填写的省份                                   |
| city       | 普通用户个人资料填写的城市                                   |
| country    | 国家，如中国为CN                                             |
| headimgurl | 用户头像，最后一个数值代表正方形头像大小（有0、46、64、96、132数值可选，0代表640*640正方形头像），用户没有头像时该项为空 |
| privilege  | 用户特权信息，json数组，如微信沃卡用户为（chinaunicom）      |
| unionid    | 用户统一标识。针对一个微信开放平台帐号下的应用，同一用户的 unionid 是唯一的。 |

建议：

开发者最好保存用户 unionID 信息，以便以后在不同应用中进行用户信息互通。

错误的 Json 返回示例:

```text
{
"errcode":40003,"errmsg":"invalid openid"
}
```

**调用频率限制**

| 接口名                     | 频率限制 |
| -------------------------- | -------- |
| 通过 code 换取access_token | 1万/分钟 |
| 刷新access_token           | 5万/分钟 |
| 获取用户基本信息           | 5万/分钟 |

## 网页应用授权用户资料变更

1、	当部分用户的资料存在风险时，平台会对用户资料进行清理，并通过消息推送服务器通知最近30天授权过的开放平台开发者，我们建议开发者留意响应该事件，及时主动更新或清理用户的头像及昵称，降低风险。
2、	当用户撤回授权信息时，平台会通过消息推送服务器通知给开放平台开发者，请开发者注意及时删除用户信息。
 点击查看[消息推送服务器配置](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/message_push.html)

### 事件推送示例：

#### XML

```xml
<xml>
    <ToUserName><![CDATA[gh_870882ca4b1]]></ToUserName>
    <FromUserName><![CDATA[owAqB1v0ahK_Xlc7GshIDdf2yf7E]]></FromUserName>
    <CreateTime>1626857200</CreateTime>
    <MsgType><![CDATA[event]]></MsgType>
    <Event><![CDATA[user_info_modified]]></Event>
    <OpenID><![CDATA[owAqB1nqaOYYWl0Ng484G2z5NIwU]]></OpenID>
    <AppID><![CDATA[wx13974bf780d3dc89]]></AppID>
    <RevokeInfo><![CDATA[1]]></RevokeInfo>
</xml>
```

####  JSON

```json
{
"ToUserName": "gh_870882ca4b1",
"FromUserName": "oaKk346BaWE-eIn4oSRWbaM9vR7s",
"CreateTime": 1627359464,
"MsgType": "event",
"Event": "user_info_modified",
"OpenID": "oaKk343WOktAaT2ygsX138BGblrg",
"AppID": "wx13974bf780d3dc89",
"RevokeInfo": "301",
}
```

### 事件字段定义

| 属性         | 类型   | 说明                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| ToUserName   | string | 网站应用的UserName                                           |
| FromUserName | string | 平台推送服务UserName                                         |
| MsgType      | string | 默认为：Event                                                |
| Event        | string | user_info_modified：用户资料变更，user_authorization_revoke：用户撤回 |
| CreateTime   | number | 发送时间                                                     |
| OpenID       | string | 撤回或变更资料的用户OpenID                                   |
| AppID        | string | 网站应用的AppID                                              |
| RevokeInfo   | string | 用户撤回的授权信息，301:用户撤回网站应用所有授权信息         |

## 消息推送服务器设置

### 第一步：填写服务器配置

登录[OPEN平台](https://mp.weixin.qq.com/)后，在移动应用/网页应用详情页面 -「消息推送」中，管理员可启用消息服务，填写服务器地址（URL）、令牌（Token） 和 消息加密密钥（EncodingAESKey）等信息。

- URL: 开发者用来接收微信消息和事件的接口 URL。开发者所填写的URL 必须以 http:// 或 https:// 开头，分别支持 80 端口和 443 端口。
- Token: 可由开发者可以任意填写，用作生成签名（该 Token 会和接口 URL 中包含的 Token 进行比对，从而验证安全性）。
- EncodingAESKey: 由开发者手动填写或随机生成，将用作消息体加解密密钥。

同时，开发者可选择消息加解密方式：明文模式（默认）、兼容模式和安全模式。可以选择消息数据格式：XML 格式（默认）或 JSON 格式。

![image.png](https://s2.loli.net/2022/11/08/uFSG1iXYB4NZRnw.png)

模式的选择与服务器配置在提交后都会立即生效，请开发者谨慎填写及选择。切换加密方式和数据格式需要提前配置好相关代码，详情请参考 [消息加解密说明](https://developers.weixin.qq.com/doc/oplatform/Third-party_Platforms/2.0/api/Before_Develop/Message_encryption_and_decryption.html)。

### 第二步：验证消息的确来自微信服务器

开发者提交信息后，微信服务器将发送 GET 请求到填写的服务器地址 URL 上，GET请求携带参数如下表所示：

| 参数      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| signature | 微信加密签名，signature结合了开发者填写的 token 参数和请求中的 timestamp 参数、nonce参数。 |
| timestamp | 时间戳                                                       |
| nonce     | 随机数                                                       |
| echostr   | 随机字符串                                                   |

开发者通过检验 signature 对请求进行校验（下面有校验方式）。若确认此次 GET 请求来自微信服务器，请原样返回 echostr 参数内容，则接入生效，成为开发者成功，否则接入失败。加密/校验流程如下：

1. 将token、timestamp、nonce三个参数进行字典序排序
2. 将三个参数字符串拼接成一个字符串进行sha1加密
3. 开发者获得加密后的字符串可与 signature 对比，标识该请求来源于微信

验证 URL 有效性成功后即接入生效。

检验 signature 的PHP示例代码：

```text
private function checkSignature()
{
    $signature = $_GET["signature"];
    $timestamp = $_GET["timestamp"];
    $nonce = $_GET["nonce"];

    $token = TOKEN;
    $tmpArr = array($token, $timestamp, $nonce);
    sort($tmpArr, SORT_STRING);
    $tmpStr = implode( $tmpArr );
    $tmpStr = sha1( $tmpStr );

    if ($tmpStr == $signature ) {
        return true;
    } else {
        return false;
    }
}
```

PHP示例代码下载：[下载](https://wximg.gtimg.com/shake_tv/mpwiki/cryptoDemo.zip)

### 第三步：接收消息和事件

当某些特定的用户操作引发事件推送时（如用户资料变更时），微信服务器会将消息（或事件）的数据包以 POST 请求发送到开发者配置的 URL，开发者可以依据自身业务逻辑进行响应。事件类型消息推荐使用 FromUserName + CreateTime 排重。

服务器收到请求必须做出下述回复，这样微信服务器才不会对此作任何处理，并且不会发起重试。详见下面说明：

1. 直接回复success（推荐方式）
2. 直接回复空串（指字节长度为0的空字符串，而不是结构体中 content 字段的内容为空）



# 企业微信

## 企业微信网页授权登录

https://developer.work.weixin.qq.com/document/path/91335

企业微信提供了OAuth的授权登录方式，可以让从企业微信终端打开的网页获取成员的身份信息，从而免去登录的环节。
企业应用中的URL链接（包括自定义菜单或者消息中的链接），均可通过OAuth2.0验证接口来获取成员的UserId身份信息。

*最后更新：2022/10/11*

### OAuth2简介

OAuth2的设计背景，在于允许用户在不告知第三方自己的帐号密码情况下，通过授权方式，让第三方服务可以获取自己的资源信息。
详细的协议介绍，开发者可以参考[RFC 6749](https://tools.ietf.org/html/rfc6749)。

下面简单说明OAuth2中最经典的Authorization Code模式，流程如下：
![image.png](https://s2.loli.net/2022/11/08/AcXUPktGxpswQRj.png)

流程图中，包含四个角色。

- ResourceOwner为资源所有者，即为用户
- User-Agent为浏览器
- AuthorizationServer为认证服务器，可以理解为用户资源托管方，比如企业微信服务端
- Client为第三方服务

调用流程为：
A) 用户访问第三方服务，第三方服务通过构造OAuth2链接（参数包括当前第三方服务的身份ID，以及重定向URI），将用户引导到认证服务器的授权页
B) 用户选择是否同意授权
C) 若用户同意授权，则认证服务器将用户重定向到第一步指定的重定向URI，同时附上一个授权码。
D) 第三方服务收到授权码，带上授权码来源的重定向URI，向认证服务器申请凭证。
E) 认证服务器检查授权码和重定向URI的有效性，通过后颁发AccessToken（调用凭证）

> D)与E)的调用为后台调用，不通过浏览器进行

### 企业微信OAuth2接入流程

![image.png](https://s2.loli.net/2022/11/08/wEALgMmc6tQuOJU.png)

### 使用OAuth2前须知

#### 关于网页授权的可信域名

REDIRECT_URL中的域名，需要先配置至应用的“可信域名”，否则跳转时会提示“redirect_uri参数错误”。
**要求配置的可信域名，必须与访问链接的域名完全一致；若访问链接URL带了端口号，端口号也需要登记到可信域名中**。举个例子：

- 假定重定向访问的链接是：http://mail.qq.com:8080/cgi-bin/helloworld：

| 配置域名            | 是否正确 | 原因                                       |
| ------------------- | -------- | ------------------------------------------ |
| mail.qq.com:8080    | √        | 配置域名与访问域名完全一致                 |
| email.qq.com        | ×        | 配置域名必须与访问域名完全一致             |
| support.mail.qq.com | ×        | 配置域名必须与访问域名完全一致             |
| *.qq.com            | ×        | 不支持泛域名设置                           |
| mail.qq.com         | ×        | 配置域名必须与访问域名完全一致，包括端口号 |

- 假定配置的可信域名是 mail.qq.com：

| 访问链接                                 | 是否正确 | 原因                                              |
| ---------------------------------------- | -------- | ------------------------------------------------- |
| https://mail.qq.com/cgi-bin/helloworld   | √        | 配置域名与访问域名完全一致                        |
| http://mail.qq.com/cgi-bin/redirect      | √        | 配置域名与访问域名完全一致，与协议头/链接路径无关 |
| https://exmail.qq.com/cgi-bin/helloworld | ×        | 配置域名必须与访问域名完全一致                    |

#### 关于UserID机制

UserId用于在一个企业内唯一标识一个用户，通过网页授权接口可以获取到当前用户的UserId信息，如果需要获取用户的更多信息可以调用 通讯录管理 - [成员接口](https://developer.work.weixin.qq.com/document/path/91335#10019) 来获取。

#### 静默授权与手动授权

- 静默授权：用户点击链接后，页面直接302跳转至 redirect_uri?code=CODE&state=STATE
- 手动授权：用户点击链接后，会弹出一个中间页，让用户选择是否授权，用户确认授权后再302跳转至 redirect_uri?code=CODE&state=STATE

![image.png](https://s2.loli.net/2022/11/08/udC1SvteLKO52Rj.png)

#### 个人敏感信息授权管理

用户首次进入oauth2页面进行手动授权后，30天内再次进入应用页面不会再弹出授权页，默认授权用户当前授权的敏感信息。若30天内用户需要修改个人敏感信息授权，可进入应用详情页的“个人敏感信息授权管理”页面，重新更改个人敏感信息授权。

> 目前仅2022.6.20 20:00后新创建的的自建应用以及代开发应用或者所有的第三方应用，且用户曾经通过进入oauth2页面进行手动授权过才会出现该入口。

![image.png](https://s2.loli.net/2022/11/08/XHGJsbdVfm8Mgt3.png)

#### 缓存方案建议

通过OAuth2.0验证接口获取成员身份会有一定的时间开销。对于频繁获取成员身份的场景，建议采用如下方案：
1、企业应用中的URL链接直接填写企业自己的页面地址
2、成员操作跳转到步骤1的企业页面时，企业后台校验是否有标识成员身份的cookie信息，此cookie由企业生成
3、如果没有匹配的cookie，则重定向到OAuth验证链接，获取成员的身份信息后，由企业后台植入标识成员身份的cookie信息
4、根据cookie获取成员身份后，再进入相应的页面

### 构造网页授权链接

*最后更新：2022/06/21*

如果企业需要在打开的网页里面携带用户的身份信息，第一步需要构造如下的链接来获取code参数：

```javascript
https://open.weixin.qq.com/connect/oauth2/authorize?appid=CORPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_base&state=STATE&agentid=AGENTID#wechat_redirect
```

**参数说明：**

| 参数             | 必须 | 说明                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| appid            | 是   | 企业的CorpID                                                 |
| redirect_uri     | 是   | 授权后重定向的回调链接地址，**请使用urlencode对链接进行处理** |
| response_type    | 是   | 返回类型，此时固定为：code                                   |
| scope            | 是   | 应用授权作用域。 snsapi_base：[静默授权](https://developer.work.weixin.qq.com/document/path/91022#15019/静默授权与手动授权)，可获取成员的基础信息（UserId与DeviceId）； snsapi_privateinfo：[手动授权](https://developer.work.weixin.qq.com/document/path/91022#15019/静默授权与手动授权)，可获取成员的详细信息，包含头像、二维码等敏感信息。 |
| state            | 否   | 重定向后会带上state参数，企业可以填写a-zA-Z0-9的参数值，长度不可超过128个字节 |
| agentid          | 是   | 应用agentid，snsapi_privateinfo时必填                        |
| #wechat_redirect | 是   | 终端使用此参数判断是否需要带上身份信息                       |

员工点击后，页面将跳转至 redirect_uri?code=CODE&state=STATE，企业可根据code参数获得员工的userid。code长度最大为512字节。

**示例：**

假定当前企业CorpID：`wxCorpId`
访问链接：`http://api.3dept.com/cgi-bin/query?action=get`

根据URL规范，将上述参数分别进行UrlEncode，得到拼接的OAuth2链接为：

```javascript
https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxCorpId&redirect_uri=http%3a%2f%2fapi.3dept.com%2fcgi-bin%2fquery%3faction%3dget&response_type=code&scope=snsapi_base&state=#wechat_redirect
```

> 注意，构造OAuth2链接中参数的redirect_uri是经过UrlEncode的

员工点击后，页面将跳转至

```javascript
http://api.3dept.com/cgi-bin/query?action=get&code=AAAAAAgG333qs9EdaPbCAP1VaOrjuNkiAZHTWgaWsZQ&state=
```

企业可根据code参数调用[获取员工的信息](https://developer.work.weixin.qq.com/document/path/91022#15047)

### 获取访问用户身份

*最后更新：2022/09/23*

该接口用于根据code获取成员信息，适用于自建应用与代开发应用

**请求方式：**GET（**HTTPS**）
**请求地址：**https://qyapi.weixin.qq.com/cgi-bin/auth/getuserinfo?access_token=ACCESS_TOKEN&code=CODE
**参数说明：**

| 参数         | 必须 | 说明                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| access_token | 是   | [调用接口凭证](https://developer.work.weixin.qq.com/document/path/91023#15074) |
| code         | 是   | 通过成员授权获取到的code，最大为512字节。每次成员授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。 |

**权限说明：**
跳转的域名须完全匹配access_token对应应用的可信域名，否则会返回50001错误。
**返回结果：**
a) 当用户为企业成员时（无论是否在应用可见范围之内）返回示例如下：

```javascript
{
   "errcode": 0,
   "errmsg": "ok",
   "userid":"USERID",
   "user_ticket": "USER_TICKET"
}
```

| 参数        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| errcode     | 返回码                                                       |
| errmsg      | 对返回码的文本描述内容                                       |
| userid      | 成员UserID。若需要获得用户详情信息，可调用通讯录接口：[读取成员](https://developer.work.weixin.qq.com/document/path/91023#10019)。如果是互联企业/企业互联/上下游，则返回的UserId格式如：CorpId/userid |
| user_ticket | 成员票据，最大为512字节，有效期为1800s。 scope为snsapi_privateinfo，且用户在应用可见范围之内时返回此参数。 后续利用该参数可以获取用户信息或敏感信息，参见["获取访问用户敏感信息"](https://developer.work.weixin.qq.com/document/path/91023#39323)。暂时不支持上下游或/企业互联场景 |

b) 非企业成员时，返回示例如下：

```javascript
{
   "errcode": 0,
   "errmsg": "ok",
   "openid":"OPENID",
   "external_userid":"EXTERNAL_USERID"
}
```

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| errcode         | 返回码                                                       |
| errmsg          | 对返回码的文本描述内容                                       |
| openid          | 非企业成员的标识，对当前企业唯一。不超过64字节               |
| external_userid | 外部联系人id，当且仅当用户是企业的客户，且跟进人在应用的可见范围内时返回。如果是第三方应用调用，针对同一个客户，同一个服务商不同应用获取到的id相同 |

**出错返回示例：**

```javascript
{
   "errcode": 40029,
   "errmsg": "invalid code"
}
```

### 获取访问用户敏感信息

*最后更新：2022/09/19*

自建应用与代开发应用可通过该接口获取成员授权的敏感字段

**请求方式：**POST（**HTTPS**）
**请求地址：**https://qyapi.weixin.qq.com/cgi-bin/auth/getuserdetail?access_token=ACCESS_TOKEN

**请求包体：**

```javascript
{
   "user_ticket": "USER_TICKET"
}
```

**参数说明：**

| 参数         | 必须 | 说明                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| access_token | 是   | [调用接口凭证](https://developer.work.weixin.qq.com/document/path/95833#15074) |
| user_ticket  | 是   | 成员票据                                                     |

 

**权限说明：**
成员必须在应用的可见范围内。

**返回结果：**

```javascript
{
   "errcode": 0,
   "errmsg": "ok",
   "userid":"lisi",
   "gender":"1",
   "avatar":"http://shp.qpic.cn/bizmp/xxxxxxxxxxx/0",
   "qr_code":"https://open.work.weixin.qq.com/wwopen/userQRCode?vcode=vcfc13b01dfs78e981c",
   "mobile": "13800000000",
   "email": "zhangsan@gzdev.com",
   "biz_mail":"zhangsan@qyycs2.wecom.work",
   "address": "广州市海珠区新港中路"
}
```

**参数说明：**

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| errcode  | 返回码                                                       |
| errmsg   | 对返回码的文本描述内容                                       |
| userid   | 成员UserID                                                   |
| gender   | 性别。0表示未定义，1表示男性，2表示女性。仅在用户同意snsapi_privateinfo授权时返回真实值，否则返回0. |
| avatar   | 头像url。仅在用户同意snsapi_privateinfo授权时返回            |
| qr_code  | 员工个人二维码（扫描可添加为外部联系人），仅在用户同意snsapi_privateinfo授权时返回 |
| mobile   | 手机，仅在用户同意snsapi_privateinfo授权时返回，第三方应用不可获取 |
| email    | 邮箱，仅在用户同意snsapi_privateinfo授权时返回，第三方应用不可获取 |
| biz_mail | 企业邮箱，仅在用户同意snsapi_privateinfo授权时返回，第三方应用不可获取 |
| address  | 仅在用户同意snsapi_privateinfo授权时返回，第三方应用不可获取 |

注：对于自建应用与代开发应用，敏感字段需要管理员在应用详情里选择，且成员oauth2授权时确认后才返回。敏感字段包括：性别、头像、员工个人二维码、手机、邮箱、企业邮箱、地址。

## 企业微信扫码授权登录

https://developer.work.weixin.qq.com/document/path/91025

企业微信提供了OAuth的扫码登录授权方式，可以让企业的网站在浏览器内打开时，引导成员使用企业微信扫码登录授权，从而获取成员的身份信息，免去登录的环节。（注：此授权方式需要用户扫码，不同于“[网页授权登录](https://developer.work.weixin.qq.com/document/path/91025#15019)”；仅企业内可以使用此种授权方式，第三方服务商不支持使用。）

在进行企业微信授权登录之前，需要先在企业的管理端后台创建一个具备“企业微信授权登录”能力的应用（见“[开启网页授权登录](https://developer.work.weixin.qq.com/document/path/91025#15063/开启网页授权登录)”）。

### 企业微信扫码登录接入流程

![image.png](https://s2.loli.net/2022/11/08/BEm9AUYXxFgyZCP.png)

### 开启网页授权登录

登录 企业管理端后台->进入需要开启的自建应用->点击 “企业微信授权登录”，进入如下页面

![image.png](https://s2.loli.net/2022/11/08/zlt6x5L91sQHY2a.png)

然后点击 "设置授权回调域"，输入回调域名，点击“保存”。

**要求配置的授权回调域，必须与访问链接的域名完全一致**。举个例子：

- 假定重定向访问的链接是：http://mail.qq.com:8080/cgi-bin/login：

| 配置域名                | 是否正确 | 原因                                       |
| ----------------------- | -------- | ------------------------------------------ |
| mail.qq.com:8080        | √        | 配置域名与访问域名完全一致                 |
| email.qq.com            | ×        | 配置域名必须与访问域名完全一致             |
| support.mail.qq.com     | ×        | 配置域名必须与访问域名完全一致             |
| *.qq.com                | ×        | 不支持泛域名设置                           |
| mail.qq.com             | ×        | 配置域名必须与访问域名完全一致，包括端口号 |
| http://mail.qq.com:8080 | ×        | 不包括协议头                               |

构造扫码登录链接

*最后更新：2022/04/15*

### 构造独立窗口登录二维码

开发者需要构造如下的链接来获取code参数：

```javascript
https://open.work.weixin.qq.com/wwopen/sso/qrConnect?appid=CORPID&agentid=AGENTID&redirect_uri=REDIRECT_URI&state=STATE
```

**参数说明**

| 参数         | 必须 | 说明                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| appid        | 是   | 企业微信的CorpID，在企业微信管理端查看                       |
| agentid      | 是   | 授权方的网页应用ID，在具体的网页应用中查看                   |
| redirect_uri | 是   | 重定向地址，需要进行UrlEncode                                |
| state        | 否   | 用于保持请求和回调的状态，授权请求后原样带回给企业。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议企业带上该参数，可设置为简单的随机数加session进行校验 |
| lang         | 否   | 自定义语言，支持zh、en；lang为空则从Headers读取[Accept-Language](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)，默认值为zh |

> 若提示“该链接无法访问”，请检查参数是否填写错误，如redirect_uri的域名与网页应用的可信域名不一致。
> 若用户不在agentid所指应用的可见范围，扫码时会提示无权限。

**返回说明**
用户允许授权后，将会重定向到redirect_uri的网址上，并且带上code和state参数

> redirect_uri?code=CODE&state=STATE

若用户禁止授权，则重定向后不会带上code参数，仅会带上state参数

> redirect_uri?state=STATE

**示例**：

```javascript
假定当前
企业CorpID：wxCorpId
开启授权登录的应用ID：1000000
登录跳转链接：http://api.3dept.com
state设置为：weblogin@gyoss9

需要配置的授权回调域为：api.3dept.com

根据URL规范，将上述参数分别进行UrlEncode，得到拼接的OAuth2链接为：
https://open.work.weixin.qq.com/wwopen/sso/qrConnect?appid=wxCorpId&agentid=1000000&redirect_uri=http%3A%2F%2Fapi.3dept.com&state=web_login%40gyoss9
```

### 构造内嵌登录二维码

为了满足网站更定制化的需求，我们还提供了第二种获取code的方式，支持网站将企业微信登录二维码内嵌到自己页面中，用户使用企业微信扫码授权后通过JS将code返回给网站。
JS企业微信登录主要用途：网站希望用户在网站内就能完成登录，无需跳转到企业微信域下登录后再返回，提升企业微信登录的流畅性与成功率。 网站内嵌二维码企业微信登录JS实现办法：

#### 步骤一：引入JS文件

在需要展示企业微信网页登录二维码的网站引入如下JS文件（支持https）：
旧版：~~http://rescdn.qqmail.com/node/ww/wwopenmng/js/sso/wwLogin-1.0.0.js~~
新版（20220415更新）：http://wwcdn.weixin.qq.com/node/wework/wwopen/js/wwLogin-1.2.7.js

#### 步骤二：在需要使用微信登录的地方实例JS对象

**注意**：从wwLogin-1.2.5.js开始需要使用new WwLogin进行实例化

```javascript
var wwLogin = new WwLogin({
		"id": "wx_reg",  
		"appid": "",
		"agentid": "",
		"redirect_uri": "",
		"state": "",
		"href": "",
		"lang": "zh",
});
```

**参数说明**

| 参数          | 必须 | 说明                                                         |
| ------------- | ---- | ------------------------------------------------------------ |
| id            | 是   | 企业页面显示二维码的DOM id                                   |
| appid         | 是   | 企业微信的CorpID，在企业微信管理端查看                       |
| agentid       | 是   | 授权方的网页应用ID，在具体的网页应用中查看                   |
| redirect_uri  | 是   | 重定向地址，需要进行UrlEncode                                |
| state         | 否   | 用于保持请求和回调的状态，授权请求后原样带回给企业。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议企业带上该参数，可设置为简单的随机数加session进行校验 |
| href          | 否   | 自定义样式链接，企业可根据实际需求覆盖默认样式。详见文档底部FAQ |
| lang          | 否   | 自定义语言，支持zh、en；lang为空则从Headers读取[Accept-Language](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)，默认值为zh |
| self_redirect | 否   | true：手机点击确认登录后可以在 iframe 内跳转到 redirect_uri，false：手机点击确认登录后可以在 top window 跳转到 redirect_uri。默认为 false。 |

**方法说明**

| 方法      | 说明                       |
| --------- | -------------------------- |
| destroyed | 无需登录时，可手动销毁实例 |

```javascript
wwLogin.destroyed() // 注意wwLogin为实例对象
```

**1.网站内嵌二维码企业微信登录JS代码中href字段作用？**
答：如果企业觉得企业微信团队提供的默认样式与自己的页面样式不匹配，可以自己提供样式文件来覆盖默认样式。举个例子，如企业觉得默认二维码过大，可以提供相关css样式文件，并把链接地址填入href字段（只支持https协议的资源地址）：

```css
.impowerBox .qrcode {width: 200px;}
.impowerBox .title {display: none;}
.impowerBox .info {width: 200px;}
.status_icon {display: none  !important}
.impowerBox .status {text-align: center;} 
```

相关效果如下：

![image.png](https://s2.loli.net/2022/11/08/bD8hONUxdWQVpwt.png)



# 普通钉钉

https://open.dingtalk.com/document/isvapp-server/tutorial-enabling-login-to-third-party-websites

## 实现登录第三方网站

*更新时间：2022-11-03*
本文档指导你如何实现用户登录第三方网站（扫码或账密方式）。在本场景中，第三方网站可以获取用户授权的个人信息。

**说明** 

企业内部应用与三方企业应用实现流程类似，本文档以企业内部应用实现流程为例

## 调用步骤

步骤一：登录[钉钉开发者后台](https://open-dev.dingtalk.com/#/index)，创建并配置应用。

（1）以创建企业内部应用-H5微应用为例。

（2）配置H5微应用相关信息，开发模式、服务器出口IP、应用首页地址等。

步骤二：添加接口调用权限。

步骤三：配置frp内网穿透，用于生成一个公网域名进行测试。

步骤四：登录[钉钉开发者后台](https://open-dev.dingtalk.com/#/index)，设置第三方网站的回调域名。

步骤五：搭建后端服务。

步骤六：实现登录第三方网站。

步骤七：访问第三方网站地址，并获取用户个人信息。

（1）在浏览器里输入构造完成的第三方网站地址

（2）使用扫码或者通过钉钉账号登录。

（3）登录后，打开授权页面。

（4）在授权页面，点击同意，并触发相关操作。

（5）获取到用户个人信息。

## 准备工作

在开始本教程前，确保你已经完成了以下准备工作：

- 需要成为钉钉开发者，详情请参考[成为钉钉开发者](https://open.dingtalk.com/document/isv/become-a-dingtalk-developer#topic-2024337)。
- 确保您已经安装了Java开发环境（安装JDK）以及Java项目构建工具Maven。

## 步骤一：创建并配置应用

在本部分，你需要在开发者后台创建一个H5微应用，并完成通讯录权限和用户个人手机号权限的配置，用于获取用户个人信息。

1. 登录[钉钉开发者后台](https://open-dev.dingtalk.com/#/index)。

    **说明** 

    只有管理员和子管理员可登录开发者后台，详情可查看[成为钉钉开发者](https://open.dingtalk.com/document/isv/become-a-dingtalk-developer#topic-2024337)。

2. 在**开发者后台**页面，选择**企业内部开发**，然后单击**创建应用**。

    ![创建企业内部应用 ](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5144871461/p344004.png)

3. 在弹出的创建应用页面中填写基本信息，然后单击确定创建。

    - 应用类型：选择H5微应用。

    - 开发方式：选择企业自主开发。

        ![内部应用设置](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9309871461/p348879.png)

4. 应用创建完成后，在**基础信息**页面，复制应用的**AppKey**和**AppSecret**备用。

    ![基础信息 ](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4319813461/p344327.png)

5. 单击**开发管理**进入开发管理页面，然后单击**修改**，并根据以下内容配置开发信息。

    - **开发模式**：选择**开发应用**。

    - **服务器出口IP**：输入调用钉钉服务端API时使用的IP即企业服务器的公网IP，多个IP请以英文逗号","隔开，支持带一个*号通配符的IP格式。

        本教程设置为`127.0.0.1`。

    - **应用首页地址**：输入应用首页URL，在移动端工作台点击应用图标会跳转到此页面。可输入后端服务部署的服务器的IP或域名。例如：`http://公网IP:8080`。

        本教程设置为`https://ding-doc.dingtalk.com/`。

## 步骤二：添加接口权限

1.登录开发者后台-点击应用开发-企业内部应用，找到对应的应用并点击。

2.单击**权限管理**进入权限管理页面，根据以下配置添加接口调用权限。

（1）权限范围选择**全部员工**。

（2）选择**个人权限**，申请**个人手机号信息**和**通讯录个人信息读权限。**

![个人权限申请](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9504335361/p345088.png)

## 步骤三：配置frp内网穿透

在本部分，你将使用frp内网穿透工具生成一个公网域名用于教程测试。详细请参考[frp内网穿透](https://developer.aliyun.com/article/853534?spm=ding_open_doc.document.0.0.22dd1a0ddDUM4J)。（原链接已不存在，[试试这个](https://cloud.tencent.com/developer/article/1837482)，但是首先需要一台具有外网IP的服务器）

## 步骤四：设置第三方网站的回调域名

在本部分，你将在开发者后台设置第三方网站的回调地址。

1. 登录[钉钉开发者后台](https://open-dev.dingtalk.com/#/index)，找到对应的应用，并点击应用。

2. 单击**钉钉登录与分享**，填写回调域名，点击**添加**。

    **说明** 

    （1）有以下方式设置域名，可以选择其中一种。在后续“搭建后端服务”章节使用，请按实际使用进行替换。

    - 使用frp内网穿透工具，设置服务端域名。
        - 推荐使用，开发者自定义一个设置的服务端域名，请求地址设置为http://xxxxx.vaiwan.cn/auth。请替换地址中的xxxxx。
        - 使用本文档配置的frp内网穿透时设置的服务端域名，请求地址设置为http://abc.vaiwan.cn/auth。该地址使用的开发者较多，可能导致无法使用。
    - 开发者使用自己的服务器域名。

    （2）回调域名的重定向地址。

    - 重定向至服务端接口，本文档配置回调域名地址为服务端接口地址。
    - 重定向至第三方网站网页地址。

    ![回调地址](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6756665461/p408545.png)

## 步骤五：搭建后端服务

在本部分，你将使用spring.io快速搭建服务端项目。

1. 使用浏览器访问https://start.spring.io/。

2. 根据以下操作下载服务端代码。

    1. 配置项目信息。

        - Project：选择Maven项目。

        - Language：选择Java。

        - Packaging：选择Jar包

        - Java：本教程选择Java8，你可以根据自身需要选择相应的Java版本。

            ![spring initializr](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9504335361/p345115.png)

    2. 单击**GENERATE**生成项目文件及下载。

3. 解压下载的项目文件，然后使用**IntelliJ IDEA**打开。

4. 点击pom文件，添加如下依赖。

    ```javascript
    <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>fastjson</artifactId>
          <version>1.2.6</version>
     </dependency>
     <dependency>
      <groupId>com.aliyun</groupId>
      <artifactId>dingtalk</artifactId>
      <version>1.1.86</version>
     </dependency>
    ```

5. 项目加载完成后，在src/main/java/com/example/demo/目录下新建一个类LoginController。

**说明** 

请确保以下内容正确，否则无法实现相应功能。

- 接口地址/auth需要和开发者后台钉钉登录与分享的地址http://xxxxx/auth保持一致。
- setClientId、setClientSecret要替换成应用基础信息-应用信息中的AppKey和AppSecret。

| 需替换内容                                                   | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 接口地址/auth                                                | 需要和开发者后台钉钉登录与分享的地址http:xxxxx/auth保持一致。![登录与分享](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5167741461/p380973.png) |
| setClientId("dingwxxxxx请替换为正确的应用信息的AppKey")      | 参数务必替换为：应用基础信息-应用信息中的AppKey。![AppKey等](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5167741461/p380972.png) |
| setClientSecret("ICLbFFjNxxx请替换为正确的应用信息的AppSecret") | 参数务必替换为：应用基础信息-应用信息中的AppSecret。![AppKey等 ](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5167741461/p380972.png) |

```javascript
package com.example.demo;

import com.alibaba.fastjson.JSON;
import com.aliyun.dingtalkcontact_1_0.models.GetUserHeaders;
import com.aliyun.dingtalkoauth2_1_0.models.GetUserTokenRequest;
import com.aliyun.dingtalkoauth2_1_0.models.GetUserTokenResponse;
import com.aliyun.teaopenapi.models.Config;
import com.aliyun.teautil.models.RuntimeOptions;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LoginController {

    public static com.aliyun.dingtalkoauth2_1_0.Client authClient() throws Exception {
        Config config = new Config();
        config.protocol = "https";
        config.regionId = "central";
        return new com.aliyun.dingtalkoauth2_1_0.Client(config);
    }
    /**
     * 获取用户token
     * @param authCode
     * @return
     * @throws Exception
     */
    //接口地址：注意/auth与钉钉登录与分享的回调域名地址一致
    @RequestMapping(value = "/auth", method = RequestMethod.GET)
    public String getAccessToken(@RequestParam(value = "authCode")String authCode) throws Exception {
        com.aliyun.dingtalkoauth2_1_0.Client client = authClient();
        GetUserTokenRequest getUserTokenRequest = new GetUserTokenRequest()
                
                //应用基础信息-应用信息的AppKey,请务必替换为开发的应用AppKey
                .setClientId("dingwxxxxx请替换为正确的应用信息的AppKey")
                
                //应用基础信息-应用信息的AppSecret，,请务必替换为开发的应用AppSecret
                .setClientSecret("ICLbFFjNxxx请替换为正确的应用信息的AppSecret")
                .setCode(authCode)
                .setGrantType("authorization_code");
        GetUserTokenResponse getUserTokenResponse = client.getUserToken(getUserTokenRequest);
        //获取用户个人token
        String accessToken = getUserTokenResponse.getBody().getAccessToken();
        return  getUserinfo(accessToken);

    }
    public static com.aliyun.dingtalkcontact_1_0.Client contactClient() throws Exception {
        Config config = new Config();
        config.protocol = "https";
        config.regionId = "central";
        return new com.aliyun.dingtalkcontact_1_0.Client(config);
    }
    /**
     * 获取用户个人信息
     * @param accessToken
     * @return
     * @throws Exception
     */
    public String getUserinfo(String accessToken) throws Exception {
        com.aliyun.dingtalkcontact_1_0.Client client = contactClient();
        GetUserHeaders getUserHeaders = new GetUserHeaders();
        getUserHeaders.xAcsDingtalkAccessToken = accessToken;
        //获取用户个人信息
        String me = JSON.toJSONString(client.getUserWithOptions("me", getUserHeaders, new RuntimeOptions()).getBody());
        System.out.println(me);
        return me;
    }
}
```



## 步骤六：实现登录第三方网站

### 使用钉钉提供的页面登录授权

1. 构造第三方网站访问地址。

    **重要**

    - 为了方便阅读，以下参数示例做了换行处理。正常情况下无需进行参数换行。
    - redirect_uri必须要做urlencode，以下示例已经进行urlencode。
    - 以下登录页面在初次校验登录状态时显示。

    redirect_uri必须要做urlencode

    ```javascript
    https://login.dingtalk.com/oauth2/auth?
    redirect_uri=https%3A%2F%2Fwww.aaaaa.com%2Fauth
    &response_type=code
    &client_id=dingxxxxxxx   //应用的AppKey 
    &scope=openid   //此处的openId保持不变
    &state=dddd
    &prompt=consent
    ```

    | 参数            | 是否必填 | 说明                                                         |
    | --------------- | -------- | ------------------------------------------------------------ |
    | redirect_uri    | 是       | 授权通过/拒绝后回调地址。**重要** 需要与开发者后台钉钉登录与分享的地址保持一致，redirect_uri需要进行urlencode。 |
    | response_type   | 是       | 固定值为code。授权通过后返回authCode。                       |
    | client_id       | 是       | 步骤一中创建的应用详情中获取。企业内部应用：client_id为应用的AppKey。第三方企业应用：client_id为应用的SuiteKey。 |
    | scope           | 是       | 授权范围，授权页面显示的授权信息以应用注册时配置的为准。当前只支持两种输入：**openid**：授权后可获得用户userid**openid** corpid：授权后可获得用户id和登录过程中用户选择的组织id，空格分隔。注意url编码。 |
    | state           | 否       | 跟随authCode原样返回。                                       |
    | prompt          | 是       | 值为consent时，会进入授权确认页。                            |
    | org_type        | 否       | 控制输出特定类型的组织列表，org_type=management 表示只输出有管理权限的组织。**重要** scope包含corpid时该参数存在意义。 |
    | corpId          | 否       | 用于指定用户需要选择的组织。**重要**scope包含corpid时该参数存在意义。传入的corpId需要是当前用户所在的组织。 |
    | exclusiveLogin  | 否       | true表示专属帐号登录，展示组织代码输入页。                   |
    | exclusiveCorpId | 否       | 开启了专属帐号功能的组织corpId。**重要**exclusiveLogin为true时，该参数表示直接进入该组织的登录页。exclusiveLogin为false时，该参数无意义。 |

### **内嵌二维码方式登录授权**

**警告** 

嵌入二维码的页面必须和redirect_uri参数所指定的页面“同源”，否则扫码后会没有反应，“同源”指：协议相同、二级或三级域名相同、端口号相同等。详情请参考文档[浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)。

1. 在页面中引入钉钉扫码登录JSSDK。

    ```javascript
    <script src="https://g.alicdn.com/dingding/h5-dingtalk-login/0.21.0/ddlogin.js"></script>
    ```

2. 在需要引入扫码登录的地方，调用如下方法。

    ```javascript
    <!-- STEP1：在HTML中添加包裹容器元素 -->
    <div id="self_defined_element" class="self-defined-classname"></div>
    <style>
        /* STEP2：指定这个包裹容器元素的CSS样式，尤其注意宽高的设置 */
        .self-defined-classname {
            width: 300px;
            height: 300px;
        }
    </style>
    <script>
        // STEP3：在需要的时候，调用 window.DTFrameLogin 方法构造登录二维码，并处理登录成功或失败的回调。
        window.DTFrameLogin(
            {
                id: 'self_defined_element',
                width: 300,
                height: 300,
            },
            {
                redirect_uri: encodeURIComponent('http://www.aaaaa.com/a/b/'),
                client_id: 'dingxxxxxxxxxxxx',
                scope: 'openid',
                response_type: 'code',
                state: 'xxxxxxxxx',
                prompt: 'consent',
            },
            (loginResult) => {
                const {redirectUrl, authCode, state} = loginResult;
                // 这里可以直接进行重定向
                window.location.href = redirectUrl;
                // 也可以在不跳转页面的情况下，使用code进行授权
                console.log(authCode);
            },
            (errorMsg) => {
                // 这里一般需要展示登录失败的具体原因
                alert(`Login Error: ${errorMsg}`);
            },
        );
    </script>
    ```

    参数说明((TypeScript语言描述))：

    ```javascript
    // ********************************************************************************
    // window.DTFrameLogin方法定义
    // ********************************************************************************
    window.DTFrameLogin: (
      frameParams: IDTLoginFrameParams, // DOM包裹容器相关参数
      loginParams: IDTLoginLoginParams, // 统一登录参数
      successCbk: (result: IDTLoginSuccess) => void, // 登录成功后的回调函数
      errorCbk?: (errorMsg: string) => void,         // 登录失败后的回调函数
    ) => void;
    
    // ********************************************************************************
    // DOM包裹容器相关参数
    // ********************************************************************************
    // 注意！width与height参数只用于设置二维码iframe元素的尺寸，并不会影响包裹容器尺寸。
    // 包裹容器的尺寸与样式需要接入方自己使用css设置
    interface IDTLoginFrameParams {
      id: string;      // 必传，包裹容器元素ID，不带'#'
      width?: number;  // 选传，二维码iframe元素宽度，最小280，默认300
      height?: number; // 选传，二维码iframe元素高度，最小280，默认300
    }
    
    // ********************************************************************************
    // 统一登录参数
    // ********************************************************************************
    // 参数意义与“拼接链接发起登录授权”的接入方式完全相同（缺少部分参数）
    // 增加了isPre参数来设定运行环境
    interface IDTLoginLoginParams {
      redirect_uri: string;     // 必传，注意url需要encode
      response_type: string;    // 必传，值固定为code
      client_id: string;        // 必传
      scope: string;            // 必传，如果值为openid+corpid，则下面的org_type和corpId参数必传，否则无法成功登录
      prompt: string;           // 必传，值为consent。
      state?: string;           // 选传
      org_type?: string;        // 选传，当scope值为openid+corpid时必传
      corpId?: string;          // 选传，当scope值为openid+corpid时必传
      exclusiveLogin?: string;  // 选传，如需生成专属组织专用二维码时，可指定为true，可以限制非组织帐号的扫码
      exclusiveCorpId?: string; // 选传，当exclusiveLogin为true时必传，指定专属组织的corpId
    }
    
    // ********************************************************************************
    // 登录成功后返回的登录结果
    // ********************************************************************************
    interface IDTLoginSuccess {
      redirectUrl: string;   // 登录成功后的重定向地址，接入方可以直接使用该地址进行重定向
      authCode: string;      // 登录成功后获取到的authCode，接入方可直接进行认证，无需跳转页面
      state?: string;        // 登录成功后获取到的state
    }
    ```

## 步骤七：访问第三方网站地址

1. 在浏览器里输入第三方网站地址或访问内嵌二维码的页面地址。

    **说明** 

    本示例采用使用钉钉提供的页面登录授权方式。

2. 使用扫码或者通过钉钉账号登录。

    - 无登录状态时显示：

        ![扫码登录 ](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/9504335361/p345164.png)![钉钉账号登录](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/0604335361/p345165.png)

    - 登录状态时显示：![登录状态显示](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/3058151461/p381075.png)

3.登录后，打开授权页面。

**说明** 

首次授权时，显示授权页面，如下图所示。

![授权页面](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/0604335361/p345166.png)4.单击同意，触发以下流程。

（1）点击同意后，触发请求步骤四设置的第三方网站的回调域名，钉钉在url返回authCode。如下图所示。

![扫码登录第三方网站-authcode截图](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6167741461/p381025.png)

（2）根据authCode，调用服务端[获取用户token](https://open.dingtalk.com/document/isvapp-server/obtain-user-token#topic-2042816)接口，获取用户个人token。

**说明** 

以下代码对应步骤五中的代码，请替换为开发者开发的应用的AppKey、AppSecret，测试时请复制使用步骤五中的代码。

```javascript
    /**
     * 获取用户token
     * @param authCode
     * @return
     * @throws Exception
     */
    //接口地址：注意/auth与钉钉登录与分享的回调域名地址一致
    @RequestMapping(value = "/auth", method = RequestMethod.GET)
    public String getAccessToken(@RequestParam(value = "authCode")String authCode) throws Exception {
        com.aliyun.dingtalkoauth2_1_0.Client client = authClient();
        GetUserTokenRequest getUserTokenRequest = new GetUserTokenRequest()
                
                //应用基础信息-应用信息的AppKey,请务必替换为开发的应用AppKey
                .setClientId("dingwxxxxx请替换为正确的应用信息的AppKey")
                
                //应用基础信息-应用信息的AppSecret，,请务必替换为开发的应用AppSecret
                .setClientSecret("ICLbFFjNxxx请替换为正确的应用信息的AppSecret")
                .setCode(authCode)
                .setGrantType("authorization_code");
        GetUserTokenResponse getUserTokenResponse = client.getUserToken(getUserTokenRequest);
        //获取用户个人token
        String accessToken = getUserTokenResponse.getBody().getAccessToken();
        return  getUserinfo(accessToken);

    }
```

（3）根据用户个人token，调用[获取用户通讯录个人信息](https://open.dingtalk.com/document/isvapp-server/dingtalk-retrieve-user-information#doc-api-dingtalk-GetUser)接口，实现获取用户个人信息。

**说明**



- 获取用户个人信息，获取当前授权人的信息，unionId参数值请传字符串me。
- 以下代码为步骤五中的代码，测试时请复制使用步骤五中的代码。

```javascript
    /**
     * 获取用户个人信息
     * @param accessToken
     * @return
     * @throws Exception
     */
    public String getUserinfo(String accessToken) throws Exception {
        com.aliyun.dingtalkcontact_1_0.Client client = contactClient();
        GetUserHeaders getUserHeaders = new GetUserHeaders();
        getUserHeaders.xAcsDingtalkAccessToken = accessToken;
        //获取用户个人信息
        String me = JSON.toJSONString(client.getUserWithOptions("me", getUserHeaders, new RuntimeOptions()).getBody());
        System.out.println(me);
        return me;
    }
```

控制台返回信息如下：![扫码登录第三方网站-获取用户个人信息](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6167741461/p381032.png)





# 专有钉钉

https://openplatform-portal.dg-work.cn/portal/#/helpdoc?apiType=DEV_GUIDE&docKey=3355041

## 实现扫码登录

更新时间：2022-11-07

## 扫码登录流程

![image.png](https://s2.loli.net/2022/11/07/QmpzTufKNLDM6xi.png)

## 准备工作

在开始本教程前，确保你已经完成了以下准备工作：

- 需要成为专有钉钉开发者，详情请参考[入驻开放平台](https://openplatform-portal.dg-work.cn/portal/#/helpdoc?apiType=QUICK_START&docKey=3355320)。

## 步骤一：创建并配置应用

首先在专有钉钉开放平台创建并配置扫码登录应用。

1. 登录[专有钉钉开放平台](https://openplatform-portal.dg-work.cn/devPage/#/appDev/myApp)。

2. 在**开放平台**页面，点击**应用开发**头部菜单，选择左侧**扫码登录**，然后单击**创建**。

3. 在弹出的扫码登录页面中填写基本信息（应用名称、应用标识、应用描述），然后单击确定创建。

    ![image-20221107170555856](https://s2.loli.net/2022/11/07/q2TgzlQByWxFaLU.png)

4. 应用创建完成后，点击**详情**,在**凭证与基础信息**页面，可查看应用的**AppKey**和**AppSecret**，**应用标识**可作为构造登录页面client_id的参数值。

    ![image-20221107171120540](https://s2.loli.net/2022/11/07/A7869zWUPuMY1on.png)

5. 单击**应用配置**进入应用配置页面，设置应用的**回调地址**。

    - **回调地址**：输入应用首页URL，在移动端工作台点击应用图标会跳转到此页面。可输入后端服务部署的服务器的IP或域名。例如：`http://公网IP:8080`。

    ![image.png](https://s2.loli.net/2022/11/07/lfwnjvSmqHpKXc5.png)

## 步骤二：构造扫码登录页面

Web系统可以通过两种方式实现政务钉钉扫码登录。

### **方式一 使用政务钉钉提供的扫码登录页面**

在企业Web系统里，用户点击使用钉钉扫码登录，第三方Web系统跳转到如下地址：

```plain
https://login.dg-work.cn/oauth2/auth.htm?response_type=code&client_id=应用标识&redirect_uri=回调地址&scope=get_user_info&authType=QRCODE
```

URL中的client_id和redirect_uri两个参数的值填入第三方web系统的应用标识和回调地址。政务钉钉用户扫码登录并确认后，会302到你指定的redirect_uri，并向url参数中追加临时授权码code（此code非authcode）及state两个参数。

**注意事项：**

**参数"redirect_uri=回调地址"涉及的域名，需和创建扫码登录应用授权时填写的回调域名一致，否则会提示无权限访问。**

生成二维码大小固定为200*200px，不支持修改。

### **方式二 支持网站将政务钉钉登录二维码内嵌到自己页面中**

**步骤1：在页面中通过iframe嵌入页面**

通过方式一构造的地址增加`embedMode=true`的参数

```plain
https://login.dg-work.cn/oauth2/auth.htm?response_type=code&client_id=应用标识&redirect_uri=回调地址&scope=get_user_info&authType=QRCODE&embedMode=true
```

**步骤2：扫码成功后需要在页面中监听扫码结果**

```javascript
<script type="application/javascript">
    window.addEventListener('message', function(event) {
                    // 这里的event.data 就是登录成功的信息
                          // 数据格式：{ "code": "aaaa", "state": "bbbb" }
        alert(JSON.stringify(event.data));
    });
</script>
```

注意：生成二维码大小固定为200*200px，不支持修改。

![image.png](https://s2.loli.net/2022/11/07/ojPNOBW75etYH9q.png)

### 各环境域名/登录域名

| 环境   | 开放平台域名（调接口使用）                                   | 登录域名（构造登录页面）                                     |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Saas   | openplatform.dg-work.cn                                      | login.dg-work.cn                                             |
| 浙政钉 | openplatform-pro.ding.zj.gov.cn（域名对应政务外网IP：59.202.52.1） | login-pro.ding.zj.gov.cn（域名对应政务外网IP：59.202.52.68） |

## 步骤三：获取应用access_token

获取access_token，请参考[获取access_token](https://openplatform-portal.dg-work.cn/portal/#/helpdoc?apiType=DEV_GUIDE&docKey=3355045)。注意请使用扫码应用的ak/sk获取access_token。

**【注意】正常情况下 access_token 有效期为7200秒，有效期内重复获取返回相同结果，并⾃动续期。**

#### **请求方式：GET（HTTPS）**

#### **接口名**

/gettoken.json

#### **请求参数**

| 参数      | 参数类型 | 必须 | 说明              |
| --------- | -------- | ---- | ----------------- |
| appkey    | String   | 是   | 应用的唯一标识key |
| appsecret | String   | 是   | 应用的密钥        |

**SDK请求示例（JAVA）**：

```java
ExecutableClient executableClient =ExecutableClient.getInstance();
executableClient.setAccessKey("appkey");
executableClient.setSecretKey("appsecrt");
executableClient.setDomainName("不同环境对应不同域名");
executableClient.setProtocal("https");
executableClient.init();
//executableClient要单例，并且使用前要初始化，只需要初始化一次

String api = "/gettoken.json";
GetClient getClient = executableClient.newGetClient(api);
//设置参数
getClient.addParameter("appkey", "37affba10137489c9cc8812b6b19590000003501");
getClient.addParameter("appsecret", "37affba10137489c9cc8812b6b19590000003501");
//调用API
String apiResult = getClient.get();
System.out.println(apiResult);
```

#### **返回参数**

| 参数        | 说明                 |
| ----------- | -------------------- |
| accessToken | 应用access_token     |
| expiresIn   | 过期时间，单位（秒） |

#### **返回结果**

```json
{
    "success":true,
    "content":{
        "data":{
            "accessToken":"c139fe44362f41b6b84862ec82ab84d9",
            "expiresIn":"7200"
        },
        "requestId":"df04428415724925400701038d663a",
        "responseMessage":"OK",
        "responseCode":"0",
        "success": true
    }
}
```

#### **常见错误**

该接口常见报错

```
OPF-B001-05-16-0009	SignatureDoesNotMatch
```

请确认appsecret 是否正确，域名配置是否正确，以及appkey与appsecret是否匹配。

## 步骤四：获取授权用户的个人信息

服务端通过临时授权码获取授权用户的个人信息

#### **请求方式：POST（HTTPS）**

#### **接口名**

/rpc/oauth2/getuserinfo_bycode.json

注意：请使用接口域名调用接口，不能使用登录域名调接口。

#### **请求参数**

| 参数         | 参数类型 | 必须 | 说明                                                         |
| ------------ | -------- | ---- | ------------------------------------------------------------ |
| access_token | String   | 是   | 调用接口凭证，应用access_token                               |
| code         | String   | 是   | 用户授权的临时授权码code，只能使用一次；在前面步骤中跳转到redirect_uri时会追加code参数 |

#### **返回结果**

```json
{
    "success":true,
    "content":{
        "data":{
            "accountId":100135,
            "lastName":"俊锋",
            "clientId":"mozi-buc-sso",
            "realmId":12371,
            "tenantName":"租户2",
            "realmName":"租户2",
            "namespace":"local",
            "tenantId":12371,
            "nickNameCn":"俊锋",
            "tenantUserId":"12371$100135",
            "account":"admin2"
        },
        "success":true,
        "responseMessage":"成功",
        "responseCode":"0"
    }
}
```

#### **注：如果是超管账号调用该接口返回结果中无employeeCode，普通账号调用该接口有。**

#### **返回参数**

| 参数         | 说明                       |
| ------------ | -------------------------- |
| accountId    | 账号id                     |
| realmId      | 租户id                     |
| realmName    | 租户名                     |
| lastName     | 姓名                       |
| nickNameCn   | 昵称                       |
| account      | 登录账号                   |
| employeeCode | 人员code                   |
| tenantUserId | 员工在当前企业内的唯一标识 |
| namespace    | 账号类型标识               |
| clientId     | 应用标识                   |
| tenantId     | 租户id                     |
| tenantName   | 租户名                     |

#### **错误码**

| responseCode | responseMessage             |
| ------------ | --------------------------- |
| 240111       | code失效或不存在            |
| 240133       | 应用accessToken失效或不存在 |

