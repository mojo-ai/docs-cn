# 魔力文档

从 Mojo 文件编译文档字符串。

## 概要[](#synopsis)

```
mojo doc [options] <path>
```

## 描述[](#description)

这是文档工具的早期版本，可从 Mojo 代码注释生成 API 参考。目前，它会将所有文档字符串的结构化输出生成到 JSON 文件中，并且不会生成 HTML。此输出格式可能会更改。

输入必须是单个 Mojo 源文件的路径。

## 选项[](#options)

### 输出选项[](#output-options)

#### `-o <PATH>`[](#o-path)

设置 JSON 输出的路径和文件名。如果未提供，输出将写入 stdout。

### 编译选项[](#compilation-options)

#### `-I <PATH>`[](#i-path)

将给定路径附加到 Mojo 将搜索任何包/模块依赖项的目录列表。也就是说，如果您传递的文件`mojo doc`导入任何不在本地路径中且不属于 Mojo 标准库的包，请使用它来指定 Mojo 可以找到这些包的路径。

#### `--parsing-stdlib`[](#parsing-stdlib)

仅限内部使用。

### 验证选项[](#validation-options)

以下验证选项有助于确保您的文档字符串使用有效的结构并满足其他样式标准。默认情况下，仅当文档字符串包含阻止转换为输出格式的错误时才会发出警告。（稍后会提供更多选项。）

#### `--warn-missing-doc-strings`[](#warn-missing-doc-strings)

针对文档字符串缺失或部分发出警告。

### 常用选项[](#common-options)

#### `--help`,`-h`[](#help--h)

显示帮助信息。
