keepalive.bat如下:

@echo off
title keepalive

 set MINIMIZED=1
if not defined MINIMIZED (
    set MINIMIZED=1
    powershell -Command "Start-Process '%~f0' -WindowStyle Minimized"
    exit
)

powershell -ExecutionPolicy Bypass -File  keep_alive.ps1
pause

-------keep_alive.ps1内容----------

<#
功能：阻止Windows自动休眠睡眠；Win+L手动锁屏后台程序网络继续运行
退出：关闭窗口 / Ctrl+C，自动恢复电源策略
#>

Add-Type @"
using System;
using System.Runtime.InteropServices;
public class Win32 {
    [DllImport("kernel32.dll", CharSet = CharSet.Auto)]
    public static extern uint SetThreadExecutionState(uint esFlags);

    public const uint ES_CONTINUOUS        = 0x80000000;
    public const uint ES_SYSTEM_REQUIRED   = 0x00000001;
    public const uint ES_DISPLAY_REQUIRED  = 0x00000002;
}
"@

$flag = [Win32]::ES_CONTINUOUS -bor [Win32]::ES_SYSTEM_REQUIRED -bor [Win32]::ES_DISPLAY_REQUIRED

Write-Host "`n===== 防自动休眠已激活 =====" -ForegroundColor Green
Write-Host "保活窗口保持打开，则电源不会关闭；当关闭窗口或者 Ctrl+C 将恢复系统原有电源策略`n" -ForegroundColor Yellow

try {
    while($true) {
        # | Out-Null 吃掉API返回值，不会打印数字
        [Win32]::SetThreadExecutionState($flag) | Out-Null
        Start-Sleep -Seconds 100
    }
}
finally {
    [Win32]::SetThreadExecutionState([Win32]::ES_CONTINUOUS) | Out-Null
    Write-Host "`n脚本已退出，恢复系统默认电源策略" -ForegroundColor Green
}
