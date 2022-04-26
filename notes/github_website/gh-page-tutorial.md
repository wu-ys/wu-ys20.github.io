# 教程：如何在GitHub上建立自己的个人网站/主页



## 0 你需要的准备

1. 拥有一个Github账号
2. 了解Git的相关知识和操作（如git分支, `git init`, `git add`, `git commit`, `git push`等命令；如果不会的话也可以用`GitHub Desktop`，或者VSCode的插件等图形化界面进行操作）
3. 了解基础的Markdown语法，或者用Typora等markdown文档编辑器；当然，愿意自己写html的话也可以

## 1 建立一个Github库

1.1 为了建立自己的主页网站，需要先在Github上建立**指定名称**的库，这里的指定名称为：`<user-name>.github.io`，其中`<user-name>`替换为你自己的Github用户名（在左侧的Owner一览可见），权限应该为public

![image-20220426214533202](https://wu-ys.github.io/notes/github_website/gh-page-tutorial.assets/image-20220426214533202.png)

1.2 创建成功后，点击Settings-pages，将Source一览设置为gh-pages分支，root文件夹。点击下方的Change theme按钮，可以设置默认生成器的网页样式（生成器的作用会在下面进行介绍）

![image-20220426214920844](https://wu-ys.github.io/notes/github_website/gh-page-tutorial.assets/image-20220426214920844.png)



1.3 点击Code页面上的Code绿色按钮，进行git clone操作。比较建议的操作方式如下：

1. 在https标签下点击链接旁边的按钮，将链接的url复制到剪贴板

![image-20220426215204086](https://wu-ys.github.io/notes/github_website/gh-page-tutorial.assets/image-20220426215204086.png)

2. 在本地建立用于存储相关文件的文件夹，在终端中打开，输入以下的命令：

```shell
git init
git clone <url>  # https://github.com/<user-name>/<user-name>.github.io.git
```

其中`<url>`就是你刚刚复制的链接地址，如果之前操作无误的话，应该和注释中的地址一致。执行完毕后输入命令`git status`, 如果输出为（或类似于）

```
On branch gh-pages
Your branch is up to date with 'origin/gh-pages'.
```

你就可以尝试在本地commit后使用`git push`命令，将本地的修改同步到Github上啦！



## 2 Github-pages的网页生成机制

1. 主页：通过地址 https://user-name.github.io 访问。如果库的根目录下有名为index.html的文件，则自动将这一网页作为主页；否则如果有名为index.md的文件，则使用上面提到的生成器，将md文件生成为网页并作为主页。

2. 其他页面：通过从库的根目录开始的路径访问。例如，有一个文件的路径为 (root)/a/b/c/d/e/f/g/h.html，则应该通过地址 https://user-name.github.io/a/b/c/d/e/f/g/h.html 访问这一页面。通过这一方式，你便可以正确地建立不同网页之间的链接了。

3. 图片：很多时候需要在网页中插入图片。只使用本地的图片地址当然是不行的，但可以使用外链图床设置图片地址；如果你懒得弄图床的话，也可以将图片一并放进库里并上传到Github库里，使用上一条的相同方式（将后缀名改为png jpg等）即可。

   例如，对于下面这张图片，将文件中的访问地址改为https://wu-ys.github.io/courses/architecture/midterm-cheatsheet.assets/image-20220418200247416.png 就可以正确建立链接了。

![image-20220426222017669](https://wu-ys.github.io/notes/github_website/gh-page-tutorial.assets/image-20220426222017669.png)

 

4. pdf文件（未经验证）：链接方式同上。



## 3 从头开始写网页——以使用Typora编辑为例

这里我们推荐的方法是，主页（即index.md）直接上传.md文件，使用在线生成器生成一个比较好看的主页

对于其他的网页（特别是内容比较多的页面），编辑一个.md文件，在Typora中打开并调整到合适的样式，导出为.html，作为一个网页文件上传到Github库。这种方法可以支持目录、数学公式等，所有Typora支持的附加功能。（而之前提到的在线生成器是不支持的）

如果希望自己在Typora定制网页样式的话，可以点击“文件-偏好设置-外观”，在下面点击“打开主题文件夹”，新建一个.css文件并在其中根据自己的喜好修改。不会的话可以上网搜一搜相关教程。

![image-20220426223348605]((https://wu-ys.github.io/notes/github_website/gh-page-tutorial.assets/image-20220426223348605.png)