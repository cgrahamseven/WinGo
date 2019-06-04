# WinGo
An extendable cross-platform Windows remote administration tool written in Go.
# About
WinGo is a tool similar to PsExec by Mark Russinovich, but rather than being tied to running the client on Windows, you can run it on Windows, MacOS, or Linux. WinGo was designed to be extendable by the end-user and allows you to write extensions in C# that are loaded automatically at runtime on the remote administration target. This is possible because the WinGo server-side shell runs within powershell on your target. This also eliminates the need to ship a large binary over the network like PsExec and it allows the server-side shell to run both itself and its extensions entirely in memory. 
# Supported Windows Versions
The WinGo server component has been tested on Windows 8+ as well as Server 2012 R2+ and relies heavily on Powershell and the .NET framework. Windows 7 and earlier Windows Server versions are not supported, although you may still have luck with them depending on the .NET framework version installed. Desktops and servers that have had Powershell severely restricted or disabled will obviously prevent WinGo from functioning. 
# Downloads
Requires macOS 10.10 or later, Intel 64-bit processor

[v1.2 download for MacOS](https://github.com/cgrahamseven/WinGoMacOS/raw/master/WinGoMacOS.tar.gz) SHA256: bd93d873afedcf76cca17046aa41ddb7148968508c9e08c32095c54cb6467727

Requires Windows 7 or later, Intel 64-bit processor

[v1.2 download for Windows](https://github.com/cgrahamseven/WinGoWindows/raw/master/WinGoWindows.zip) SHA256: 1bf3784ab258a1b9681862131e72aca89d05632459e4113ef7878511757b37af

Requires Linux 2.6.23 or later, Intel 64-bit processor

[v1.2 download for Linux](https://github.com/cgrahamseven/WinGoLinux/raw/master/WinGoLinux.tar.gz) SHA256: 49baadb5ca28a3a0bd4cb685de3a744b3ed15da7415c629b37a373a0616a5489


# Extensions License
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

# Usage
WinGo supports authentication via password or ntlm hash. If you wish to use a password, simply omit the ntlm hash option.
```
Usage: fields in <brackets> are required. Fields in (parenthesis) are optional.
WinGo <address> <username> <domain> (ntlm hash)
./WinGo 192.168.7.142 Administrator TESTDOMAIN
Password: 
[*] Checking for new extensions...
[*] Initiating remote WinGo session...
[*] Loading extensions...
[+] Successfully loaded extension: cmd
[+] Successfully loaded extension: process
[+] Successfully loaded extension: registry
[+] Successfully loaded extension: services
[+] Successfully loaded extension: shell

___       ______       _________      
__ |     / /__(_)________  ____/_____ 
__ | /| / /__  /__  __ \  / __ _  __ \
__ |/ |/ / _  / _  / / / /_/ / / /_/ /
____/|__/  /_/  /_/ /_/\____/  \____/ 
                                                                          

Windows Remote Administration Shell v1.2

WinGo C:\WINDOWS\system32>
```
# Folders
There are two folders currently associated with WinGo. They are 'downloads' and 'extensions'. The downloads folder will store any files transferred from the remote host via the 'get' command, and extensions holds any standard api extensions as well as your own custom extensions. 
# Extensions
Extensions are simply C# source code files. When writing your own extension, the source file name must be unique, should be lowercase, and must have no file extension. In addition, a second file should be created with the same name as your extension but saved with a .refs file extension. The .refs file must contain a list of .NET assembly references required for your source code to function, with each dll being comma-separated. If you forget to include an assembly reference for a particular class that is used in your source, the shell will throw an exception and your extension will not be loaded. The majority of extensions will likely only require System.dll and System.Core.dll. For reference, you can look at the existing extensions that ship with WinGo as part of the standard api. 
# Invoking Custom Extension Methods
Custom extensions are at the core of WinGo and are designed to allow users the flexibility and power to extend the base client. The first step in using custom extensions is to write the source code, add the required references, and save the file in the extensions folder.

Every custom extension needs to consist of a namespace, class, and empty main method, in addition to the methods you implement yourself. To invoke your custom extension, assuming it was loaded without error, use the following syntax:
```
[Namespace.Class]::Method(arg1, arg2, arg3...)
```
If you need to pass arguments to your methods, the following argument types are currently supported:
```
string - "This is a string"
bool - must be passed as True or False (note capitalization of first letter)
number - 1234567 (floats are not currently supported and any number you pass will be converted to the long .NET type which is 64 bits)

Example:

[MyNamespace.MyClass]::MyMethod("some string", True, 1024)
```
All current WinGo commands that are part of the standard api are essentially just aliased for faster use. If you wanted to use the cmd alias to get netstat output, you could invoke it like so:
```
[WinGo.Terminal]::Cmd("netstat -ano")
```
Unfortunately, there is currently no supported way to alias custom extension methods, however that may be something that is added in the future. If you wanted to help build out the standard api, you could implement more methods and request that they be added as aliased commands. 
# Standard API Aliases
A number of standard api routines are exposed as part of the core functionality of WinGo and their source is contained in the extensions folder under cmd, process, services, and shell. Their usage is as follows:
## Shell
**cd** - Change the current shell directory. This works similarly to cmd.exe in that you can change directories using accessible environment variables, using fully qualified paths, or using paths available in your current directory.
```
Examples:
cd %TMP% - changes directory to your temp folder
WinGo C:\> cd %TMP%


WinGo C:\WINDOWS\TEMP>

cd Users - change directory to Users folder from your current directory
WinGo C:\> cd Users


WinGo C:\Users>

cd "Program Files" - change directory to Program Files folder from your current directory. Note the required use of double quotes. In general, any command that is followed by another argument that contains a space should be denoted by the use of double quotes, otherwise the command will likely fail. 
WinGo C:\> cd "Program Files"


WinGo C:\Program Files>
```
**dir** - get a complete directory listing for either the current directory (if no path is supplied) or for a specified path. Environment variables are allowed here as well.
```
Examples:
WinGo C:\> dir
10/30/2015  12:24 AM    <DIR>           $Recycle.Bin
4/3/2019    11:29 AM    <DIR>           Documents and Settings
2/13/2016   5:22 AM     <DIR>           Logs      
9/15/2018   12:33 AM    <DIR>           PerfLogs  
9/15/2018   12:33 AM    <DIR>           Program Files
9/15/2018   12:33 AM    <DIR>           Program Files (x86)
9/15/2018   12:33 AM    <DIR>           ProgramData
4/3/2019    11:29 AM    <DIR>           Recovery  
4/3/2019    11:27 AM    <DIR>           System Volume Information
9/14/2018   11:09 PM    <DIR>           Users     
9/14/2018   11:09 PM    <DIR>           Windows   
5/6/2019    10:02 AM    <DIR>           Windows.old
10/30/2015  1:13 AM     400,228         bootmgr   
10/30/2015  1:13 AM     01              BOOTNXT   
4/3/2019    11:27 AM    1,231,228,928   pagefile.sys
4/3/2019    11:27 AM    16,777,216      swapfile.sys


WinGo C:\>
```
**del** - deletes a file either from the current directory (if no complete path is provided) or a specified file path
```
Examples:
WinGo C:\> dir
10/30/2015  12:24 AM    <DIR>           $Recycle.Bin
4/3/2019    11:29 AM    <DIR>           Documents and Settings
2/13/2016   5:22 AM     <DIR>           Logs      
9/15/2018   12:33 AM    <DIR>           PerfLogs  
9/15/2018   12:33 AM    <DIR>           Program Files
9/15/2018   12:33 AM    <DIR>           Program Files (x86)
9/15/2018   12:33 AM    <DIR>           ProgramData
4/3/2019    11:29 AM    <DIR>           Recovery  
4/3/2019    11:27 AM    <DIR>           System Volume Information
9/14/2018   11:09 PM    <DIR>           Users     
9/14/2018   11:09 PM    <DIR>           Windows   
5/6/2019    10:02 AM    <DIR>           Windows.old
10/30/2015  1:13 AM     400,228         bootmgr   
10/30/2015  1:13 AM     01              BOOTNXT   
5/15/2019   10:12 AM    4,145,664       filetodelete
4/3/2019    11:27 AM    1,231,228,928   pagefile.sys
4/3/2019    11:27 AM    16,777,216      swapfile.sys


WinGo C:\> del filetodelete
[+] Success

WinGo C:\>
```
**copy** - copies a file from the current directory (if no complete path is provided) to the provided destination or from the provided full path to the provided destination
```
Examples:
WinGo C:\> copy c:\windows\system32\cmd.exe test.exe
[+] Success

WinGo C:\> dir
10/30/2015  12:24 AM    <DIR>           $Recycle.Bin
4/3/2019    11:29 AM    <DIR>           Documents and Settings
2/13/2016   5:22 AM     <DIR>           Logs      
9/15/2018   12:33 AM    <DIR>           PerfLogs  
9/15/2018   12:33 AM    <DIR>           Program Files
9/15/2018   12:33 AM    <DIR>           Program Files (x86)
9/15/2018   12:33 AM    <DIR>           ProgramData
4/3/2019    11:29 AM    <DIR>           Recovery  
4/3/2019    11:27 AM    <DIR>           System Volume Information
9/14/2018   11:09 PM    <DIR>           Users     
9/14/2018   11:09 PM    <DIR>           Windows   
5/6/2019    10:02 AM    <DIR>           Windows.old
10/30/2015  1:13 AM     400,228         bootmgr   
10/30/2015  1:13 AM     01              BOOTNXT   
4/3/2019    11:27 AM    1,231,228,928   pagefile.sys
4/3/2019    11:27 AM    16,777,216      swapfile.sys
5/15/2019   10:13 AM    278,528         test.exe  


WinGo C:\>
```
**env** - Get a list of all accessible environment variables
```
Examples:
WinGo C:\> env
COMPUTERNAME=DESKTOP-7T587IE
PUBLIC=C:\Users\Public
LOCALAPPDATA=C:\WINDOWS\system32\config\systemprofile\AppData\Local
PSModulePath=WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
PROCESSOR_ARCHITECTURE=AMD64
Path=C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;C:\WINDOWS\system32\config\systemprofile\AppData\Local\Microsoft\WindowsApps
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
ProgramFiles(x86)=C:\Program Files (x86)
PROCESSOR_LEVEL=6
ProgramFiles=C:\Program Files
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
USERPROFILE=C:\WINDOWS\system32\config\systemprofile
SystemRoot=C:\WINDOWS
ALLUSERSPROFILE=C:\ProgramData
DriverData=C:\Windows\System32\Drivers\DriverData
ProgramData=C:\ProgramData
PROCESSOR_REVISION=9e0a
USERNAME=DESKTOP-7T587IE$
CommonProgramW6432=C:\Program Files\Common Files
CommonProgramFiles=C:\Program Files\Common Files
OS=Windows_NT
PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 158 Stepping 10, GenuineIntel
ComSpec=C:\WINDOWS\system32\cmd.exe
PROMPT=$P$G
SystemDrive=C:
TEMP=C:\WINDOWS\TEMP
NUMBER_OF_PROCESSORS=2
APPDATA=C:\WINDOWS\system32\config\systemprofile\AppData\Roaming
TMP=C:\WINDOWS\TEMP
ProgramW6432=C:\Program Files
windir=C:\WINDOWS
USERDOMAIN=WORKGROUP


WinGo C:\>
```
**get** - gets a file from the remote server and saves it locally in your downloads folder. Requires either the full path to the remote file, or the filename of the file you want to retrieve from your shell's current directory. Additionally, the name to save the file as locally is also required. 
```
Examples:
WinGo C:\Windows\system32> get c:\windows\system32\cmd.exe cmd.exe
[+] Successfully downloaded file to downloads/cmd.exe


WinGo C:\Windows\system32> get cmd.exe cmd.exe
[+] Successfully downloaded file to downloads/cmd.exe


WinGo C:\Windows\system32>
```
**put** - transfers a local file to the remote server. The path to the local file is required in addition to the full path to save the file as on the remote server. If a full path is not given, WinGo saves the file in the current directory of the shell.
```
Examples:
WinGo C:\> put test.exe test.exe
[+] Successfully wrote file to C:\test.exe


WinGo C:\>
```
**exit** - terminates the remote administration session.
## Services
**services** - Gets a complete list of all installed services on the remote server (includes device drivers). Provides service name, display name, current state, and service type.
```
Examples:
WinGo C:\WINDOWS\system32> services
State       Type                        Name            Display Name

STOPPED     WIN32_SHARE_PROCESS         AJRouter        AllJoyn Router Service
STOPPED     WIN32_OWN_PROCESS           ALG             Application Layer Gateway Service
STOPPED     WIN32_SHARE_PROCESS         AppIDSvc        Application Identity
RUNNING     WIN32_SHARE_PROCESS         Appinfo         Application Information
STOPPED     WIN32_SHARE_PROCESS         AppMgmt         Application Management
STOPPED     WIN32_SHARE_PROCESS         AppReadiness    App Readiness
STOPPED     WIN32_OWN_PROCESS           AppVClient      Microsoft App-V Client
STOPPED     WIN32_SHARE_PROCESS         AppXSvc         AppX Deployment Service (AppXSVC)
...truncated...
```
**service-query** - Provides the same information as the services command, but for the specified service. Requires service name.
```
Examples:
WinGo C:\WINDOWS\system32> service-query eventlog
RUNNING     WIN32_SHARE_PROCESS         eventlog        Windows Event Log

WinGo C:\WINDOWS\system32>
```
**service-start** - Starts the requested service. Requires service name.
```
Examples:
WinGo C:\Windows\system32> service-start gupdate
[+] Success

WinGo C:\Windows\system32>
```
**service-stop** - Stops the requested service. Requires service name.
```
Examples:
WinGo C:\Windows\system32> service-stop eventlog
[+] Success

WinGo C:\Windows\system32>
```
## Process
**ps** - Gets a list of all running processes and provides details such as pid, process name, process image, and physical memory usage.
```
Examples:
WinGo C:\WINDOWS\system32> ps
Pid         Memory Usage (bytes)        Process Name            Process Image Path

6672        368,640                     RuntimeBroker           C:\Windows\System32\RuntimeBroker.exe
3544        00                          svchost                 C:\WINDOWS\system32\svchost.exe
1376        00                          svchost                 C:\WINDOWS\system32\svchost.exe
4132        00                          cmd                     C:\WINDOWS\system32\cmd.exe
7676        208,896                     dllhost                 C:\WINDOWS\system32\DllHost.exe
2356        00                          SearchUI                C:\Windows\SystemApps\Microsoft.Windows.Cortana_cw5n1h2txyewy\SearchUI.exe
8068        00                          conhost                 C:\WINDOWS\system32\conhost.exe
5112        6,283,264                   explorer                C:\WINDOWS\Explorer.EXE
6964        00                          RuntimeBroker           C:\Windows\System32\RuntimeBroker.exe
1564        00                          ApplicationFrameHost    C:\WINDOWS\system32\ApplicationFrameHost.exe
1760        00                          svchost                 C:\WINDOWS\System32\svchost.exe
4320        1,585,152                   conhost                 C:\WINDOWS\system32\conhost.exe
3528        00                          ctfmon                  C:\WINDOWS\system32\ctfmon.exe
5892        540,672                     RuntimeBroker           C:\Windows\System32\RuntimeBroker.exe
1360        1,495,040                   SearchIndexer           C:\WINDOWS\system32\SearchIndexer.exe
768         3,743,744                   svchost                 C:\WINDOWS\system32\svchost.exe
9332        1,265,664                   LogonUI                 C:\WINDOWS\System32\LogonUI.exe
6280        192,512                     RuntimeBroker           C:\Windows\System32\RuntimeBroker.exe
7264        724,992                     RuntimeBroker           C:\Windows\System32\RuntimeBroker.exe
2732        7,467,008                   WmiPrvSE                C:\WINDOWS\system32\wbem\wmiprvse.exe
...truncated...
```
**ps-start** - Creates a new process. Takes two arguments; a fully qualified image path and an optional argument to provide to the process at startup.
```
Examples:
WinGo C:\Windows\system32> ps-start c:\windows\notepad.exe c:\windows\win.ini
[+] Success

WinGo C:\Windows\system32>
```
**ps-kill** - Kills a process. Requires process id. 
```
Examples:
WinGo C:\Windows\system32> ps-kill 2008
[+] Success

WinGo C:\Windows\system32>
```
**ps-suspend** - Suspends a process. Requires process id.
```
Examples:
WinGo C:\Windows\system32> ps-suspend 2332
[+] Success

WinGo C:\Windows\system32>
```
**ps-resume** - Resumes a suspended process. Requires process id.
```
Examples:
WinGo C:\Windows\system32> ps-resume 2332
[+] Success

WinGo C:\Windows\system32>
```
## Cmd
**cmd** - Launches cmd.exe with the provided command passed as an argument to the process and returns the result of the command. Strings requiring additional escaping can use the grave accent character.
```
Examples:
WinGo C:\Windows\system32> cmd whoami
nt authority\system


WinGo C:\Windows\system32>

cmd <argument that has spaces which must be denoted with double quotes>
WinGo C:\Windows\system32> cmd "netstat -ano"

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       904
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       1244
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       636
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       1020
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       792
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       736
  TCP    0.0.0.0:49157          0.0.0.0:0              LISTENING       748
  ...truncated...
  
cmd <argument that has spaces which must be denoted with double quotes and which requires additional quotes (use grave accent)>
WinGo C:\Program Files> cmd "type `c:\program files\desktop.ini`"

[.ShellClassInfo]
LocalizedResourceName=@%SystemRoot%\system32\shell32.dll,-21781


WinGo C:\Program Files>
```
## Registry
```
Predefined hive values for accessing subkeys:

HKLM - represents HKEY_LOCAL_MACHINE
HKCU - represents HKEY_CURRENT_USER
HKUSERS - represents HKEY_USERS
HKCR - represents HKEY_CLASSES_ROOT
```
**reg-listsubkeys** - Lists all subkeys under a registry hive or subkey. Requires at a minimum a hive value. 
```
Examples:
WinGo C:\WINDOWS\system32> reg-listsubkeys HKLM\Software\Microsoft\Windows\CurrentVersion
AccountPicture
ActionCenter
AdvertisingInfo
App Management
App Paths
AppHost
Applets
ApplicationFrame
AppModel
AppModelUnlock
AppReadiness
Appx
Audio
...truncated...
```
**reg-listkeyvalues** - Lists a key's value names and associated values. Where values are of type binary, they will be presented as a hex dump. Requires at a minimum a hive value.
```
Examples:
WinGo C:\WINDOWS\system32> reg-listkeyvalues HKLM\Software\Microsoft\Windows\CurrentVersion
Name                            Value               

ProgramFilesDir                 C:\Program Files    
CommonFilesDir                  C:\Program Files\Common Files
ProgramFilesDir (x86)           C:\Program Files (x86)
CommonFilesDir (x86)            C:\Program Files (x86)\Common Files
CommonW6432Dir                  C:\Program Files\Common Files
DevicePath                      C:\WINDOWS\inf      
MediaPathUnexpanded             C:\WINDOWS\Media    
ProgramFilesPath                C:\Program Files    
ProgramW6432Dir                 C:\Program Files    
SM_ConfigureProgramsName        Set Program Access and Defaults
SM_GamesName                    Games
```
**reg-createsubkey** - Create a new subkey under an existing hive. Requires at a minimum an existing hive and new subkey name.
```
Examples:
(Create a single new sub key called Test)
WinGo C:\WINDOWS\system32> reg-createsubkey HKLM\Software\Microsoft\Windows\CurrentVersion\Test
[+] Success

(Create a new sub key under previously created subkey Test called more)
WinGo C:\WINDOWS\system32> reg-createsubkey HKLM\Software\Microsoft\Windows\CurrentVersion\Test\more
[+] Success

(Create multiple children under Test\More. Creates a subkey called 'even' and another called 'more' under 'even'.)
WinGo C:\WINDOWS\system32> reg-createsubkey HKLM\Software\Microsoft\Windows\CurrentVersion\Test\more\even\more
[+] Success
```
**reg-deletesubkey** - Deletes a single subkey under an existing hive. Requires at a minimum an existing hive and subkey name.
```
Examples:
WinGo C:\WINDOWS\system32> reg-deletesubkey HKLM\Software\Microsoft\Windows\CurrentVersion\Test
[+] Success
```
**reg-deletetree** - Deletes an entire registry tree. This command is used when you have a subkey that has multiple children and you want to remove the entire tree. You cannot use reg-deletesubkey to delete a registry key that has one or more children. In that case, use this command. 
```
Examples:
(Consider previous example under reg-createsubkey where Test has children: more\even\more)
WinGo C:\WINDOWS\system32> reg-deletetree HKLM\Software\Microsoft\Windows\CurrentVersion\Test
[+] Success
```
**reg-getvalue** - Retrieves the value for the specified subkey and value name. Requires both a hive and/or subkey as well as the value name to retrieve the value for. Values that contain binary data will be presented as a hex dump.
```
Examples:
WinGo C:\WINDOWS\system32> reg-getvalue HKLM\Software\Microsoft\Windows\CurrentVersion ProgramFilesDir
C:\Program Files
```
**reg-setvalue** - Sets a subkey value. Requires at least 3 arguments. When a subkey, EXISTING value name, and value are given, this command will look up the existing value data type (REG_SZ, DWORD, etc) and set the value with what you have provided as the existing data type. If you provide a subkey, EXISTING value name, value, and value type, this command will CHANGE the existing value type to the value type you specified and set the value to what you provided. If you provide a  subkey, a NEW value name, and a value, you MUST provide the value type as well. In that case, a new value will be created. 
```
The following are the predefined value types of which one must be used when creating a new value name:
dword - a 32 bit integer value (ie REG_DWORD)
qword - a 64 bit integer value (ie REG_QWORD)
estring - an expandable null-terminated string (may contain an environment var such as %SYSTEMROOT% ie REG_EXPAND_SZ) 
mstring - a multi string that may contain several null-terminated strings (ie REG_MULTI_SZ)
string - a simple null-terminated string (ie REG_SZ)
binary - contains binary data (IMPORTANT: You must base64 encode your binary data and provide that string as the argument to reg-setvalue for the value data in order to set data of this type)
```
```
Examples:
(An existing value name of SM_GamesName has been provided. Since only a value name and value data was provided, this command will result in WinGo looking up the data type for the existing value 'SM_GamesName' and then setting the value to 'MyGames'.)
WinGo C:\WINDOWS\system32> reg-setvalue HKLM\Software\Microsoft\Windows\CurrentVersion SM_GamesName MyGames
[+] Success

(A new value name of 'NewValue' has been provided. In this case, a value type is required and 'dword' has been chosen. This command will create a new value name under the subkey 'Test' called 'NewValue' with a value type of 'dword' set to '1'.)
WinGo C:\WINDOWS\system32> reg-setvalue HKLM\Software\Microsoft\Windows\CurrentVersion\Test NewValue 1 dword
[+] Success

(An existing value name has been provided as well as a value type. This command will now CHANGE the value type of 'NewValue' from a 'dword' to a 'string' (REG_SZ) and set its value to 'blahblah'.)
WinGo C:\WINDOWS\system32> reg-setvalue HKLM\Software\Microsoft\Windows\CurrentVersion\Test NewValue blahblah string
[+] Success

(Changes the existing value name 'NewValue's type to REG_BINARY and uses the base64-encoded binary data 01 02 03 04 as the value to set. The registry extension will convert the provided base64-encoded binary data to binary and set it.)
WinGo C:\WINDOWS\system32> reg-setvalue HKLM\Software\Microsoft\Windows\CurrentVersion\Test NewValue AQIDBA== binary
[+] Success
```
