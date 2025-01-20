---
layout: post
title: 用 Windows Application 访问 UEFI variable
date: 2019-03-21 21:00:00.000000000 +08:00
description: 本文介绍如何通过Windows API实现在Windows OS下对UEFI variable的访问。
author: aaron-li
categories: [代码相关, BIOS]
tags: [uefi, uefi variable, windows api, windows]
---

## 0. 前言

参与过UEFI编程的同学可能会对“Variable”有所了解。现行的[UEFI Spec 2.7](http://www.uefi.org/sites/default/files/resources/UEFI_Spec_2_7.pdf) Chap 8.2详细描述了Variable Services。

UEFI中很多module中都应用了Variable Service，作为一种Runtime Service，Variable可以向OS传递信息。

那么在与OS联调的时候，如何在OS中读取Variable呢？本文将描述如何在Windows下读取和写入UEFI中的Variable（参考MFST的online doc 
[Access UEFI firmware variables from a Universal Windows App](https://docs.microsoft.com/en-us/windows/desktop/SysInfo/access-uefi-firmware-variables-from-a-universal-windows-app)）。

## 1. 准备工作

Windows下做UEFI的同学都对Visual Studio不陌生，如果没有就准备一个（需要支持Windows API），本文作者使用的是Visual Studio 2015。
我们还需要一台非安装在Legacy Bios上的Windows 8/10系统，这里有条件的同学可以直接装一台系统，没有的可以在Virtual Box上安装一个Windows（注意打开EFI，[Virtual Box 6.0](
https://www.virtualbox.org/wiki/Downloads)可能会存在EFI系统不能正常安装Windows，可以使用[5.2](https://www.virtualbox.org/wiki/Download_Old_Builds_5_2)的版本）

## 2. 程序提权 (Grant Privilege)
>To read a firmware environment variable, the user account that the app is running under must have the SE_SYSTEM_ENVIRONMENT_NAME privilege. A Universal Windows app must be run from an administrator account and follow the requirements outlined in Access UEFI firmware variables from a Universal Windows App. [[1]](https://docs.microsoft.com/en-us/windows/desktop/api/Winbase/nf-winbase-getfirmwareenvironmentvariablea)

根据MFST online doc（上面引用），首先我们需要对Universal APP进行提权，赋予APP `SE_SYSTEM_ENVIRONMENT_NAME` 权限，注意这个APP需要运行在Admin下（使用管理员权限运行）。
```c
  HANDLE   hToken;
  PHANDLE  phToken = &hToken;
  //
  // Grant Privilege
  //
  if (!OpenProcessToken(GetCurrentProcess(),
    TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, phToken))
  {
    printf("OpenProcessToken failed!\n");
    return 0;
  }
  if (!SetPrivilege(hToken, SE_SYSTEM_ENVIRONMENT_NAME, TRUE)) 
  {
    printf("Please Try run as Administrator.\n");
    return 0;
  }
```
SetPrivilege函数为MFST例程，参见[Enabling and Disabling Privileges in C++](https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--)。

OpenProcessToken 为 Windows API，参见[OpenProcessToken function](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-openprocesstoken)。

GetCurrentProcess 为 Windows API，参见[GetCurrentProcess function](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-getcurrentprocess)。

不使用管理员权限运行会报`ERROR_NOT_ALL_ASSIGNED`错误：
>The token does not have the specified privilege.
>Please Try run as Administrator.


## 3. Get/Set Variable
进入正题，在提权之后，APP可以通过GetFirmwareEnvironmentVariable与SetFirmwareEnvironmentVariable对Variable进行访问了。

对其进行简单的封装：
```c
DWORD
GetVariable(
  _In_ LPCSTR lpName,
  _In_ LPCSTR lpGuid,
  _Out_writes_bytes_to_opt_(nSize, return) PVOID pBuffer,
  _In_ DWORD    nSize
) {
  DWORD Status;
  DWORD VariableSize = 0;
  //
  // Read Variable
  //
  VariableSize = GetFirmwareEnvironmentVariable(
    lpName,
    lpGuid,
    pBuffer,
    nSize
  );

  Status = GetLastError();
  if (Status == ERROR_SUCCESS) {
    printf("Get %s success.\n", lpName);
  }
  else if (Status == ERROR_INVALID_FUNCTION) {
    printf("System in Lagency Mode.\n");
  }
  else {
    printf("Error: 0x%x\n", Status);
  }
  return VariableSize;
}
```
```c
BOOL
SetVariable(
  _In_ LPCSTR lpName,
  _In_ LPCSTR lpGuid,
  _In_reads_bytes_opt_(nSize) PVOID pValue,
  _In_ DWORD    nSize
) {
  DWORD Status;
  //
  // Write Variable
  //
  SetFirmwareEnvironmentVariable(
    lpName,
    lpGuid,
    pValue,
    nSize
  );

  Status = GetLastError();
  if (Status == ERROR_SUCCESS) {
    printf("Set %s success.\n", lpName);
    return TRUE;
  }
  else if (Status == ERROR_INVALID_FUNCTION) {
    printf("System in Lagency Mode.\n");
  }
  else {
    printf("Error: 0x%x\n", Status);
  }
  return FALSE;
}
```

如果运行在Legacy Bios下，会报`ERROR_INVALID_FUNCTION`错误：
>System in Lagency Mode.
>Incorrect function.

## 4. Format Error
`GetLastError()`还有很多return值，我们并不能一一将其列举，详见：[System Error Codes](https://docs.microsoft.com/en-us/windows/desktop/Debug/system-error-codes)。
但可以将其转化为格式化打印输出，这里直接在上面的封装中进行修改（只修改`GetVariable()`)。
MFST提供了FormatMessage API来处理所有的系统Error，详见：[FormatMessage function](https://docs.microsoft.com/en-us/windows/desktop/api/WinBase/nf-winbase-formatmessage)。

```c
DWORD
GetVariable(
  _In_ LPCSTR lpName,
  _In_ LPCSTR lpGuid,
  _Out_writes_bytes_to_opt_(nSize, return) PVOID pBuffer,
  _In_ DWORD    nSize
) {
  DWORD Status;
  DWORD VariableSize = 0;
  //
  // Read Variable
  //
  VariableSize = GetFirmwareEnvironmentVariable(
    lpName,
    lpGuid,
    pBuffer,
    nSize
  );

  Status = GetLastError();
  if (Status == ERROR_SUCCESS) {
    printf("Get %s success.\n", lpName);
  }
  else if (Status == ERROR_INVALID_FUNCTION) {
    printf("System in Lagency Mode.\n");
  }
  else {
      UINT8 * pBuff;
      FormatMessage(
        FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM,
        NULL,
        Status,
        0,
        (LPTSTR)&pBuff,
        0, 
        NULL);
      printf("%s\n", pBuff);
  }
  return VariableSize;
}
```
可能会出现的情况：
`ERROR_ENVVAR_NOT_FOUND` : 没有要访问的Variable
>The system could not find the environment option that was entered.

`ERROR_PRIVILEGE_NOT_HELD`: 没有取得相应的权限（参见Chap2）
>A required privilege is not held by the client.

## Reference
[Access UEFI firmware variables from a Universal Windows App](https://docs.microsoft.com/en-us/windows/desktop/SysInfo/access-uefi-firmware-variables-from-a-universal-windows-app)  
[UEFI Spec 2.7](http://www.uefi.org/sites/default/files/resources/UEFI_Spec_2_7.pdf)  
[Enabling and Disabling Privileges in C++](https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--)  
[OpenProcessToken function](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-openprocesstoken)  
[GetCurrentProcess function](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-getcurrentprocess)  
[System Error Codes](https://docs.microsoft.com/en-us/windows/desktop/Debug/system-error-codes)  
[FormatMessage function](https://docs.microsoft.com/en-us/windows/desktop/api/WinBase/nf-winbase-formatmessage)  
