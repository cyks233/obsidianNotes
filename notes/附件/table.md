# QOS
## IP（TOS）

|  值  |  解释  |                 |          |
| :-: | :--: | :-------------: | -------- |
| 111 | 网络控制 | network control | 网络数据使用   |
| 110 | 网间控制 |  internetwork   | 网络数据使用   |
| 101 |  关键  |    critical     | 语音数据     |
| 100 |  急速  | flash-override  | 视频会议和视频流 |
| 011 |  闪速  |      flash      | 语音控制数据   |
| 010 |  快速  |    immediate    | 数据业务     |
| 001 |  优先  |    priority     | 数据业务     |
| 000 |  普通  |    routinue     | 缺省       |

|        | 低丢弃优先级 | 中丢弃优先级 | 高丢弃优先级 |
| ------ | ------ | ------ | ------ |
| class4 | AF41   | AF42   | AF43   |
| class3 | AF31   | AF32   | AF33   |
| class2 | AF21   | AF22   | AF23   |
| class1 | AF11   | AF12   | AF13   |

^5db625


