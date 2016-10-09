---
layout:     post
title:      "Markdown中如何画UML图"
subtitle:   "使用Graphviz在Markdown中画UML图"
date:       2016-10-09
author:     "BilboDai"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - Java
---

下面介绍使用[Graphviz](http://www.gravizo.com/)在Markdown中画UML图

实现原理
---
[Graphviz](http://www.gravizo.com/)提供一个HTTP接口通过传入用于绘制的脚本，返回对应的图像数据后加以渲染

支持哪些脚本
---

[Graphviz](http://www.gravizo.com/)支持

- [DOT](https://en.wikipedia.org/wiki/DOT_(graph_description_language))
- [PlantUML](http://plantuml.com/sequence-diagram)
- UMLGraph

实例
---

```
![Alt text](http://g.gravizo.com/svg?
  digraph G {
    aize ="4,4";
    main [shape=box];
    main -> parse [weight=8];
    parse -> execute;
    main -> init [style=dotted];
    main -> cleanup;
    execute -> { make_string; printf}
    init -> make_string;
    edge [color=red];
    main -> printf [style=bold,label="100 times"];
    make_string [label="make a string"];
    node [shape=box,style=filled,color=".7 .3 1.0"];
    execute -> compare;
  }
)
```

将会采用SVG的模式渲染出下面的图案

![Alt text](http://g.gravizo.com/svg?
  digraph G {
    aize ="4,4";
    main [shape=box];
    main -> parse [weight=8];
    parse -> execute;
    main -> init [style=dotted];
    main -> cleanup;
    execute -> { make_string; printf}
    init -> make_string;
    edge [color=red];
    main -> printf [style=bold,label="100 times"];
    make_string [label="make a string"];
    node [shape=box,style=filled,color=".7 .3 1.0"];
    execute -> compare;
  }
)


提供低画质的PNG格式和高画质的SVG
---
如果想采用PNG格式的话只需要把`http://g.gravizo.com/svg`替换为`http://g.gravizo.com/g`即可，支持图像效果要差好多
![Alt text](http://g.gravizo.com/g?
  digraph G {
    aize ="4,4";
    main [shape=box];
    main -> parse [weight=8];
    parse -> execute;
    main -> init [style=dotted];
    main -> cleanup;
    execute -> { make_string; printf}
    init -> make_string;
    edge [color=red];
    main -> printf [style=bold,label="100 times"];
    make_string [label="make a string"];
    node [shape=box,style=filled,color=".7 .3 1.0"];
    execute -> compare;
  }
)

更多样例请参见[Graphviz](http://www.gravizo.com/)