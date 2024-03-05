# Hunting In the Shadows: CrowdStrike’s Advanced Threat Detection Prowess

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/c432c9e6-b9fa-402b-8a68-d3d4c6baaccb)

CrowdStrike Falcon is an enterprise-grade endpoint detection and response security product that helps security and incident response engineers identify potential threats on their networks. It is especially prolific at helping its engineers hunt threats and distinguish between detected threats and benign events, more commonly known as false positives. This lab will be an example of my prior work responsibilities while using CrowdStrike Falcon.

Using the CrowdStrike Hunting guide, I bolted together a query that would help with the team’s threat hunting efforts. This query helped identify malicious processes, connectivity, and behavior.

Here is the query that I commonly used to key in on common threats and cut through a lot of the noise.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/ad5a772d-a0ae-4c0b-9385-890a421b3d36)


The initial results of running this query regularly yielded thousands of events.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/a057afbc-bf43-4ee2-beca-e2b185a70f6d)


Altering the query to group common commands with the count helps cut down a lot of the non-relevant results.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/b3b7d0ad-1bce-4009-982a-21dd855db76d)


This also shows how frequently each command is being run.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/eb9cacc2-d2f0-4414-b727-d0e9caf4aa67)


The more common commands can typically be ignored since they usually signify benign tasks. We want to zero in on the unique tasks. The less frequent the process, the more suspect it is.

For instance, the process shown below warrants suspicion for a few reasons:

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/0124115e-3257-442d-8301-8b0577fe03dd)


It’s an executable called at.exe, it seems to be attempting to schedule a task each weekday and it’s passing through another command called escalate.exe. This process certainly warrants further attention:

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/649b6b89-37e9-4be3-821c-03a549282a6d)

To dig further we need to dig into the event information where relevant data is revealed like the IP address of the remote system and the hash of the executable.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/532e0cb1-8c41-41fb-b636-bce71d44b46e)

The beauty of CrowdStrike lies with its elegant interfaces and pivot processes that lets you seemingly go another layer deep. Here we will select the option to use the Explorer view.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/eedcd3a3-cf64-4e10-80d5-9d9e2d9eb8d2)


The process tree shows where the command to schedule the task was called by A.EXE:

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/11a1c4d5-afb1-458c-af48-ec787c463093)

So what exactly sticks out about this shady event that we haven’t already gleaned from looking at the event details?

Let’s take a further look at the command line data in the screenshot.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/d7d705d6-58ed-407f-b6aa-3376ea8bfbe9)

Things that immediately stick out:

Single letter executable names are not common in most environments.
Executables are not typically stored in the documents folder.
The accepteula parameter is suspicious — it is common with sysinternals. This may be indicating that it’s trying to use psexec.
escalate.exe is not a known tool or something that the infrastructure team is known to leverage.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/4d40a929-453f-4e41-b82b-38a51ee9b683)

As shown in the above screenshot, a.exe is shown as common both locally and globally so maybe inspecting the hash value will give us more details.

![image](https://github.com/InfoSecDion/CrowdStrike-Threat-Detection-Lab/assets/105241007/7f5d55e5-7651-4434-b850-29185b3e303b)

It appears the hash matched the sysinternals psexec and confirmed that a.exe was simply a renamed variation of this program.

So what does that really tell us? This is a spoofed copy of psexec that is trying to get elevated command line access to schedule a task that likely escalates account privileges. At this point I would open a formal incident in order to additionally investigate and remediate this confirmed threat.
