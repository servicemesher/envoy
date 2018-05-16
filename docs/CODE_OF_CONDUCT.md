# 翻译规范

本文中指定了该文档的翻译规范。

翻译进度可以通过 [Project](https://github.com/servicemesher/envoy/projects/1) 查看。

## 文档组织规则

- 所有文档使用 markdown 格式撰写
- 文档目录严格按照英文官方文档组织
- 所有图片都放在最顶层的 `images` 目录下
- API 参考章节直接链接到英文文档

## 文档规范

- 所有的英文跟中文之间要有一个空格

## 翻译流程

1. 首先加入到 ServiceMesher 组织中，请联系 [Jimmy Song](https://jimmysong.io/about) 加入。
2. 在 [Issues](https://github.com/servicemesher/envoy/issues) 中找到你想翻译的文档的标题（对应的文档路径即为标题），在 issue 中回复“认领”，并将该 issue 指派给自己。
3. 创建一个翻译分支，翻译完成后提交 PR。
4. 由 owner 审核后 merge 进 master 分支。

### 专有名词

下面的词汇作为专有名词，不需要翻译。

- filter
- listner
- Cluster manager
- CDS
- RDS
- LDS
- CORS