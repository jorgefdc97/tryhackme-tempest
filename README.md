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

  <h2>🔍 Task 3: Preparation - Tools and Artifacts</h2>
  <p>In this task, we will prepare the artifacts and introduce the tools needed for the investigation. </p>
  <h3>Compare by hash</h3> 
  <p>Before conducting the investigation, one of the most important steps is to compare the artifacts by their hashes. It is a common practice to verify if the artifacts are expected as it is.<br><br>
    You can get the hashes of each artifact by running Powershell from the taskbar and executing the following command:</p>
  <pre><code>Get-FileHash * -Algorithm SHA256</code></pre>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/8b3530aa-0e3d-4ba2-8f3c-c70ad041e888" alt="Timeline Explorer" width="700" />
  </p>
  
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
    
  So, filter for <b>Event ID 11</b> to locate files created and search for <code>.doc</code> extension:</p>
  <p align="center">
    <img src="https://github.com/user-attachments/assets/c00c00e8-4f21-4fca-b6b0-7ae0535f7767" alt="Timeline Explorer" width="900" />
  </p>
  <ul>
    <li><b>Suspicious file found:</b> <code>free_magicules.doc</code></li>
    <li><b>Compromised user:</b> <code>benimaru</code></li>
  </ul>
  <p>
    <img src="https://github.com/user-attachments/assets/ed3a4640-4304-408e-abab-a56ef83bf56b" alt="Timeline Explorer" width="900" />
  </p>
  <ul>
    <li><b>Machine:</b><code>benimaru-TEMPEST</code></li>
  </ul>
  <hr/>

 

</body>
</html>
