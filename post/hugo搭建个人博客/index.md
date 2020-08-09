在win10上利用Hugo+github搭建个人博客包括以下步骤：
1. 安装Hugo(https://gohugo.io/getting-started/installing)
2. 添加主题Even(https://github.com/olOwOlo/hugo-theme-even)
3. 编写批处理文件，从而快速、便捷地创建新的博客，批处理代码如下：

		@echo off
		set /p name=FileName:  ::从键盘输入文件名，需要带后缀”*.md“
		hugo new post/%name%   
		notepad++.exe content/post/%name%  ::需要下载notepad++，并配置环境变量

4. 执行以下命令生成pubic文件夹，其中包含了该站点的静态页面
	- `hugo -t even`  // even 是主题名
	- 'hugo -D` //生成post文件夹
5. 在github新建一个名字为`username.github.io`的respository，并执行以下命令将静态页面传至github
	- `cd public`
	- `git init`
	- `git remote add origin repositoryaddress`
	- `git add .`
	- `git commit -m "origin`
	- `git push origin master`	