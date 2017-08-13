title: DOM的节点类型NodeType
date: 2012-03-01 10:08:34
tags: 
- DOM
categories: 基础

---

总结下DOM下的节点类型：

### nodeType

根据 W3C 的 HTML DOM 标准，HTML 文档中的所有内容都是节点：

1. 整个文档是一个文档节点
2. 每个 HTML 元素是元素节点
3. HTML 元素内的文本是文本节点
4. 每个 HTML 属性是属性节点
5. 注释是注释节点

nodeType 属性可返回节点的类型。最重要的节点类型是：

| 元素类型 | 节点类型 |
|:-------:|:-----:|
| 元素element |	1 | 
| 属性attr	| 2 | 
| 文本text	| 3 | 
| 注释comments|	8 | 
| 文档document	| 9 | 


### nodeName 属性含有某个节点的名称。

1. 元素节点的 nodeName 是标签名称
2. 属性节点的 nodeName 是属性名称
3. 文本节点的 nodeName 永远是 #text
4. 文档节点的 nodeName 永远是 #document
5. 注释：nodeName 所包含的 XML 元素的标签名称永远是大写的

## nodeValue

* 对于文本节点，nodeValue 属性包含文本。
* 对于属性节点，nodeValue 属性包含属性值。
* nodeValue 属性对于文档节点和元素节点是不可用的。
