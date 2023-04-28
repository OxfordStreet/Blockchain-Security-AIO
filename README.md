# Blockchain-Security-AIO

## 资料缓冲区

[资料缓冲区文章视频链接只管扔进来] 这个文件的组织结构是csv，如果要本地查看可以添加「.csv」后缀。

1. csv中字段之间使用英文逗号（comma）隔开，
2. 如果多个项存在同一字段中，项与项之间可以使用'/'做分隔；
3. 资料缓冲区NumID字断是由Term字断经过*SHA1算法*散列而来，唯一标识一个item。关于此项的各种相关内容都可以带上NumID方便全局查找。
4. 在vs code中快速将Term哈希成NumID:https://marketplace.visualstudio.com/items?itemName=PhilippT.voop
