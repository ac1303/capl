# CAPL工具库 V2.0

- 基于osektp.dll封装诊断相关函数
- 对常用功能，例如打印、output等函数进行增强

## 功能
- [x] 诊断发送与接收

- [ ] 断言（部分完成）

- [ ] 文件解析（部分完成）

- [ ] 封装故障注入

- [ ] 测试报告美化

- [x] 打印增强

- [ ] 常用测试函数

- [ ] ...

## 最近更新

* 2023年9月8日更新2.0版本
  - [x] 对诊断数据存储结构进行优化
  - [x] 修复发送多帧情况下诊断响应超时的情况
  - [x] 增加logging相关函数，美化打印
  - [x] 优化outputPro相关逻辑

- 2023年8月20日 更新节点丢失相关DTC检测工具（demo）
- 2023年8月17日 更新诊断无应答相关函数

  

------

  

# 快速开始

> 本工具与osektp.dll强关联，请务必在节点中配置osektp.dll

> 如果要使用27解锁相关函数，请确保CANoe诊断面板内能正常解锁

### 创建连接

```c
variables
{
  long handle;
  char ecuName[10] = "RLDHC";
  byte lv1=0x01;
  byte lvFbl = 0x09;
}
MainTest(){
  handle = CreateCANFDConnection(0x746,0x74E);
    // or CreateCANConnection(0x746,0x74E);
}
```

### 发送诊断和断言

```c
//发送10 01
sendDiag1001(); 
// 判断应答是否为肯定应答
AssertDiagTrue(); 
// 判断响应数据第4个byte是否为0
AssertDiagCompareBytes(4,0x0);
// 发送密钥0000
sendDiag27_key(handle,lv1,0x00000000);
// 判断是否为否定应答，且应答码为0x24
AssertDiagFalseWithNRC(0x24);
// 解锁密钥
sendDiag27_UnlockSecurityLevel(handle,lv1,ecuName);

// 22服务读取DID
int len;
byte buffer[100];
sendDiag22xxxx(diagHandle,0x0200);
len = getDiagResp(buffer); // 获取诊断响应数据
```

```c
testcase DFRTC005(){
  word segmentSize,len;
  byte buffer[100];
  
  testCaseTitle("DFRTC005","刷新块大小参数测试");
  testCaseDescription("校验 ECU 刷新块大小参数的准确性");
 
  Pre_Programming();
  if (TestGetVerdictLastTestCase() == 1)
    return;
  sendDiag1002(diagHandle);
  AssertDiagTrue();
  sendDiag27_UnlockSecurityLevel(diagHandle,lvFbl,ecuName);
  sendDiag(diagHandle,q2EF184);
  AssertDiagTrue();
  if (TestGetVerdictLastTestCase() == 1)  //如果测试用例已经失败，则停止继续执行
    return;
  memcpy_off(q340044,3,gacDriverBin.AddressRegion[0],0,8);
  sendDiag(diagHandle,q340044);
  AssertDiagTrue();
  len=getDiagResp(buffer); // 获取34应答中的传输数据大小
  segmentSize = buffer[len-2];
  segmentSize <<= 8;
  segmentSize +=buffer[len- 1];
  // 测试传输数据超过预期
  sendDiag36(diagHandle,gacDriverBin.fileBlock[0],gacDriverBin.fileBlockLen[0],0,segmentSize+2);
  AssertDiagFalseWithNRC(0x13);
  //测试传输数据是预期的一半
  fbl(gacDriverBin,gacAppBin,(segmentSize/2+1));
  //测试传输数据大小为128
  fbl(gacDriverBin,gacAppBin,130);
}
```



## ？

如果有问题可以提交pr , 一定会改，但是时间不好说。

由于才工作几个月，经验严重不足，代码中绝对有bug和不合理的地方，希望大佬能指点。

如果有小伙伴和我一起完善那就更好了，[许愿.jpg]