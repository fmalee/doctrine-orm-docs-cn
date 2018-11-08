# Doctrine ORM文档

Github：https://github.com/doctrine/doctrine2/tree/master/docs

## 同步记录

- `2.7`
  - `01cff97edae8d7d80cc8fb3ac6804cdb7cf9a4d5`（2018-09-23）

> `2.7` 和 `2.6` 的文档保持一致，所以直接翻译 `2.7` 分支
>
> `master` 为全新的 3.0 版本，文档和之前的版本完全不同

## 术语约定

- `Meatdata`：元数据
- `Schema`：模式
- `Repository`：仓库
- `Entity`：实体
- `Entity Manger`：实体管理器
- `Query Builder `：查询生成器

## 生成主题

### 如何生成

参考Symfony文档的说明

### 主题问题

如果出现“主题错误”，请检查 `en/_theme` 子目录是否为空，
在这种情况下，你将需要运行：

```bash
# 初始化submodule
$ git submodule init
# 更新submodule
$ git submodule update
```