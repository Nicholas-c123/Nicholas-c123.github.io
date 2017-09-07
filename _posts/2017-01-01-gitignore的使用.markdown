---
layout: post
title:  "gitignore的配置使用"
date:   2017-01-01 19:40:12 +0800
categories: 
- 开发经验
- 工具链
tag:
- git
---
# gitignore的配置使用
gitignore 是git中用来表示不需要添加到index的文件。
有了gitignore我们就可以将编译出来的文件，不需要的二进制文件，或者其他不需要的文件排除在外，然后愉快地`git add .`
但是如果有些文件已经被添加到index 那么我们需要首先 从索引中移除这些文件  
例如`git rm --cache -r target/`
然后我们在根目录下新建`.gitignore`  
下面解释一下`.gitignore`的规则  
```
/target ->忽略根目录下的target文件或者目录下的所有内容
target  ->忽略所有目录下的target文件或者目录
*.class ->忽略所有目录下的class文件
/*      ->忽略根目录下所有文件
!README.md  ->不要忽略README.md文件
```
另外注意后面的规则会覆盖前面的规则
