Today I am going to complete a memory forensics challenge published in CyberDefenders.

![image](https://user-images.githubusercontent.com/93026147/150980002-3c54f728-179a-4c95-bd09-5d0c3d208492.png)


**Scenario:**
One of the SOC analysts took a memory dump from a machine infected with a meterpreter malware. As a Digital Forensicators, your job is to analyze the dump, extract the available indicators of compromise (IOCs) and answer the provided questions.

**Questions** 

**#1. What is the SHA1 hash of triage.mem (memory dump)?**

 This is pretty straightforward. We can simply run the shasum command on the triage image and obtain the answer.
![image](https://user-images.githubusercontent.com/93026147/150980192-2ceb8485-c457-4227-9146-c03f82ae0766.png)

<c95e8cc8c946f95a109ea8e47a6800de10a27abd>
  
  --------------------------------------------------------------------------------------
**#2. What volatility profile is the most appropriate for this machine? (ex: Win10x86_14393)**
  ![image](https://user-images.githubusercontent.com/93026147/150980305-faf16ee8-a3d2-47c7-aa5e-2585268ce2bd.png)

The Volatility profile can be found using kdbgscan on the image.

<Win7SP1x64>
  
  ------------------------------------------------------------------------------------
**#3. What was the process ID of notepad.exe?**
  
  ![image](https://user-images.githubusercontent.com/93026147/150980278-25e7dd48-bbda-403a-94a9-7128751d98a1.png)

To determine the process ID of notepad, we can list the running processes at the time of memory capture and figure out the process ID.

This will give us the running processes and scrolling down below, we can see the process ID of notepad.exe.
<3032>
  
  ----------------------------------------------------------------------------------
**#4. Name the child process of wscript.exe.**
  
  ![image](https://user-images.githubusercontent.com/93026147/150980339-fa227117-cdb9-4778-8a78-f785fdc78b5b.png)

Volatility comes with a lot of plugins to make our life easier. pstree is another handy plugin that lists the processes in the parent-child structure.

There is another process below wscript.exe that is running as its child process.
<UWkpjFjDzM.exe>
  
  ----------------------------------------------------------------------------------------
**#5. What was the IP address of the machine at the time the RAM dump was created?**
  ![image](https://user-images.githubusercontent.com/93026147/150980362-723d3438-de4d-4780-9eea-4b8bd9cdbfd1.png)

The trick I used here is to look for network connections that have been established with this machine.

We can clearly see the IP address of the system.
<10.0.0.101>
  
  -----------------------------------------------------------------------------------------
**#6. Based on the answer regarding the infected PID, can you determine the IP of the attacker?**
  
Among all the established network connections, UWkpjFjDzM.exe looks to be the suspicious one. Further, we can see the IP address of the attacker on port 4444.
<10.0.0.106>
  
  --------------------------------------------------------------------------------------
**#7. How many processes are associated with VCRUNTIME140.dll?**
  ![image](https://user-images.githubusercontent.com/93026147/150980412-e26294af-9c09-4b76-8e73-cc66b92dc8b6.png)

We can use the dlllist plugin to list all the DLLs of the processes and grep that with VCRUNTIME140.dll to get the number of processes that are associated with that DLL. Here, the number is 5.

<5>
  
  -------------------------------------------------------------------------------------------------------
**#8. After dumping the infected process, what is its md5 hash?**
  ![image](https://user-images.githubusercontent.com/93026147/150980442-09aa86eb-59d6-4510-bd34-41eb51c67ec6.png)

We have already found out about the infected process. We can use procdump on the process ID of UWkpjFjDzM.exe to dump the process and find out the md5 hash.

Let us now find the md5 hash of this dump.
  
  ![image](https://user-images.githubusercontent.com/93026147/150980499-733e94cc-157a-41fc-935a-f458e5eca470.png)

  <690ea20bc3bdfb328e23005d9a80c290>
  
-------------------------------------------------------------------------------------------------------------
**#9. What is the LM hash of Bob’s account?**
All of the user account-related information is stored in registry hives. There are useful plugins to traverse the hives and one of them is hashdump.
    
![image](https://user-images.githubusercontent.com/93026147/150980558-f916faea-e366-4c43-8b3f-fff309332c29.png)

Volatility you beauty. One plugin got the job done.
<aad3b435b51404eeaad3b435b51404ee>
  
  -----------------------------------------------------------------------------------------------------------
**#10. What memory protection constants does the VAD node at 0xfffffa800577ba10 have?**
  
To obtain the VAD information of any memory image, we can use the vadinfo plugin.
![image](https://user-images.githubusercontent.com/93026147/150980602-bc592b52-abc2-453b-8d6b-156394c95c56.png)

We can see the memory protection as PAGE_READONLY.
<PAGE_READONLY>
  
  ----------------------------------------------------------------------------------------------------------
**#11. What memory protection did the VAD starting at 0x00000000033c0000 and ending at 0x00000000033dffff have?**
Let me use the same above command and look for one of the memory addresses and compare the ending to find out the correct memory protection.
![image](https://user-images.githubusercontent.com/93026147/150980627-cc8a1fc0-c131-46d6-9ad6-11dafc724661.png)

The memory protection starting at 0x00000000033c0000 and ending at 0x00000000033dffff is PAGE_NOACCESS.
<PAGE_NOACCESS>
  
  --------------------------------------------------------------------------------------------------------------
**#12. There was a VBS script that ran on the machine. What is the name of the script? (submit without file extension)**
The approach that I took for this question is to look at all the commands that had been executed and a handy plugin for this is cmdline.
![image](https://user-images.githubusercontent.com/93026147/150980661-999e4da2-c2d3-4bb3-ab5a-be8da1bdc003.png)

This will also give us the command line parameter for all the processes that can come in handy.
![image](https://user-images.githubusercontent.com/93026147/150980686-8eb5223d-d4a9-4323-b707-a995120ea3b3.png)

Scrolling the output, we will see this vbs script being run and this must be the answer.
<vhjReUDEuumrX>
  
  -------------------------------------------------------------------------------------------------------------
**#13. An application was run at 2019–03–07 23:06:58 UTC. What is the name of the program? (Include extension)**
I got stuck in this question for quite a bit. I found the following link useful for solving it. https://www.andreafortuna.org/2017/10/16/amcache-and-shimcache-in-forensic-analysis/
Basically, Shimcache will keep track of any program’s modification time within the system and record the timestamps.
Volatility has another impressive plugin to find the process which was last modified.
![image](https://user-images.githubusercontent.com/93026147/150980719-29c1bdd1-5051-40b4-abe7-d285674f258c.png)

There we have it, the application is Skype.exe.
<Skype.exe>
  
  ----------------------------------------------------------------------------------------------------------------
**#14. What was written in notepad.exe at the time when the memory dump was captured?**
To figure out what was written in notepad.exe, let me firstly dump its memory and use strings to figure out if I can see any recognizable text.
![image](https://user-images.githubusercontent.com/93026147/150980781-e8c866a3-c263-409a-8b76-48267125a086.png)

There is the dump, now I can look for strings.

This command does the trick for me and the answer has to be flag<REDBULL_IS_LIFE>
  
  ---------------------------------------------------------------------------------------------------------------
**#15. What is the short name of the file at file record 59045?**
I have got no idea what the short name of the file is. However, I do know that most of the information about the files in the NTFS filesystem is stored in the Master File Table.
There is a plugin called **mftparser** that might help us here.
  
![image](https://user-images.githubusercontent.com/93026147/150980831-588014c2-e1f8-4144-87de-0dc348881688.png)

Let me search the file with that record number.
  
![image](https://user-images.githubusercontent.com/93026147/150980849-b086073a-9619-4202-8aaa-108fc50c4976.png)

There it is.
<EMPLOY~1.XLS>
  
  ------------------------------------------------------------------------------------------------------------------
**#16. This box was exploited and is running meterpreter. What was the infected PID?**
  
We know that the machine has a malicious process that is running with the name UWkpjFjDzM.exe
Going through the process listing, I got the PID.
<3496>
  
  ------------------------------------------------------------------------------------------------------------
****Feedback****
  
Overall, this is a really fun challenge to try out especially if you are getting started with memory forensics and volatility. Let me know if you found this walkthrough useful.
