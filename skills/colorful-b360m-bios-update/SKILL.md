---
name: colorful-b360m-bios-update
description: This skill should be used when the user asks about "七彩虹 BIOS更新", "Colorful motherboard BIOS flash", "B360M-D BIOS upgrade", "七彩虹主板刷BIOS", "colorful bios update", or mentions upgrading CPU on a Colorful Battle Axe B360 series motherboard and needing a BIOS update. Also applies when the user has issues with Intel FPT Error 3, FreeDOS boot USB, or BIOS flash descriptor security on consumer Intel boards.
version: 1.0.0
---

# 七彩虹 Battle Axe B360M BIOS 更新指南
# Colorful Battle Axe B360M BIOS Update Guide

适用主板 / Applies to:
- 七彩虹 战斧 C.B360M-D 魔音版 V20
- Colorful Battle Axe C.B360M-D / C.B360M-F / C.B360M-HD series

## 背景说明

七彩虹B360M系列主板出厂BIOS（2018年版）不包含Intel 9代CPU微码，换装i7-9700F等9代CPU后必须更新BIOS，否则会出现频繁重启、热保护关机等问题。

该主板不支持标准EZ Flash方式，必须使用Intel FPT工具通过DOS启动盘刷写。

## 第一步：诊断重启原因

运行以下命令确认是否为硬件崩溃（BugcheckCode=0）：

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Id=41,6008} -MaxEvents 10 | ForEach-Object {
    $xml = [xml]$_.ToXml()
    Write-Host "时间:" $_.TimeCreated
    $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'BugcheckCode'} | ForEach-Object {
        Write-Host "BugcheckCode:" $_.'#text'
    }
}
```

BugcheckCode=0 且连续崩溃间隔越来越短 → 确认为过热或BIOS兼容性问题。

## 第二步：下载BIOS文件

1. 访问：`https://www.colorful.cn/home/DownLoadInfo?mid=84&id=727`
2. 点击「BIOS」标签下载最新版本（1006版，2019/05/29）
3. 解压得到以下结构：
   ```
   1006/
   ├── Dos/
   │   ├── 90529030.BIN    ← BIOS固件
   │   ├── FPT.exe         ← DOS版刷写工具
   │   ├── fparts.txt
   │   └── update.bat
   └── Windows64/
       ├── 90529030.BIN
       ├── FPTW64.exe      ← Windows版（受安全限制，通常失败）
       └── update.bat
   ```

## 第三步：为何Windows方式失败

Intel FPT在消费级主板上受Flash描述符安全限制，Windows下的FPTW64.exe会报：
```
Error 3: Internal Error. Unexpected error occurred.
```

必须使用DOS版FPT，通过底层硬件访问绕过此限制。

## 第四步：制作FreeDOS启动U盘

1. 下载Rufus便携版（约2MB）：
   ```
   https://github.com/pbatard/rufus/releases/download/v4.13/rufus-4.13p.exe
   ```
   或使用PowerShell下载：
   ```powershell
   Invoke-WebRequest -Uri 'https://github.com/pbatard/rufus/releases/download/v4.13/rufus-4.13p.exe' -OutFile 'C:\Users\$env:USERNAME\Desktop\rufus.exe' -UseBasicParsing
   ```

2. Rufus配置：
   - 引导类型：**FreeDOS**
   - 分区类型：**MBR**
   - 文件系统：**FAT32**
   - 点「开始」完成格式化

3. 将Dos文件夹中4个文件复制到U盘根目录：
   ```powershell
   $dosDir = "C:\Users\$env:USERNAME\Desktop\1006\Dos"
   $usbDrive = "F:"  # 根据实际盘符修改
   Copy-Item "$dosDir\FPT.exe", "$dosDir\90529030.BIN", "$dosDir\fparts.txt", "$dosDir\update.bat" -Destination "$usbDrive\"
   ```

## 第五步：修改BIOS设置

进入BIOS（开机按DEL键）→「启动」选项卡：

| 选项 | 刷BIOS前改为 | 刷完后改回 |
|------|------------|----------|
| 引导模式选择 | **LEGACY** | UEFI |
| ME子系统状态 | Enabled（默认）| Enabled |

> 注意：引导模式必须改为LEGACY，否则FreeDOS MBR启动盘无法引导。

F10保存退出。

## 第六步：从U盘启动并刷写

1. 重启，按**F11**进启动菜单
2. 选U盘（不带「UEFI:」前缀的那个）
3. 进入FreeDOS后，切换到U盘：
   ```
   A:
   dir
   ```
   确认能看到 `UPDATE.BAT`、`FPT.EXE`、`90529030.BIN`

4. 执行刷写：
   ```
   update.bat
   ```
   等待完成，全程不要断电（约1-2分钟）

## 第七步：刷写后恢复

1. 重启，进BIOS（DEL键）
2. 找「Language」改回中文
3. `Exit` → `Load Optimized Defaults` → 确认
4. 将「引导模式选择」改回 **UEFI**
5. F10保存退出

## 验证更新成功

```powershell
$bios = Get-ItemProperty 'HKLM:\HARDWARE\DESCRIPTION\System\BIOS'
Write-Host "BIOS日期:" $bios.BIOSReleaseDate
# 应显示 05/29/2019，表示更新成功
```

## 常见问题

**Q: Error 3 Internal Error**
A: Windows下FPT被Flash描述符安全限制拦截，必须用DOS方式。

**Q: U盘选了但立即弹回启动菜单**
A: BIOS引导模式是UEFI，FreeDOS是MBR格式，需先在BIOS将引导模式改为LEGACY。

**Q: 刷完后BIOS变英文**
A: 正常现象，进BIOS找Language选项改回中文，再Load Optimized Defaults。

**Q: 版本号还是5.13没变**
A: 正常，版本号不变但日期从2018/03/21变为2019/05/29即为成功，该版本正式支持Intel 9代CPU。
