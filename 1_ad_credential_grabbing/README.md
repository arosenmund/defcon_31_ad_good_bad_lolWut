# Credential Grabbing

Ryan to include notes here. I got this, go away.

## [Common Credential Grabbing Attacks](./1_credentual_grabbing/README.md)


Ryan to include notes here. I got this, go away.

## [Detecting Common Attacks](./2_detection_credential_grabbing/README.md)

Ryan to include notes here. I got this, go away.

## [Advanced Credential Grabbing & Evading Detections](./3_advanced_credential_grabbing/README.md)

One of the issues with Mimikatz, is just how popular it is.  Because of this exact popularity, many anti-virus, and EDR solutions have numerous signature based and behavioral detections for such tools.

To evade these detections, more advanced attackers will create custom tools, generally modular so they are only loading what is necesary to accomplish the job.  Generally, these will be injected into a process, and will be custom compiled for each implementation, variating the signature even in memory.  This evades the following detections:
- On write file signature detection
- Process in memory not backed by an executable
- Memory scans identifying known bad hex bytes

For this worskhop we won't be getting into all the proecess injection, but we will be dumping lsass with custom tools with up to date defender turned back on.
