# Credential Grabbing

When a TA obtains access to an environment, they do so under some form of account context. The user context under which the TA's processes run depends on the entry method. For example, a simple RDP session to a host means that you are logging in as a specific user. That one is a bit obvious. An exploit of an internet-facing service will most likely result in the remote code execution that runs under the service account tied to the exploited service.

In our case, you as the TA will obtain access to the victim environment via RDP using a local administrator account on the machine. Thus you will need to elevate overall privileges by gaining access to more privileged accounts. We will be showing you how the low-skilled attacker does so (*AHEM* ransomware *AHEM*), showing you how to detect these methods, and then wrap with much more advanced and custom-crafted methods to accomplish the task.

## [Common Credential Grabbing Attacks](./1_credential_grabbing/README.md)

Our initial credential grabbing attacks will occur by way of tools such as [Mimikatz](https://github.com/gentilkiwi/mimikatz). If you've spent much time working in infosec, the term "Mimikatz" should make you think "child's first attack tool." The tool is day one content in pentesting 101 courses. While signatures for Mimikatz are widely available, TAs still find quite a bit of success with the tool. We incident responders often find the tool in AV logs, as TAs often attempt to bring the tool in before they are even able to disable AV/EPP/EDR/etc.

## [Detecting Common Attacks](./2_detection_credential_grabbing/README.md)

Credential grabbing tools are noisy. _Very_ noisy. To begin, common toolkits can be detected easily via process execution monitoring. On top of detecting the tools themselves, Windows Event Logs (WELs), file system artifacts, and registry changes can be analyzed to identify the attacks themselves. In this section you will be learning about WEL Event ID 4703, how to identify credential attacks via Sysmon's process and registry modules, and how to monitor file system changes to detect credential attacks.

## [Advanced Credential Grabbing & Evading Detections](./3_advanced_credential_grabbing/README.md)

One of the issues with Mimikatz, is just how popular it is.  Because of this exact popularity, many anti-virus, and EDR solutions have numerous signature based and behavioral detections for such tools.

To evade these detections, more advanced attackers will create custom tools, generally modular so they are only loading what is necesary to accomplish the job.  Generally, these will be injected into a process, and will be custom compiled for each implementation, variating the signature even in memory.  This evades the following detections:
- On write file signature detection
- Process in memory not backed by an executable
- Memory scans identifying known bad hex bytes

For this worskhop we won't be getting into all the proecess injection, but we will be dumping lsass with custom tools with up to date defender turned back on.
