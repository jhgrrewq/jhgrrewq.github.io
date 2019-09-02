## 跨域认证问题

互联万服务用户认证一般流程如下

- 用户向服务器发送用户名密码
- 服务器验证通过后，在当前对话 session 中保存相关数据，如用户角色、登录时间等
- 服务器向用户返回一个 session_id，写入用户 cookie
- 用户随后每个请求，都会通过 cookie，将 session_id 传回服务器
- 服务器收到 session_id，找到前期保存数据

这种模式的问题在于扩展性不好，对于服务器集群或是跨域的服务导向架构，要求 session 数据共享。一般有两种解决方法：

- session 数据持久化，写入数据库或别的持久层
- 服务器不保存 session 数据，所有数据保存在客户端，每次请求都发回服务器。JWT 方法就是其中代表

## JWT 原理

```js
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```

JWT 原理是服务器认证后生成一个 json 对象 (类似上面这个 json 对象) 发回给用户，之后用户和服务器通信都要发回这个 json 对象。服务器完全只靠这个对象认定用户身份。为防止用户出篡改数据，服务器在生成这个对象时会加上签名

## JWT 数据结构

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

JWT 是一字符串，中间用点`.` 分割成了三部分：

- Header (头部)
- Payload (负载)
- Signature (签名）

写成一行就是 `Header.Payload.Signature`

### Header

Header 部分是一个 json 对象，描述 JWT 的元数据， 一般如下

```js
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg` 属性表示签名算法，默认是 HMAC SHA256（写成 HS256）

- `typ` 属性表示令牌的类型，JWT 令牌统一为 `JWT`

最终要将上述 json 对象使用 `Base64URL `算法转成字符串

### Payload

Payload 部分也是一个 json 对象，用来存放实际需要传输的数据。JWT 规定了 7 个官方字段供选用

- iss(issuser)  发送人
- exp(expiration time) 过期时间
- sub(subject) 主题
- aud(audience) 受众
- nbf(Not Before) 生效时间
- iat(Issued At) 签发时间
- jti(JWT ID) 编号

除了官方字段，还可以在这部分定义私有字段,  如

```js
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

JWT 默认是不加密的，任何人都可以读取

Payload 部分也要使用 `Base64URL `算法转成字符串

### Signature

Signature 部分是对前两部分的签名，防止篡改

首先需要指定一个`密钥（secret）`。 该密钥只能服务器知道，不能泄露给用户。然后使用 Header 指定的签名算法（默认是HMAC SHA256），按照下面公司是产生签名

```js
HMACSHA256(base64UrlEncode(header)+'.'+base64UrlEncode(payload),
  secret)
```

算出签名后，把 Header Payload Signature 三部分使用 `.`  拼接成字符串，返回给用户

### base64UrlEncode

之前提到 Header 和 Payload 串形算法是 base64URL 这个算法类似 base64. 因为 JWT 作为令牌有时候可能会放到 url 中（如 http://xxx.xx.com/?token=xx）。base64 中有三个字符 `+` `/` `=`

在 url 中有特殊含义，因此要替换掉，`=` 省略、`+` 替换成 `-`、`/` 体替换成`_`， 这就是base64URL 算法

## JWT 使用方式

客户端收到服务器发的 JWT，可以存储在 cookie 中，也可以存储在 localStorage 中。此后客户端每次与服务器通信都要带上这个 JWT。可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面

```bash
Authorization: Bearer <token>
```

## JWT 特点

- JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次
- JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑
- JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证
- 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输

## jwt-simple 源码

```js
/**
 * module dependencies
 */
var crypto = require('crypto');
/**
 * support algorithm mapping
 */
var algorithmMap = {
  HS256: 'sha256',
  HS384: 'sha384',
  HS512: 'sha512',
  RS256: 'RSA-SHA256'
};
/**
 * Map algorithm to hmac or sign type, to determine which crypto function to use
 */
var typeMap = {
  HS256: 'hmac',
  HS384: 'hmac',
  HS512: 'hmac',
  RS256: 'sign'
};
/**
 * expose object
 */
var jwt = module.exports;
/**
 * version
 */
jwt.version = '0.5.6';
/**
 * Decode jwt
 *
 * @param {Object} token
 * @param {String} key
 * @param {Boolean} [noVerify]
 * @param {String} [algorithm]
 * @return {Object} payload
 * @api public
 */
jwt.decode = function jwt_decode(token, key, noVerify, algorithm) {
  // check token
  if (!token) {
    throw new Error('No token supplied');
  }
  // check segments
  var segments = token.split('.');
  if (segments.length !== 3) {
    throw new Error('Not enough or too many segments');
  }
  // All segment should be base64
  var headerSeg = segments[0];
  var payloadSeg = segments[1];
  var signatureSeg = segments[2];
  // base64 decode and parse JSON
  var header = JSON.parse(base64urlDecode(headerSeg));
  var payload = JSON.parse(base64urlDecode(payloadSeg));

  if (!noVerify) {
    if (!algorithm && /BEGIN( RSA)? PUBLIC KEY/.test(key.toString())) {
      algorithm = 'RS256';
    }

    var signingMethod = algorithmMap[algorithm || header.alg];
    var signingType = typeMap[algorithm || header.alg];
    if (!signingMethod || !signingType) {
      throw new Error('Algorithm not supported');
    }

    // verify signature. `sign` will return base64 string.
    // 将 Header 和 Payload 重新生成签名，和已有的签名比对
    var signingInput = [headerSeg, payloadSeg].join('.');
    if (!verify(signingInput, key, signingMethod, signingType, signatureSeg)) {
      throw new Error('Signature verification failed');
    }

    // Support for nbf and exp claims.
    // According to the RFC, they should be in seconds.
    if (payload.nbf && Date.now() < payload.nbf*1000) {
      throw new Error('Token not yet active');
    }

    if (payload.exp && Date.now() > payload.exp*1000) {
      throw new Error('Token expired');
    }
  }
  return payload;
};
/**
 * Encode jwt
 *
 * @param {Object} payload
 * @param {String} key
 * @param {String} algorithm
 * @param {Object} options
 * @return {String} token
 * @api public
 */
// 一般用法 jwt.encode(payload, secret)
jwt.encode = function jwt_encode(payload, key, algorithm, options) {
  // 检查 key 密钥
  if (!key) {
    throw new Error('Require key');
  }
  // 检查 algorithm, default is HS256
  if (!algorithm) {
    algorithm = 'HS256';
  }
  var signingMethod = algorithmMap[algorithm];
  var signingType = typeMap[algorithm];
  if (!signingMethod || !signingType) {
    throw new Error('Algorithm not supported');
  }
  // header, typ is fixed value.
  var header = { typ: 'JWT', alg: algorithm };
  if (options && options.header) {
    assignProperties(header, options.header);
  }
  // create segments, all segments should be base64 string
  var segments = [];
  segments.push(base64urlEncode(JSON.stringify(header)));
  segments.push(base64urlEncode(JSON.stringify(payload)));
  segments.push(sign(segments.join('.'), key, signingMethod, signingType));

  return segments.join('.');
};
/**
 * private util functions
 */
function assignProperties(dest, source) {
  for (var attr in source) {
    if (source.hasOwnProperty(attr)) {
      dest[attr] = source[attr];
    }
  }
}
function verify(input, key, method, type, signature) {
  if(type === "hmac") {
    return (signature === sign(input, key, method, type));
  }
  else if(type == "sign") {
    return crypto.createVerify(method)
                 .update(input)
                 .verify(key, base64urlUnescape(signature), 'base64');
  }
  else {
    throw new Error('Algorithm type not recognized');
  }
}
function sign(input, key, method, type) {
  // HMACSHA256(base64UrlEncode(header)+'.'+base64UrlEncode(payload),secret)
  // sign(input, key)
  var base64str;
  if(type === "hmac") {
    // hmc 签名算法
    base64str = crypto.createHmac(method, key).update(input).digest('base64');
  }
  else if(type == "sign") {
    base64str = crypto.createSign(method).update(input).sign(key, 'base64');
  }
  else {
    throw new Error('Algorithm type not recognized');
  }
  return base64urlEscape(base64str);
}
function base64urlDecode(str) {
  return Buffer.from(base64urlUnescape(str), 'base64').toString();
}
function base64urlUnescape(str) {
  str += new Array(5 - str.length % 4).join('=');
  return str.replace(/\-/g, '+').replace(/_/g, '/');
}
function base64urlEncode(str) {
  // str 转为 Buffer 二进制数据，再转为 base64 串
  return base64urlEscape(Buffer.from(str).toString('base64'));
}
function base64urlEscape(str) {
  //base64 中有三个字符 + / = 在 url 中有特殊含义，因此要替换掉，= 省略、+ 替换成 -、/ 替换成_， 这就是base64URL 算法
  return str.replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
}
```

### Node crypto 模块

HMAC 散列运算消息认证码：

- 运算使用散列算法，以一个密钥 (secret)和一个消息为输入，生成一个消息摘要作为输出
- HMAC运算可以用来验证两段数据是否匹配，以确认该数据没有被篡改

在使用 HMAC 算法前，使用 createHmac 方法创建一个 hmac 对象

#### **crypto.createHmac(params, key)**

该方法中使用两个参数，第一个参数含义是在Node.js中使用的**算法**，比如'sha1', 'md5', 'sha256', 'sha512'等等，该方法返回的是**hmac对象**。第二个参数 key 参数值为一个字符串，用于指定一个PEM格式的**密钥**。

#### **hmac.update(data)**

在该方法中，使用一个参数，其参数值为一个**Buffer对象或一个字符串**，用于**指定摘要内容**。也是一样可以在被输出之前使用多次update方法来添加摘要内容。

**最后一步就是使用 hmac 对象的 digest 方法来输出摘要内容了；使用了digest方法作为输出后，因此是不能向hmac对象中追加内容**

####**hmac.digest([encoding])**

该方法有一个参数，该参数是一个可选值，表示的意思是 用于**指定输出摘要的编码格式**，可指定参数值为 'hex', 'binary', 及 'base64'. 如果不使用该参数，那么digest方法返回一个是**Buffer对象**

