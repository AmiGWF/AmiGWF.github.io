# SyncXiao个人网站

捣鼓一些技术，算是一点点积累

散落一点文章，就当做闲时的干粮

***
#### 1. hexo无效，按提示安装hexo
```
npm install hexo --save
```

#### 2. 发布更新步骤
- 新建文章 ： 
```
hexo new "文章名称"
```
- 习惯性清理 ：
```
hexo clear
```
- 生成静态页面 ：
```
hexo g
```
- 开启服务，本地预览 ： 
```
hexo s （本地预览地址：localhost:4000）
```
- 部署到github ：
```
hexo d （必须部署，否则无法看到提交内容）
```
- git命令三部曲，提交到github保存
```
git add .
git commit -m "new"
git push origin hexo  （此处只提交到hexo分支即可）
```
