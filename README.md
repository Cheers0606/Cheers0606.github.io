# cheer_blog
hexo 博客 源码


## init
```bash
# 需要先安装git
git clone https://github.com/Cheers0606/cheer_blog.git
# 需要先安装好nodejs、npm和hexo模块
# npm install -g hexo-cli
rm -rf node_modules && npm install --force
hexo g
hexo server
```

## 配置github deploy
```bash
## git config user.name "enoch"
## git config user.email "cheers0606520@gmail.com"
ssh-keygen -t rsa -C "cheers0606520@gmail.com"
cat id_rsa.pub
## 复制id_rsa.pub
## 到github上 --> settings --> SSH and GPG keys --> New SSH key  粘贴
## 测试是否配置成功
➜ ssh -T git@github.com
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:xxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,xxx' (RSA) to the list of known hosts.
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.

# 修改_config.yml的deploy配置
# 安装deploy-git模块
npm install hexo-deployer-git --save

hexo clean
hexo generate
hexo deploy

```

## hexo 常用操作
```bash
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
hexo clean #清理public的内容
```
