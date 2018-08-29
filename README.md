# dev-config
记录开发所用到的各种配置文件

## [prettier](./.prettierrc.yml)

prettier 是一个代码格式化工具。优势是考虑了代码的最大长度，对换行处理很好。而且可选的配置项很少，避免了团队里对代码风格的过多的争议。官方网站：https://prettier.io/

.prettierrc.yml：

```yaml
printWidth: 120
singleQuote: true
trailingComma: 'es5'
jsxBracketSameLine: true
```

将每列最大显示宽度 printWidth 设置为 120，这比较符合习惯，并且也看着更舒服。

对于前端，通常来说字符串会使用单引号，HTML 和 jsx 会使用双引号。

多行 object 字面量，最后一尾加上分号，这样看着更整齐。

jsx 的末尾中括号不另起一行，和 react 官方的配置相同。

语句末尾默认是加上分号的，在我观察了几个大型前端项目后，我决定不去改变它。我的习惯依然是不加分号，不过我会依赖自动格式化给语句添加分号。

### [lint-staged](https://github.com/okonet/lint-staged)

使用 lint-staged 在 git 暂存文件的时候，运行 prettier。

```js
// package.json
{
  ...
  "lint-staged": {
    "*.{js,json,css,md}": ["prettier --write", "git add"]
  },
  ...
}
```

### [husky](https://github.com/typicode/husky)

使用 husky 配置 git hook。如 `pre-commit pre-push`。我们配置 pre-commit 运行 prettier

```js
// package.json
{
  ...
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  ...
}
```

## [gitignore](./.gitignore)

不需要添加到 git 版本库的文件，基于 create-react-app，做了一些自己的定制

```bash
# dependencies
/node_modules

# testing
/coverage

# production
/build

# misc
.DS_Store
.env.local
.env.development.local
.env.test.local
.env.production.local

# editor
/.vscode/settings.json
/.idea

npm-debug.log*
yarn-debug.log*
yarn-error.log*
```

## HTTPS

证书我们选用 Let's Encrypt。按照官方教程，推荐使用 [certbot](https://certbot.eff.org/) ACME client。certbot 可以自动进行证书的颁发和安装。

按照 certbot 的官网：https://certbot.eff.org/，选择服务器 nginx 和操作系统 ubuntu

![](./assets/certbot.png)

执行以下命令

![](./assets/certbot1.png)

输入 `sudo certbot --nginx` 自动配置 nginx。

它将会对 nginx 新增如下类似的配置：

```bash
server {
  ...
  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/bbs.chainx.org/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/bbs.chainx.org/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

## [nvm](https://github.com/creationix/nvm)

curl 下载并执行安装脚本

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```

在 ~/.bashrc（会自动添加）或者 ~/.zshrc（如果用的是 zsh）中添加下列配置

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

使用 `source ~/.bashrc` 或 `source ~/.zshrc` 可以不重启终端，使 nvm 命令生效。

下载最新版 node

```bash
nvm install node
```

使用指定版本

```bash
nvm use 8.0
```

安装时，迁移旧版本 node 的 node_module

```bash
nvm install 6 --reinstall-packages-from=5
```

在不同项目之间切换版本：添加 [.nvmrc](./nvmrc) 文件。

```bash
#.nvmrc
8.11.4
```

此后在该目录下使用如：`nvm use`，`nvm install` 等命令，会自动指定 nvmrc 中的所指定的版本。

## gzip

使用 nginx 开启 gzip。

官方文档： http://nginx.org/en/docs/http/ngx_http_gzip_module.html

配置：

```bash
# 是否开启gzip
gzip on;

# User-Agent 如何和该正则表达式匹配，则不禁止 gzipping
gzip_disable "msie6";

# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1024;

# 针对代理请求，可以为 off | expired | no-cache | no-store | private | no_last_modified | no_etag | auth | any ...;
# any 表示所有代理请求都开启 gzip，no-cache 代表只有 包含 no-cache 请求头开启。
gzip_proxied any;

# gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间
gzip_comp_level 5;

# 进行压缩的文件类型。其中的值可以在 mime.types 文件中找到。图片不要通过 gzip 压缩，没有意义。
gzip_types
    text/css
    text/javascript
    text/xml
    text/plain
    text/x-component
    application/javascript
    application/x-javascript
    application/json
    application/xml
    application/rss+xml
    application/x-font-ttf
    application/vnd.ms-fontobject
    image/svg+xml
    svg
    svgz;

# 是否在http header中添加Vary: Accept-Encoding
gzip_vary on;
```

## [nodemon](https://github.com/remy/nodemon)

命令行基本用法

```shell
nodemon [options] [script.js] [args]
```

部分参数：

- -e, --ext 文件后缀名
- -x, --exec 需要执行的命令，如 `nodemon --exec node index`
- -w, --watch 需要监控的目录或者文件
- -i, --ignore 需要忽略的目录或文件

例子：

```bash
$ nodemon server.js
$ nodemon -w ../foo server.js apparg1 apparg2
$ PORT=8000 nodemon --debug-brk server.js
$ nodemon --exec python app.py
$ nodemon --exec "make build" -e "styl hbs"
$ nodemon app.js -- -V
```

## tslint

## ts config

## jest

## yarn

## babel

## postcss

## LICENSE

## package

## editorconfig

## chrome 扩展

## vs 扩展

## webpack

## vscode-chrome-debug
