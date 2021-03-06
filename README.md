# 安装  
  
## 安装前提  

- Node.js
- Git

## 安装说明

```bash
$ npm install -g hexo-cli
```


# 建站

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

### 目录结构   

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```


# 生成文章并发布


```bash
$ hexo clean
$ hexo generate --deploy
$ hexo deploy --generate

# 可简写为
$ hexo g -d
$ hexo d -g
```


# 发布到Github配置

### 申请github域名
在github上新建工程，名字固定[用户名.github.io]   
比如我的用户名是blanker，工程名就必须是blanker.github.io


### 生成ssh公钥
   
先设置git选项
```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```
   
生成ssh密钥文件：
```bash
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```
然后需要把公钥注册到github相应的ssh key那里


### 安装 hexo-deployer-git。
```bash
$ npm install hexo-deployer-git --save
```

### 配置_config.yml 
```yaml
deploy:
  type: git
  repo: git@github.com:blanker/blanker.github.io.git
  branch: master
```


# 本地服务器

首先安装hexo-server
```bash
$ npm install hexo-server --save
```

运行服务器，默认端口是4000，也可以使用-p 参数指定端口
```bash
$ hexo server
```

# Next主题使用

```bash
git submodule add https://github.com/theme-next/hexo-theme-next.git themes/next
```

修改配置
```yaml
theme: next
```


# 同步hexo工程到github上

首先新建一个工程，叫做hexo
然后把本地的hexo工程推送到远程

```bash
git init
git submodule add https://github.com/theme-next/hexo-theme-next.git themes/next
git add .
git commit -m 'init'

git remote add origin git@github.com:blanker/hexo.git
git push -u origin master
```

* 以上步骤第一遍完成后，后面就不需要做了

* 其他电脑的使用

```bash
git clone git@github.com:blanker/hexo.git --recursive
cd hexo
npm install -g hexo-cli
npm install
```

