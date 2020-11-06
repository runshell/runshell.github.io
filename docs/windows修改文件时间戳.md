# windows修改文件时间戳

* powershell  

使用Get-ChildItem获得FileInfo对象，直接给其对应属性赋值即可，时间格式为`2018-08-25 15:23:36`,如果选用当前时间可直接使用命令`date`代替时间。  

```powershell
PS C:\Users\HUAWEI\Desktop> Get-ItemProperty -Path .\1.jsp | fl




Name           : 1.jsp
Length         : 2598
CreationTime   : 2020/8/24 14:29:10
LastWriteTime  : 2020/8/24 14:29:10
LastAccessTime : 2020/11/6 14:54:12
Mode           : -a----
LinkType       :
Target         : {}
VersionInfo    : File:             C:\Users\HUAWEI\Desktop\1.jsp
                 InternalName:
                 OriginalFilename:
                 FileVersion:
                 FileDescription:
                 Product:
                 ProductVersion:
                 Debug:            False
                 Patched:          False
                 PreRelease:       False
                 PrivateBuild:     False
                 SpecialBuild:     False
                 Language:




PS C:\Users\HUAWEI\Desktop> Set-ItemProperty -Path .\1.jsp -Name CreationTime -Value '2020年11月6日 14:56:25'
PS C:\Users\HUAWEI\Desktop> Get-ItemProperty -Path .\1.jsp | fl


    目录: C:\Users\HUAWEI\Desktop



Name           : 1.jsp
Length         : 2598
CreationTime   : 2020/11/6 14:56:25
LastWriteTime  : 2020/8/24 14:29:10
LastAccessTime : 2020/11/6 14:57:36
Mode           : -a----
LinkType       :
Target         : {}
VersionInfo    : File:             C:\Users\HUAWEI\Desktop\1.jsp
                 InternalName:
                 OriginalFilename:
                 FileVersion:
                 FileDescription:
                 Product:
                 ProductVersion:
                 Debug:            False
                 Patched:          False
                 PreRelease:       False
                 PrivateBuild:     False
                 SpecialBuild:     False
                 Language:




PS C:\Users\HUAWEI\Desktop>
```
一个问题值得思考：我们只是对powershell中的一个变量赋值，这顶多改变内存中的值，它怎么就同步到磁盘将文件时间戳永久改变了呢？一开始我在网上看到这中操作的时候，我是试都不想试，但最终事实摆在眼前，不得不服这种骚操作。

* c/c++  

在不了解powershell的时候，没有找到很好的工具来修改文件时间戳，于是有了下面的代码，用户交互上完全模仿linux中的touch。所以用法和linux中touch相同。  
该程序本质是调用`SetFileTime(f,ct,at,mt)`函数修改时间戳，一大把的代码都在做参数解析，真正实现修改时间戳的就这一个函数。  

