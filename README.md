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
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,13.229.188.59' (RSA) to the list of known hosts.
Hi Cheers0606! You've successfully authenticated, but GitHub does not provide shell access.
```

## hexo 常用操作

