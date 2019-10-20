# whblogsource
hexo blog source


# how use

**refer to:** https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md

**change the file _config.xml:**
first create the reposity: your_github_usrname.github.io
```yml
deploy:
  type: git
  repository: https://github.com/your_github_usrname/your_github_usrname.github.io
  branch: master
```
then run the following command in terminal(git bash or cmder  etc. on windows or other os)

```bash
git clone git@github.com:njustwh2014/whblogsource.git

cd whblogsource

npm install -g hexo

npm install

npm install hexo-deployer-git --save

npm install hexo-generator-feed --save

npm install hexo-generator-sitemap --save

hexo g

hexo d
```