[wtouch.cpp⬇](https://raw.githubusercontent.com/shashade250/shashade250.github.io/master/other/wtouch.cpp) &nbsp;&nbsp; [wtouch.exe](../other/wtouch.exe)
```c
#include <iostream>
#include <windows.h>
#include <stdlib.h>
#include <string.h>

using namespace std;

int main(int argc,char *argv[])
{
    string inputtime,rfile,file;
    string help="用法：wtouch [选项]... 文件...\n"
    "更新文件的访问时间、修改时间和创建时间为当前时间。\n"
    "如果文件不存在将创建一个空文件，除非提供了-c参数\n\n\n"
    "通过使用一些特定的参数对文件时间做相应的修改。\n\n"
    "  -a                       只更改访问时间\n"
    "  -c                       只更改创建时间\n"
    "  -m                       只更改修改时间\n"
    "  -n, --no-create          不创建任何文件\n"
    "  -r, --reference=FILE     使用这个文件的时间而不是当前时间\n"
    "  -t STAMP                 使用[YYYY]MMDDhhmm[.ss]而非当前时间\n"
    "  -h, --help               显示此帮助信息并退出\n\n"
    "请注意，-d 和-t 选项可接受不同的时间/日期格式。\n";
    FILETIME ctim,atim,mtim;
    FILETIME *ct;
    FILETIME *mt;
    FILETIME *at;
    SYSTEMTIME systime;
    HANDLE rf,f;
    int i;
    bool m=true,a=true,c=true,n=true;
    if(argc==1)
    {
        cout<<help;
        return 0;
    }
    for(i=1;i<argc;i++)
    {
        if(argv[i]==string("-m")) {a=false;c=false;}
        else if(argv[i]==string("-c")) {a=false;m=false;}
        else if(argv[i]==string("-a")) {m=false;c=false;}
        else if(argv[i]==string("-acm")||
                argv[i]==string("-cam")||
                argv[i]==string("-cma")||
                argv[i]==string("-amc")||
                argv[i]==string("-mac")||
                argv[i]==string("-mca"))
            {m=true;a=true;c=true;}
        else if(argv[i]==string("-cm")||argv[i]==string("-mc")) a=false;
        else if(argv[i]==string("-ac")||argv[i]==string("-ca")) m=false;
        else if(argv[i]==string("-am")||argv[i]==string("-ma")) c=false;
        else if(argv[i]==string("-n")||argv[i]==string("--no-create")) n=true;
        else if(argv[i]==string("--reference")||argv[i]==string("-r")) rfile=argv[++i];
        else if(argv[i]==string("-t")) inputtime=argv[++i];
        else if(argv[i]==string("--help")||argv[i]==string("-h")) cout<<help;
        else file=argv[i];
    }
    //获取当前时间
    GetLocalTime(&systime);
    //从-t获取时间覆盖当前时间
    if(inputtime.length()!=0)
    {
        i=inputtime.length()-1;
        if(inputtime[i-2]=='.')
        {
            systime.wSecond=atoi(inputtime.substr(i-1,2).c_str());
            i-=3;
        }
        else
            systime.wSecond=0;
        if(i>0)
        {
            systime.wMinute=atoi(inputtime.substr(i-1,2).c_str());
            i-=2;
        }
        else
            return 0;
        if(i>0)
        {
            systime.wHour=atoi(inputtime.substr(i-1,2).c_str());
            i-=2;
        }
        else
            return 0;
        if(i>0)
        {
            systime.wDay=atoi(inputtime.substr(i-1,2).c_str());
            i-=2;
        }
        if(i>0)
        {
            systime.wMonth=atoi(inputtime.substr(i-1,2).c_str());
            i-=4;
        }
        if(i>0)
        {
            systime.wYear=atoi(inputtime.substr(i-1,4).c_str());
        }

    }
    SystemTimeToFileTime(&systime,&ctim);
    LocalFileTimeToFileTime(&ctim,&atim);
    ctim=atim;mtim=atim;
    //从参照文件获得文件时间
    if(rfile.length()!=0)
    {
        rf=CreateFileA(rfile.c_str(),GENERIC_READ | GENERIC_WRITE,
                       FILE_SHARE_READ | FILE_SHARE_WRITE,
                       NULL,OPEN_EXISTING,
                       FILE_ATTRIBUTE_NORMAL,NULL);
        if(rf==INVALID_HANDLE_VALUE)
        {
            cout<<"打开参照文件失败！";
            return 0;
        }
        if(!GetFileTime(rf,&ctim,&atim,&mtim))
        {
            cout<<"获取参照文件时间信息失败！";
            CloseHandle(rf);
            return 0;
        }
        CloseHandle(rf);
    }
    //i暂时用作dwCreationDisposition
    if(n) i=OPEN_EXISTING;//不创建新文件
    else i=OPEN_ALWAYS;//文件不存在时新建空文件
    f=CreateFileA(file.c_str(),GENERIC_READ | GENERIC_WRITE,
                    FILE_SHARE_READ | FILE_SHARE_WRITE,
                    NULL,i,
                    FILE_ATTRIBUTE_NORMAL,NULL);
    if(f==INVALID_HANDLE_VALUE)
    {
        cout<<"打开文件失败！";
        return 0;
    }
    ct=&ctim;mt=&mtim;at=&atim;
    if(!m) mt=NULL;
    if(!c) ct=NULL;
    if(!a) at=NULL;
    if(!SetFileTime(f,ct,at,mt))
        cout<<"修改失败！";
    CloseHandle(f);
    return 0;
}
```
