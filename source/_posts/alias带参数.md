---
title: alias 如何带参数
date: 2019-11-21 21:11:14
---



比如我要使用**python**的虚拟环境，我想要激活一个叫做 **flask36** 的环境，每次都要使用

```bash
source flask36/bin/activate
```

好麻烦，所以使用了一个`alias`来简化这个操作

```bash
alias ac='source flask36/bin/activate'
```

然后输入`ac`就可以了

但是这样的话，当使用其他环境时就不能用这个命令了

所以想要带个参数，直接用又不行，因为这个`flask36`这个东西要放在中间

所以研究了一下，可以这样用

```bash
alias ac='func() { source $1/bin/activate; }; func'
```

然后就可以带参数了，比如：

```bash
ac flask36
```

然后如果在不同的路径下也可以使用 ，比如

```bash
ac the/path/of/the/virtual/env/flask36
```

