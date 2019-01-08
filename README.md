# Docker for Windows Knowledge Base

- [HNS Issues](#hns)
- [MTU](#mtu)
- [Blob File](#blob)

## OS

Windows Server 2016 LTS, 1709, 1803, 1809 work well.
Windows Server 2019 not suppoted yet

<a name="hns"></a>
## HNS Issues :
Host Network Services (HNS) are a bit messy when it comes to Docker crashes or re-installs, or even Windows Upgrades. You might run the following two solutions to make your HNS with Docker Networks work again.

1) HNS/Docker stops working after Windows Upgrade
```
mofcomp C:\Windows\System32\wbem\NetNat.mof
```
Source: https://github.com/moby/moby/issues/27984


2) HNS/Docker works only partially or dockerd does not start anymore.
```
#leave swarm, remove node from ucp,
docker swarm leave

stop-service hns
stop-service docker

Get-ContainerNetwork | Remove-ContainerNetwork -Force
stop-service hns
del 'C:\ProgramData\Microsoft\Windows\hns\hns.data'

start-service hns
start-service docker

#Re-run windows script for preparing joining UCP.
#Join UCP.
docker swarm join
```

<a name="mtu"></a>
## MTU Size:
Currently you might get stuck in UCP with "Awaiting healthy status in classic node inventory"
This is because the UCP agent is set up with a MTU of 1500 while Windows default is less (1490).

One way to fix this is to change the MTU size of your Windows Host:
```
netsh interface ipv4 set subinterface "vEthernet (Ethernet0)" mtu=1500 store=persistent
```
Note: make sure the vEthernet Name in brackets matches your Interface name.
Source: https://github.com/docker/escalation/issues/848#issuecomment-442177830


<a name="blob"></a>
## Blob File Link:
To check Windows Manifests and versions you can find them here:
https://dockermsft.blob.core.windows.net/dockercontainer/DockerMsftIndex.json