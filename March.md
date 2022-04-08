## 0301

```
头端
61.16.140.1

尾端
61.16.140.3
入口 Eth-Trunk100
出口 25GE4/0/0

```

```
问题现象：非主用主控板上，subelb和l3elb的mcid不一致；
原因：subelb1当前的mcid为100，此subelb1(mcid=100)的删除消息丢失；
subelb1(mcid=200)update消息同步到接口板上，因为此时subelb1存在于db中，且状态为下发成功，所以认定此消息为对账，此update消息丢弃。即subelb1的mcid永远为100.

修改方案：subelb在本板扩散和删除，不再同步。
1）L3ELB在本板扩散和删除SUBELB表项；
2）资源超限反刷通过L3ELB反刷到subelb，不再直接反刷subelb，延续主控板上的行为；
3）vlan成员变化，在每个板上都通知。
4）subelb不再同步到各个板上
```

mcid 规格限制



```
# 协议命令行，统计日志，放开
reset multicast  component  message last component-type dgmp last-size 6000 list-size 4000
reset multicast  component  message last component-type mc last-size 6000 list-size 4000
reset multicast  component  message last component-type mcds last-size 6000 list-size 4000
reset multicast  component  message last component-type pimcore last-size 6000 list-size 4000
```



[告警开发流程指导~](http://3ms.huawei.com/km/blogs/details/11861441?l=zh-cn)



```
MCAST_ENTRY_ProcTrunkSubIfRefresh

trunk成员变化时，反刷子接口使能，三层trunk子接口下配的pim sm，igmp enable使能
这个使能要下icib表，芯片上没有trunk成员时不会有对应的icib
所以在trunk成员变化时要加反刷重新遍历芯片下icib
```

```
# 显示接口对端 ip
disp ip int b
# 显示邻居
display lldp neighbor brief
# 需要在两端都使能
lldp enable
```

```
目前已知问题有2个
1.如果发现cache残留了，用命令行查询时，发现wdb有fail，那是db的问题。
2.如果查询mfib和elb的db有下发失败，且errcode是3的，应该是nse的问题。
```

```
# mfib 硬表数量
dis for info enp s 1 chip 0 "lsw get oda table count name mfibv4"
```

```
typedef enum {
    FWM_OPTYPE_ADD,
    FWM_OPTYPE_MODIFY,
    FWM_OPTYPE_DELETE,
    FWM_OPTYPE_HW_VERITY_ADD,
    FWM_OPTYPE_HW_VERITY_DEL,
    FWM_OPTYPE_RSM_VERITY,
    FWM_OPTYPE_VERIFY_BEGIN,
    FWM_OPTYPE_VERIFY_END,
    FWM_OPTYPE_BUTT
} FwmOpType;

VOS_VOID MCAST_SYNC_L2MfibIncreaseDfx(VOS_UINT32 ret, FwmOpType opType)
{
    if (ret == VOS_OK) {
        MCAST_KPI_AddKpiData(opType == FWM_OPTYPE_ADD ? MCAST_L2MFIB_UPDATE_SYNC_SUCCESS :
                                                        MCAST_L2MFIB_DELETE_SYNC_SUCCESS);
        MCAST_DfxCounterInc(opType == FWM_OPTYPE_ADD ? L2ELB_STAT_SEND_SYNC_UPDATE_SUCC :
                                                       L2ELB_STAT_SEND_SYNC_DELETE_SUCC);
    } else {
        MCAST_KPI_AddKpiData(opType == FWM_OPTYPE_ADD ? MCAST_L2MFIB_UPDATE_SYNC_FAIL :
                                                        MCAST_L2MFIB_DELETE_SYNC_FAIL);
        MCAST_DfxCounterInc(opType == FWM_OPTYPE_ADD ? L2ELB_STAT_SEND_SYNC_UPDATE_FAIL :
                                                       L2ELB_STAT_SEND_SYNC_DELETE_FAIL);
    }
}

9825012a8
```

```
display error-in 10 verbose all
```

```
测试仪下载链接：1. http://10.93.173.211:4000/xstream/ 2. http://10.93.157.61:4000/GUI/
```

## 0307

![img](file:///C:/Users/g00603379/AppData/Roaming/eSpace_Desktop/UserData/g00603379/imagefiles/BCE948E6-EE06-4BC3-B74A-6FFF0904E81F.png)

```
https://codehub-y.huawei.com/DCP_DCPPlatform_Galaxy_ICP_L3/FWM-MCAST_omu_resource/merge_requests/572
修改二层命令行，bridge-domain参数未适配slot，cpu参数
```

```
# 查询能力集项
display capset slot 9 cpu 0 item MFIB_NUM
# 查看是否是大mac
display current-configuration | inc mac
# 配置大mac
system resource large-mac
```

| 城市 | 实验室 | IP   |
| ---- | ------ | ---- |
| 北京 | 绿区   | 61   |
| 南京 | 绿区   | 120  |
| 廊坊 | 绿区   | 8    |
| 廊坊 | 黄     | 65   |
| 杭州 | 黄     | 96   |
| 南京 | 黄     | 125  |
| 杭州 | 绿区   | 90   |
| 苏州 | 绿区   | 80   |
|      |        |      |

[能力集网站](http://power.rnd.huawei.com/#home.html)

```
# 关闭对账
verification disable
# fwm 关闭对账
verification fwm slot 9 cpu 0 module 26 disable
# 查询对账
display fwm verification state slot 9 cpu 0 | inc MCAST
```

## 0309

```
也需要考虑缩写的通用性与统一性：https://github.com/kisvegabor/abbreviations-in-code
有个网站，代码变量取名 https://unbug.github.io/codelf/
这个网站可以直接抓取火焰图
http://probe.openx.huawei.com/home?p=tenantList
```

```
#被强制改过期密码怎么办
system
aaa
local-aaa-user password policy administrator
undo password alert original
commit
```

如何找流水线中使用的 repo？

搜索 `repo`，找到类似 `./build.sh -a aarch64 -r /usr1/repo/xm/FWM-L3/L3_TR6_moddf_228.repo` 的命令，最后的

## 0314

```
# 整机批备状态
display switchover state
```

```
21.1  vxlan交付范围：
CE只交P5芯片，二层和三层vxlan组播
lsw只交二层vxlan
BUM全款型，具体需要看vxlan业务的支持范围
```

## 0316

```
disp igmp snoop statis bridge-domain 100
```

[R20流水线支撑人员](http://3ms.huawei.com/hi/group/3893525/wiki_5829580.html)

## 0317

```
rollback config last 1
disp warn
(libdopra.so libfwm_utils_base.so libfwm_utils_sync.so libsecurec.so ld-2.31.so)

vxlan
mlag
基础 L2 L3
nat
mcfrm
```

## 0321

```
section include xxx(SWT)

(gdb) p *mfibMissMsg 
$2 = {sourceAddr = 1677787650, groupAddr = 3774873733, l3Ifindex = 15, l2Ifindex = 0, vrId = 0, srcVrf = 4097, instanceId = {instType = FWM_L2MC_INST_BD, instId = 4294967295}, 
  srcVtepIp = 50529027, dstVtepIp = 3775070721, pktLen = 110, pktData = 0xffdc6ccecf48}
  
100.1.2.2, 225.0.0.133
3.3.3.3 225.3.2.1
```

```
display multicast component message last compnent-type [pimcore | dgmp] message-type Tnl_WrongIf
```

## 0322

```
最近定位问题 如果碰到计数能查出来这两个的 可以叫上我一起看看
Mc Oda Add L3 Oif Token Err
Mc Oda Del L3 Oif Token Err
Mc Oda Del L3 Oif PreFreeToken Err 这个可以忽略
```

```
pim sm下组播使能，未知组播上送两个字段，igmp enable和igmp snooping enable，下组播使能，未知组播上送，classid三个字段
```

```
display forward information enp slot 1 chip 0 "lsw get fe nse cmd-id 12 para1 0 para2 d"
# 查询 oda mfib 表 add/del 次数（最后一个参数 d 是 MFIBV4_VG，e 是 MFIBV4_VGS)
e.g.
display forward information enp slot 1 chip 0 "lsw get fe nse cmd-id 12 p 0 p e"
```

```
vbdif 下配置 igmp enable，则 bd 内默认使能 igmp snooping
```

```
MCAST_ELB_TOKEN_NOTIFY

bridge-domain 100
multicast layer-2 source-lifetime 5000

```

## 0324

```
01010101
DFFFFFFF


collect diagnostic information slot 收集相关板子的日志
set middleware stlm slot 6 command “logging collect” 收集某个板子的STLM的日志 
set middleware stlm all command "logging collect"

vxlan
	丁一鸣

ifm
	杨键鼐
	姜立

骆俊波
```

```
1. 需求
2. 定位
3. 质量提升
4. 团队贡献
```

## 0325

```
C 语言小技巧
git 入门
算法入门
hw cstl 简介

数通扫盲，业务流程，数据流向
debug 手段，典型问题
```

```
// 初始化
	if (FEIBRK_CFG_ClientCmdRegister(MCAST_CMD_ID_L3MC_QUERY_FWD, FEIBRK_MCAST_L3FwdRecProc) != VOS_OK) {
        FEIBRK_MCAST_CFG_LOG("[feibrk mcast] AddSetFunc failed %d.", ret);
        ret = VOS_ERR;
    }
    
    
    { CLASSID_DIMDYN_L3MCFORWARDTBL, LOCAL_PROC, 0, sizeof(CLASS_FWM_MCAST_DISMULTICASTFIB_S),
      sizeof(CLASS_FWM_MCAST_DISMULTICASTFIB_S), FEIBRK_MCAST_QueryL3Fwd, FEIBRK_MCAST_QueryL3FwdCond },
      
收到 orch 层回显，转换后发送给前台
FEIBRK_MCAST_L3FwdRecProc

发送 key 给 orch
FEIBRK_MCAST_QueryL3Fwd

get l3_forward_table_cond, assign to DBGINFO_KEY->pKey
FEIBRK_MCAST_QueryL3FwdCond
```

```
// mcast orch
FRM_ProxyCmdRegister(MCAST_CMD_ID_L3MC_QUERY_FWD, MCAST_CFG_QueryL3FwdTbl);
typedef VOS_INT32 (*McastCmdFunc)(VOS_UINT8 *request, VOS_UINT32 reqLen, VOS_UINT8 **reply, VOS_UINT32 *repLen);
```

```
disp fwm rsm re mcid a in 22573 s 9
```

## 0328

```
FEIBRK_MCAST_DisDfxInfo, FEIBRK_CFG_GetMcastDfxInfoCond
FEIBRK_MCAST_DfxInfoRecProc
```

```
McastCmdRegParam g_mcastCmdProcFunc[] = {
    {FWM_MCAST_ORCH_CMD_ID,          MCAST_CFG_FwdTblCmdProc},
    {MCAST_CMD_ID_L3MC_QUERY_FWD,    MCAST_CFG_QueryL3FwdTbl},
    {MCAST_CMD_ID_DFX_COUNTER,       MCAST_DFX_InfoQueryProc},
    {MCAST_CMD_ID_RST_DFX_COUNTER,   MCAST_DFX_RstStatProc},
    {MCAST_CMD_ID_PORT_INFO,         MCAST_CFG_QueryPortInfo},
    {MCAST_CMD_ID_HASH_CONFLICT,     MCAST_CFG_QueryHashConflict},
    {MCAST_CMD_ID_DFX_PORTGROUP,     MCAST_DFX_ProcPortGroupQuery},
};
```

## 0330

- [分布式消息中间件介绍](http://ilearning.huawei.com/edx/next/micro/course-v1:HuaweiX+CNE202203250953051-6+microcourse?blockID=3a51dd6583014774af243d583939711e)

```
以前基于ARP/ND整理过一个文档，组播也可以考虑把各种资料整合起来供大家学习参考。 @高宝儒 

数据通信/Galaxy/Docs/V100R020/V100R020C00/04 软件开发组/10 FWM_L3/ARP&ND/FWM-L3 ARP&ND模块实现说明书.docx
```

## 0331

```
25GE6/0/3
查看表项数量：
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable count"
基于mcid过滤：（最后一个数为mcid）
disp fwm-utils component fwmd slot 6 cpuid 0 "fwm coll ElbTokenTable record fid-filter 0 4 614"
查看全量表项内容（暂无翻页功能）
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable record"
填充elbkey中的非0字段过滤：
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable record key-filter [mid=518][port=67109066]"
```





## 0401

```
lldp enable

查看表项数量：
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable count"
基于mcid过滤：（最后一个数为mcid）
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable record fid-filter 0 4 518"
查看全量表项内容（暂无翻页功能）
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable record"
填充elbkey中的非0字段过滤：
disp fwm-utils component fwmd slot 1 cpuid 0 "fwm coll ElbTokenTable record key-filter [mid=518][port=67109066]"
```

```
display FES table 1470 all
disp fwm-utils component fwmd slot 6 cpuid 0 "fwm sync-rcver FWM_MCAST_Rcv_0[1.9] msg-stat-sp 135"
```

## 0408

```
# 手动触发平滑
set fei-broker frame smooth
```

```
TR6 收尾问题定位
4.12 周二维护串讲
issue
刷新

设计文档

21.1 TR5 回合 22.0
Deadline: 18 号，尽量 15 号前搞定（下周）
```

```
协议
 王恒（专家）
 刘君同
 连胜杨
 吴炜国
 李凯师
```

