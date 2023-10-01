## Nexus3



##### Inspect 查询分析其中内容

```
docker inspect --format='{{ XXX }}' $(docker ps -aq)

一级属性{{.属性}} 二级属性 {{.属性.属性}} 三级属性 {{.属性.属性.属性}}

docker inspect --format='{{ .NetworkSettings.IPAddress }}' $(docker ps -aq)
```

##### inspect nexus3 内容

```json
[
    {
        "Id": "ab9aae9e126e040712448a5f7c70395e11ef4bc45bf3cadf38841d829e92bd8c",
        "Created": "2019-08-22T08:36:03.27095745Z",
        "Path": "sh",
        "Args": [
            "-c",
            "${SONATYPE_DIR}/start-nexus-repository-manager.sh"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 99307,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-08-24T02:47:29.9671894Z",
            "FinishedAt": "2019-08-22T11:48:58.5389278Z"
        },
        "Image": "sha256:35ca857d5b19078b80ee1ce18825e172824a2402d313f2fd6ac2912750704a15",
        "ResolvConfPath": "/var/lib/docker/containers/ab9aae9e126e040712448a5f7c70395e11ef4bc45bf3cadf38841d829e92bd8c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/ab9aae9e126e040712448a5f7c70395e11ef4bc45bf3cadf38841d829e92bd8c/hostname",
        "HostsPath": "/var/lib/docker/containers/ab9aae9e126e040712448a5f7c70395e11ef4bc45bf3cadf38841d829e92bd8c/hosts",
        "LogPath": "/var/lib/docker/containers/ab9aae9e126e040712448a5f7c70395e11ef4bc45bf3cadf38841d829e92bd8c/ab9aae9e126e040712448a5f7c70395e11ef4bc45bf3cadf38841d829e92bd8c-json.log",
        "Name": "/nexus3",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/Users/mac/Documents/data/nexus-data:/nexus-data"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "8081/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8081"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "shareable",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": true,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": [
                "label=disable"
            ],
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": null,
            "ReadonlyPaths": null
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/275518777b60f7ec86d697e6c9b418eea6ef5c8b26bb27c41a93160d9c821570-init/diff:/var/lib/docker/overlay2/be73b63d4f2530166af2adba6ea136f0c86995ca936d819786e5b4127cf07daa/diff:/var/lib/docker/overlay2/a5c422b5a032ccd2e13b35bca41c41054fd32158c99a0de36975608c03631719/diff:/var/lib/docker/overlay2/5e4f15960681d92d5814532071014d2ec3f48d8a9ea4507db8f5edb021581d18/diff:/var/lib/docker/overlay2/81c6cdca1b2acaba87f41d50ceaa216a544d4f5433bd2f9260fcbfc7760dd34e/diff",
                "MergedDir": "/var/lib/docker/overlay2/275518777b60f7ec86d697e6c9b418eea6ef5c8b26bb27c41a93160d9c821570/merged",
                "UpperDir": "/var/lib/docker/overlay2/275518777b60f7ec86d697e6c9b418eea6ef5c8b26bb27c41a93160d9c821570/diff",
                "WorkDir": "/var/lib/docker/overlay2/275518777b60f7ec86d697e6c9b418eea6ef5c8b26bb27c41a93160d9c821570/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/Users/mac/Documents/data/nexus-data",
                "Destination": "/nexus-data",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "ab9aae9e126e",
            "Domainname": "",
            "User": "nexus",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8081/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "container=oci",
                "SONATYPE_DIR=/opt/sonatype",
                "NEXUS_HOME=/opt/sonatype/nexus",
                "NEXUS_DATA=/nexus-data",
                "NEXUS_CONTEXT=",
                "SONATYPE_WORK=/opt/sonatype/sonatype-work",
                "DOCKER_TYPE=rh-docker",
                "INSTALL4J_ADD_VM_PARAMS=-Xms1200m -Xmx1200m -XX:MaxDirectMemorySize=2g -Djava.util.prefs.userRoot=/nexus-data/javaprefs"
            ],
            "Cmd": [
                "sh",
                "-c",
                "${SONATYPE_DIR}/start-nexus-repository-manager.sh"
            ],
            "ArgsEscaped": true,
            "Image": "sonatype/nexus3",
            "Volumes": {
                "/nexus-data": {}
            },
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "architecture": "x86_64",
                "authoritative-source-url": "registry.access.redhat.com",
                "build-date": "2019-07-23T16:20:54.720447",
                "com.redhat.build-host": "cpt-1006.osbs.prod.upshift.rdu2.redhat.com",
                "com.redhat.component": "ubi8-container",
                "com.redhat.license_terms": "https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI",
                "com.sonatype.license": "Apache License, Version 2.0",
                "com.sonatype.name": "Nexus Repository Manager base image",
                "description": "The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications, middleware and utilities. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly.",
                "distribution-scope": "public",
                "io.k8s.description": "The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications, middleware and utilities. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly.",
                "io.k8s.display-name": "Red Hat Universal Base Image 8",
                "io.openshift.expose-services": "",
                "io.openshift.tags": "base rhel8",
                "maintainer": "Sonatype <cloud-ops@sonatype.com>",
                "name": "ubi8",
                "release": "154",
                "summary": "Provides the latest release of Red Hat Universal Base Image 8.",
                "url": "https://access.redhat.com/containers/#/registry.access.redhat.com/ubi8/images/8.0-154",
                "vcs-ref": "b80158ff460c2cd8958e35c55b0c7f50ca17638e",
                "vcs-type": "git",
                "vendor": "Sonatype",
                "version": "8.0"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "4cca5961632cd4185af4c3e43df54635cc926b08af673c934b9cf49701663559",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "8081/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8081"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/4cca5961632c",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "55b73b6b5689146669244f64bf242b4e21b69efef71cd2834c5fe143de1ccdb1",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "05811ceb2b3b6d0a8954d5139ff03513712a5fb842b30fe79253d2d09e24c8dd",
                    "EndpointID": "55b73b6b5689146669244f64bf242b4e21b69efef71cd2834c5fe143de1ccdb1",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

