# Docker

-   A containerisation platform
-   Helps you Package applications along with everything they need(code, libraries, system tools, dependencies)
-   https://github.com/iam-veeramalla/Docker-Zero-to-Hero

Properties: - Lightweight (Don’t have complete OS, Base OS that is enough to create a virtual isolation from other containers running on the underlying OS) - Portability - runs same everywhere - Fast Startup - seconds vs minutes in VMs - Easy Scaling - runs multiple containers quickly

## WHY?

-   Solved WORKS ON MY MACHINE problem as applications would behave differently in different:
    -   Operating systems
    -   Installed libraries and dependencies
    -   Configurations

VM:

```
Allocate resources, they won’t get used up for most of the time
[Hardware]
└── [Host OS]
└── [Hypervisor]
├── VM1 -> Guest OS + App
├── VM2 -> Guest OS + App
└── VM3 -> Guest OS + App
```

Hypervisors:
Software that lets you run multiple virtual machines

-   Type 1 (Bare Metal Hypervisor) - Runs directly on hardware - Microsoft Hyper - V
-   Type 2 (Hosted Hypervisor) - Runs on top of a operating system

```
Docker:
[Hardware]
└── [Host OS + Docker Engine]
├── Container1 -> App + Dependencies
├── Container2 -> App + Dependencies
└── Container3 -> App + Dependencies
```

### Docker Deamon(dockerd)

-   listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.
-   A daemon can also communicate with other daemons to manage Docker services.

### Docker Client

-   When we use docker run, client sends commands to dockerd, which carries them out

Dockerfile is a file where you provide the steps to build your Docker Image.
