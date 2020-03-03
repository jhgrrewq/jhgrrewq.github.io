## 搭建 npm 私服原因

- 统一管理（公司内部开发的私有包，方便统一管理、开发和使用）
- 安全性（由于公司内部开发的模块等并不希望公开, 但又希望内部能方便使用）
- 加速（自建服务器自带常用 package 的缓存，会缓存下载过的包，节省时间）

## 使用verdaccio

verdaccio 是 sinopia 开源框架的一个fork，由于sinopia 已不再维护，因此采用 verdaccio

### 安装

```bash
npm i –g verdaccio
```

### 运行

```bash
verdaccio

 warn --- config file  - /home/admin/.config/verdaccio/config.yaml # 默认的配置文件
 warn --- Verdaccio started
 warn --- Plugin successfully loaded: verdaccio-htpasswd
 warn --- Plugin successfully loaded: verdaccio-audit
 warn --- http address - http://localhost:4873/ - verdaccio/4.4.0 # 默认端口
```

可以使用 pm2 来管理进程

```bash
npm i -g pm2
pm2 start verdaccio
# pm2 ls 查看进程情况
```

### 腾讯云服务器配置 nginx

verdaccio 默认是启动在 4873 端口，腾讯云服务器需要配置 nginx 反向代理到该端口，nginx 配置需要在 `/etc/nginx/conf.d/` 下新建文件

```bash
server {
  listen 80;
  server_name registry.npm.your.server;
  location / {
    proxy_pass              http://127.0.0.1:4873/;
    proxy_set_header        Host $host;
  }
}
```

重启 nginx

```bash
sudo nginx -s reload
```

### verdaccio 配置 （/home/admin/.config/verdaccio/config.yaml）

- verdaccio 默认使用的是 npm官方的源，可改成淘宝的源
- 私有npm 里面不包含的包, 例如要安装一个vue  包, 找不到的话, 会被代理到默认 npm 源仓库去下载, 并且会缓存在 ./storage 文件夹

```bash
# https://github.com/verdaccio/verdaccio/tree/master/conf

# path to a directory with all packages
# 所有包缓存目录
storage: ./storage
# path to a directory with plugins to include
# 插件目录
plugins: ./plugins

# 开启 web 服务,能够通过 web 访问
web:
  title: Verdaccio
  # comment out to disable gravatar support
  # gravatar: false
  # by default packages are ordercer ascendant (asc|desc)
  # sort_packages: asc

# 验证信息
auth:
  htpasswd:
    file: ./htpasswd
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    # max_users: 1000

security:
  api:
    jwt:
      sign:
        expiresIn: 60d
        notBefore: 1
  web:
    sign:
      expiresIn: 7d
      notBefore: 1

# a list of other known repositories we can talk to
# 公有仓库配置
uplinks:
  npmjs:
  	# url: https://registry.npmjs.org/
    url: https://registry.taobao.org/

packages:
  '@*/*':
    # scoped packages
    # 代理 表示没有的仓库会去 npmjs 里面去找 
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs

  '**':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish/publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated
    unpublish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

# You can specify HTTP/1.1 server keep alive timeout in seconds for incomming connections.
# A value of 0 makes the http server behave similarly to Node.js versions prior to 8.0.0, which did not have a keep-alive timeout.
# WORKAROUND: Through given configuration you can workaround following issue https://github.com/verdaccio/verdaccio/issues/301. Set to 0 in case 60 is not enought.
server:
  keepAliveTimeout: 60

middlewares:
  audit:
    enabled: true

# log settings
logs:
  - { type: stdout, format: pretty, level: http }
```

## 使用

- 切换源

```js
# 使用 nrm 切换源
npm i -g nrm
nrm add jhgrrewq http://registry.npm.jhgrrewq.com/
nrm use jhgrrewq
```

- 注册用户（或登录）

```js
npm adduser / npm login
# 输入用户名密码
```

- 发布

```js
# npm publish 标准编写包并发布
npm publish
```

