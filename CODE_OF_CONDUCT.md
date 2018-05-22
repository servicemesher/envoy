# 规范

本文中指定了该文档的翻译规范与注意事项。

翻译进度可以通过 [Project](https://github.com/servicemesher/envoy/projects/1) 查看。

## 文档组织规则

- 所有文档使用 markdown 格式撰写。
- 文档目录严格按照英文官方文档组织。
- 所有图片都放在最顶层的 `images` 目录下。
- API 参考章节直接链接到英文文档。

## 翻译流程

1. 首先加入到 ServiceMesher 组织中，请联系 [Jimmy Song](https://jimmysong.io/about) 加入。
2. 在 [Issues](https://github.com/servicemesher/envoy/issues) 中找到你想翻译的文档的标题（对应的文档路径即为标题），在 issue 中回复“认领”，确认不要与他人认领的重复。
3. 创建一个翻译分支，翻译全文，删除原文中的英文部分，保留所有可达的链接，翻译完成后提交 PR。
4. 由 owner 审核后 merge 进 master 分支。

## 注意事项

- 一次性不要认领太多文章，最多不要超过三篇，如果一周内没有提交则取消指派，可以由其他人翻译。
- 文中有些地方的链接可能无法到达，尤其是指向 API v1 和 API v2 参考文档的链接，因为代码库中没有加入这部分文档，所有当遇到这种情况时请直接使用官方文档上可抵达的互联网链接。

## 翻译规范

- 所有的英文跟中文之间要有一个空格

## 专有名词

下面的词汇作为专有名词，不需要翻译。

- CDS
- RDS
- LDS
- CORS

以下词汇的翻译统一。

- pluggable：可拔插的
- cluster：集群
- load balancer：负载均衡器