# 翻译背景

目前网上的airflow官方文档中文版（基于airflow 1.10.2）已经过时了，很多case在最新版中无法运行，因此开始`airflow 2.2.1`官方文档翻译工作。

# 本地运行
我们使用gitbook作为markdown文档管理工具，首先需要一些工具安装工作：
```
npm install gitbook-cli -g

gitbook -V
```

clone本项目后，首先需要安装gitbook插件：
```
gitbook install advanced-emoji

gitbook install expandable-chapters
```

然后执行`serve`命令可以本地浏览文档：
```
gitbook serve
```
本地默认访问地址是 http://localhost:4000/


# 项目地址
- https://github.com/tanhaipeng/airflow-doc-zh
- https://tanhaipeng.github.io/airflow-doc-zh


# 贡献者
- [@tanhaipeng](https://github.com/tanhaipeng)