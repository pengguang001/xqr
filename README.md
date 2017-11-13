# 什么是XQR？
XQR(XiangQi Replay)是一种全新的象棋棋谱格式，它有以下特点：
1. 二进制
2. 支持变着
3. 不加密
4. 带32位CRC检查
5. 精简，节省存储空间

# 为什么需要XQR？
目前流行的棋谱格式有诸多缺点：
* PGN - 文本格式，不支持变着，存在歧义，计算机处理起来不方便
* XQF - 加密的格式，设计比较混乱。另外，红黑选手名称只支持16字节（8个汉字）
* CBR - 私有格式，不开放

# XQR格式
XQR文件由多个TLV（type-length-value）序列，Type和Length各占一字节，Value部分长度由Length字段决定。XQR文件包含以下内容：
* 棋谱信息
* 着法
* CRC

1. 棋谱信息，一个或多个TLV。如果Value存放的是字符串，则以UTF8格式编码。目前定义的Type有：
* TYPE_MAGIC = 0
* TYPE_VERSION = 1
* TYPE_EVENT = 2
* TYPE_SITE = 3
* TYPE_DATE = 4
* TYPE_RED = 5
* TYPE_BLACK = 6 
* TYPE_RESULT = 7
* TYPE_FEN = 8
* TYPE_MOVE = 9
* TYPE_CRC = 10

第一个TLV为MAGIC数据，length为2，value部分必须为{0x20, 0x17}，否则这是一个非法的XQR文件。

如果碰到TYPE_MOVE的TLV，则棋谱信息部分结束。TYPE_MOVE为一个特殊的TLV，Length字段为0，后面跟的是着法数据。

2. 着法部分仅包含一个TLV，Type为TYPE_MOVE。由于此时不能确定Value部分即着法数据的总长度，Length为0。Value部分为一棵以左孩子右兄弟方式存储的树，每个节点表示一个着法。如果不包含评论，每个节点存储4字节数据：
```
[B0][B1][B2][B3]
```

B0和B1分别为FROM和TO，表示棋子的源位置和目的位置。高4位表示哪一行（0-9，从上到下递增），低4位表示哪一列（0-8，从左到右递增）。B2字节为FLAG，b0位置1表示有孩子节点，b1位置1表示有兄弟节点，b2位置1表示包含评论。B3字节为0，保留使用。

如果节点包含评论（B2b2=1），则后面跟的4字节表示评论长度，以Little-Endian存放，再往后跟的则是以UTF8编码的评论数据。
```
[B0][B1][B2][B3][B4][B5][B6][B7]comment...
```

树的根节点表示初始盘面，它的B2b1位不能为1（根节点不会包含兄弟节点）

树的存放方式：
```
saveTree(Node node) {
    save(node->data)
    if (有孩子节点）{
        saveTree(node->child);
    }

    if (有兄弟节点）{
        saveTree(node->sibling);
    }
}
```

3. 最后一个TLV为CRC数据，Length为4字节，Value部分为4字节的CRC校验，它覆盖从文件的第一个字节到本TLV的Type之前的那个字节的所有数据。

# PGN转XQR

# XQF转XQR

# XQR转PGN

# XQR转XQF
