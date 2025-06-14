<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</head>
<body style="font-family:Arial, sans-serif; line-height:1.6; max-width:900px; margin:0 auto; padding:2rem;">

  <h1>🔐 TryHackMe Tempest Write-up</h1>
  <p><strong>Author:</strong> xoxoh<br/>
     <strong>Date:</strong> June 2025<br/>
     <strong>Join the room:</strong> <a href="https://tryhackme.com/room/tempestincident" target="_blank">Tempest on TryHackMe</a></p>
  <hr/>



  <h2>📘 Introduction</h2>
  <p>
    The <strong>Tempest</strong> room on TryHackMe challenges you to investigate a simulated malware incident using system logs and packet captures. In this walkthrough, we'll analyze the attack chain step-by-step using Sysmon logs, PCAP data, and reverse-engineering encoded payloads.
  </p>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/53d24ee6-9cdf-4a14-8a38-34117e2bf9c4" alt="Timeline Explorer" width="200" />
  </p>

<hr/>


<br>


  <h2>🔍 Task 3: Preparation - Tools and Artifacts</h2>
  
  <p>Before conducting the investigation, one of the most important steps is to compare the artifacts by their hashes. It is a common practice to verify if the artifacts are expected as it is.<br><br>
    You can get the hashes of each artifact by running Powershell from the taskbar and executing the following command:</p>
  <pre><code>Get-FileHash * -Algorithm SHA256</code></pre>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/8b3530aa-0e3d-4ba2-8f3c-c70ad041e888" alt="Timeline Explorer" width="700" />
  </p>
  <ul>
    <li><code>EvtxECmd.exe</code> is located on <code>C:\Tools\EvtxECmd\</code>. You can either move into the directory or use full path to run it.</li>
    <li><code>EvtxECmd.exe</code> is located on <code>C:\Tools\EvtxECmd\</code>. You can either move into the directory or use full path to run it.</li>
    <li><code>EvtxECmd.exe</code> is located on <code>C:\Tools\EvtxECmd\</code>. You can either move into the directory or use full path to run it.</li>
  </ul>

<br>
<hr/>


<br>
  
  <h2>🔍 Task 4: Initial Access - Malicious Document</h2>

  <p>Use <code>EvtxECmd</code> to convert logs:</p>
  <ul>
    <li><code>EvtxECmd.exe</code> is located on <code>C:\Tools\EvtxECmd\</code>. You can either move into the directory or use full path to run it.</li>
  </ul>
  <pre><code>.\EvtxECmd.exe -f 'C:\Users\user\Desktop\Incident Files\sysmon.evtx' --csv 'C:\Users\user\Desktop\Incident Files' --csvf sysmon.csv</code></pre>
  
  <p align="center">
    <img src="https://github.com/user-attachments/assets/2688072b-c285-4641-94e4-cdab09a6e907" alt="Timeline Explorer" width="900" />
  </p>

  <p>Open the <code>sysmon.csv</code> in Timeline Explorer. Remember to follow up the description of tasks and challenges to be easier to accomplish the final objective. For this one, I will highlight it for you: 
  <ul>
    <li>The malicious document has a <code>.doc</code> extension.</li>
    <li>The user downloaded the malicious document via <code>chrome.exe</code>.</li>
    <li>The malicious document then executed a chain of commands to attain code execution.</li>
  </ul>

  When a file is downloaded, it will be registered in Sysmon with <b>Event ID 11</b>. So, filter for <b>Event ID 11</b> to locate files created and search for <code>.doc</code> extension:</p>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/c00c00e8-4f21-4fca-b6b0-7ae0535f7767" alt="Timeline Explorer" width="900" />
  </p>
  <ul>
    <li><b>Suspicious file found:</b> <code>free_magicules.doc</code></li>
    <li><b>Compromised user:</b> <code>benimaru</code></li>
  </ul>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/ed3a4640-4304-408e-abab-a56ef83bf56b" alt="Timeline Explorer" width="900" />
  </p>
  <ul>
    <li><b>Machine:</b><code>benimaru-TEMPEST</code></li>
  </ul>

<i><b>Note:</b> If you filter by Event ID 1 you will find the same information in this case, but it's only detected because the file has been executed, as mentioned in the room.</i>

<p>Since we are looking for an IPv4 address resolved by the malicious domain, we will search for made DNS queries correlating with the Process ID that we found. Let's take a look at our results:</p>
<p align="center">
    <img src="https://github.com/user-attachments/assets/60d7b0a6-ff91-47a6-9d70-f83d8a2cde44" alt="Timeline Explorer" width="900" />
  </p>
<br>
<p>If we take a look at the request's payload we can see the IPv4 address we are looking for:</p>
<p align="center">
    <img src="https://github.com/user-attachments/assets/16f0f746-4ae2-4c70-ad59-499d338f5483" alt="Timeline Explorer" width="900" />
  </p>
What is the base64 encoded string in the malicious payload executed by the document? Let's see... 

Searching for <code>Process ID: 496</code> we can check for <b>Executable Info</b> column and analysing it, we will find a suspicious command line:
<p align="center">
    <img src="https://github.com/user-attachments/assets/372ffcf9-dac2-4a89-a199-66430927fb60" alt="Timeline Explorer" width="900" />
  </p>

Using CyberChef, we can decode Base64 string:
<p align="center">
    <img src="https://github.com/user-attachments/assets/0ac326de-0492-4b75-baca-95e9feef495f" alt="Timeline Explorer" width="900" />
  </p>

By curiosity, I dived into the command line to understand the execution.
<p>
    The command launches <code>msdt.exe</code> with a specially crafted <code>ms-msdt</code> URI, exploiting a known method 
    (similar to the <strong>Follina vulnerability</strong>) to execute arbitrary code via diagnostic tools. This is a classic 
    <strong>Living Off the Land Binary (LOLBIN)</strong> attack—using built-in Windows tools to avoid detection.
  </p>
 <p><code>msdt.exe ms-msdt:/id PCWDiagnostic ...</code>: This triggers a Windows diagnostic task, often used in legitimate troubleshooting, but abused here for code execution.</p>

  <p><code>/param "... $(Invoke-Expression(...))"</code>: This parameter includes a PowerShell expression injection, which uses:</p>
  <ul>
    <li><code>Invoke-Expression</code> – executes strings as PowerShell code.</li>
    <li><code>FromBase64String(...)</code> – decodes a base64-encoded command.</li>
  </ul>

  <p>The decoded base64 command does this:</p>
  <ul>
    <li>Gets the path to the <code>ApplicationData</code> folder.</li>
    <li>Changes directory to the <strong>Startup folder</strong>.</li>
    <li>Downloads a file (<code>update.zip</code>) from <code>hxxp[://]phishteam[.]xyz/02dcf07/update[.]zip</code>code>.</li>
    <li>Extracts the ZIP file.</li>
    <li>Deletes the ZIP file after extraction.</li>
  </ul>

  <p>Ends with path to <code>mpsigstub.exe</code>. Likely used to bypass detection or escalate privileges. <code>mpsigstub.exe</code> is a signed Microsoft binary that attackers 
    sometimes use for proxy execution.  </p>

After this, I Googled <code>msdt.exe rce</code> and I found the answer for our last question:
<p align="center">
    <img src="https://github.com/user-attachments/assets/a3a8cc33-97e3-4a6d-a31c-a48e7b08b09e" alt="Timeline Explorer" width="900" />
  </p>
  <br>
<hr/>

<br>



 <h2>🔍 Task 5: Initial Access - Stage 2 execution</h2>



<hr/>
</body>
</html>
