1. What is Docker and why is it used?
Docker is a containerization platform that packages an application and all its dependencies into a container, so it runs consistently across different environments.
How Docker Works 
• Applications run inside containers, which share the host OS kernel.
• Each container includes the app, libraries, and configs—but not a full OS.
Why Docker Is Used
• Consistency: “Works on my machine” problem is eliminated
• Lightweight & fast: Containers start faster than virtual machines
Simple Example
• App works in developer laptop → test → production without changes.
2. Difference between Docker and Virtual Machines
FeatureDocker (Containers)Virtual Machines (VMs)ArchitectureShares host OS kernelRuns its own full OSResource usageLightweightHeavyStartup timeSecondsMinutesPerformanceNear nativeSlight overheadIsolationProcess-level isolationFull OS-level isolationPortabilityVery highLower than containers
