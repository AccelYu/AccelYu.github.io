---
layout: post
title: 在github上搭建hexo博客
categories: [Hexo,Github]
---
使用hexo和NexT主题
<!-- more -->
### 创建仓库
1. 新建一个名为“你的用户名.github.io”的仓库，直接点击“Create repository”按钮

2. 进入仓库，根据github上的提示创建master分支和第一次提交

### hexo
1. 安装hexo
   ```
   npm install hexo -g
   ```

2. 更新hexo
   ```
   npm update hexo -g
   ```

3. 初始化
   ```
   cd 目标文件夹 # 本文之后的命令全在目标文件夹内执行
   hexo init
   ```

4. 生成静态网页
   ```
   hexo g # 改动后需执行
   ```

5. 部署
- github配置ssh密钥
   ```
   ssh -T git@github.com # 测试添加ssh是否成功
   ```
- 修改hexo的_config.yml
   ```yml
   deploy:
      type: git
      repository: git@github.com:xxx/xxx.github.io.git
      branch: master
   ```
- 部署本地站点到github
   ```
   npm install hexo-deployer-git --save # 需安装插件，只要执行一次
   hexo clean # 清除本地 public 文件
   hexo deploy # 部署
   ```

### NexT主题
1. 下载主题
   ```
   git clone https://github.com/theme-next/hexo-theme-next themes/next
   ```

2. 切换主题  
修改hexo的_config.yml
   ```yml
   theme : next
   ```

3. 安装搜索框插件
   ```
   npm install hexo-generator-searchdb --save
   ```