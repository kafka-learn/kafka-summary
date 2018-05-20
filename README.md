# kafka-summary
kafka相关知识点总结、面试真题解析、常见问题

## 环境配置

下载simiki：

```
pip install simiki
```

更新simiki：

```
pip install -U simiki
simiki update
```

生成静态文件：

```
simiki g
```

本地预览simiki效果：

```
simiki p
```

wiki部署工具配置（下面的配置大家可以不装，是直接进行wiki部署的，流程暂定提交到各自分支，大家review由我merge到主分支，再统一部署到gh-pages）

```
pip install ghp-import
```

## wiki写作

- 写作目录位于/content目录下

- 每一个带有.md文件的为wiki的一个栏目

- 图片上传问题还未解决，我会找时间完成，如果有同学有解决经验，可传达一下

## wiki部署（部署暂时由我代大家完成）

```
ghp-import -p -m "Update output documentation" -r origin -b gh-pages output
```