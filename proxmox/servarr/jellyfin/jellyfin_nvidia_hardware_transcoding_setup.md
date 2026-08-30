# Jellyfin Hardware Transcoding with NVIDIA GPU Passthrough 

* Document is AI-Generated based on user prompts *

This guide documents the setup used to enable **NVIDIA hardware
transcoding in Jellyfin** when Jellyfin runs in a Docker container
inside an Ubuntu VM on Proxmox.

## Example hardware/software

This guide is based on:

-   Proxmox VE host
-   Intel CPU with Intel VT-d/IOMMU support
-   Intel UHD 630 used by the Proxmox host
-   NVIDIA GeForce RTX 2080 SUPER
-   Ubuntu 24.04 LTS VM
-   NVIDIA 580-series driver
-   Docker
-   NVIDIA Container Toolkit
-   LinuxServer.io Jellyfin Docker image
-   Jellyfin NVENC/NVDEC

> The commands and PCI IDs below are specific to the example RTX 2080
> SUPER. If using a different NVIDIA GPU, identify its PCI IDs with
> `lspci -nnk` first.

------------------------------------------------------------------------

# 1. BIOS/UEFI Configuration

Before configuring Proxmox, enter the motherboard BIOS/UEFI.

Enable:

-   **Intel VT-x / Intel Virtualization Technology**
-   **Intel VT-d**
-   **Above 4G Decoding** --- recommended for PCIe GPU passthrough

If available, it is generally best to disable:

-   **CSM / Legacy Boot**

For this setup, the NVIDIA GPU does not need to be the primary display
device. The Intel iGPU can remain the Proxmox host's display adapter.

------------------------------------------------------------------------

# 2. Verify IOMMU on Proxmox

Check whether the system detected the Intel DMAR/IOMMU tables:

``` bash
dmesg | grep -e DMAR -e IOMMU
```

You want to see lines similar to:

``` text
DMAR: IOMMU enabled
Intel(R) Virtualization Technology for Directed I/O
```

Check the IOMMU groups:

``` bash
find /sys/kernel/iommu_groups/ -type l
```

To find the NVIDIA GPU specifically:

``` bash
find /sys/kernel/iommu_groups/ -type l | grep '01:00'
```

Example:

``` text
/sys/kernel/iommu_groups/2/devices/0000:01:00.0
/sys/kernel/iommu_groups/2/devices/0000:01:00.1
/sys/kernel/iommu_groups/2/devices/0000:01:00.2
/sys/kernel/iommu_groups/2/devices/0000:01:00.3
```

The NVIDIA RTX 2080 SUPER appears as four PCI functions:

  PCI function   Device
  -------------- ------------------------------
  `01:00.0`      RTX 2080 SUPER GPU
  `01:00.1`      NVIDIA HD Audio
  `01:00.2`      NVIDIA USB 3.1 controller
  `01:00.3`      NVIDIA USB-C/UCSI controller

For Jellyfin hardware transcoding, start by passing through **only
`01:00.0`**.

The HD Audio function is only needed if the VM needs to use the GPU's
HDMI/DisplayPort audio. The USB/USB-C functions are not required for
Jellyfin transcoding.

------------------------------------------------------------------------

# 3. Enable Intel IOMMU in Proxmox

Edit:

``` bash
nano /etc/default/grub
```

For an Intel system, make sure the kernel command line contains:

``` text
intel_iommu=on
```

Example:

``` text
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

Then update GRUB:

``` bash
update-grub
```

Reboot if this setting was newly added:

``` bash
reboot
```

After reboot, verify:

``` bash
cat /proc/cmdline
```

You should see:

``` text
intel_iommu=on
```

Then verify again:

``` bash
dmesg | grep -e DMAR -e IOMMU
```

------------------------------------------------------------------------

# 4. Identify the NVIDIA GPU

Run:

``` bash
lspci -nn | grep -E "VGA|3D|Display"
```

Example:

``` text
00:02.0 VGA compatible controller [0300]: Intel Corporation CoffeeLake-S GT2 [UHD Graphics 630] [8086:3e98]
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2080 SUPER] [10de:1e81]
```

The Intel GPU is being used by the Proxmox host:

``` bash
lspci -nnk -s 00:02.0
```

Example:

``` text
Kernel driver in use: i915
```

This is useful because the host can continue using the Intel iGPU while
the NVIDIA GPU is passed to the VM.

Inspect the NVIDIA functions:

``` bash
lspci -nnk -s 01:00.0
lspci -nnk -s 01:00.1
lspci -nnk -s 01:00.2
lspci -nnk -s 01:00.3
```

------------------------------------------------------------------------

# 5. Add the GPU to the Proxmox VM

Shut down the target VM.

In the Proxmox web interface:

**VM → Hardware → Add → PCI Device**

Select:

``` text
01:00.0 NVIDIA GeForce RTX 2080 SUPER
```

For the initial Jellyfin setup:

-   Enable **PCI-Express** if available/recommended by your Proxmox
    configuration.
-   Do not enable **Primary GPU** for a normal headless Jellyfin VM.
-   Do not select all functions unless you specifically need the GPU's
    other PCI functions.

Start the VM.

If the VM starts successfully, verify the GPU from inside Ubuntu.

> If Proxmox reports that IOMMU is unavailable, return to the
> IOMMU/GRUB/BIOS steps above. If the VM starts with the GPU attached,
> the PCI passthrough path is functioning.

------------------------------------------------------------------------

# 6. Verify the GPU inside Ubuntu

Inside the Ubuntu VM:

``` bash
lspci -nnk | grep -A3 -E "NVIDIA|VGA|3D"
```

The RTX 2080 SUPER should be visible.

Initially, it may show:

``` text
Kernel driver in use: nouveau
```

This means Ubuntu sees the passed-through GPU, but the proprietary
NVIDIA driver has not yet taken control.

------------------------------------------------------------------------

# 7. Install the NVIDIA Driver

Install the NVIDIA driver in the **Ubuntu VM**, not inside the Jellyfin
container.

For the setup documented here, the 580-series driver was used.

First:

``` bash
sudo apt update
```

Then install the driver:

``` bash
sudo apt install nvidia-driver-580
```

Reboot:

``` bash
sudo reboot
```

After reboot:

``` bash
nvidia-smi
```

A successful result should show something similar to:

``` text
NVIDIA-SMI 580.x
Driver Version: 580.x
CUDA Version: ...
NVIDIA GeForce RTX 2080 SUPER
```

Also verify the NVIDIA device files:

``` bash
ls -l /dev/nvidia*
```

You should see devices such as:

``` text
/dev/nvidia0
/dev/nvidiactl
/dev/nvidia-modeset
/dev/nvidia-uvm
/dev/nvidia-uvm-tools
```

------------------------------------------------------------------------

# 8. Verify NVIDIA NVENC/NVDEC Availability

Run:

``` bash
nvidia-smi -q | grep -E "Encoder|Decoder"
```

It is normal to see:

``` text
Encoder : 0 %
Decoder : 0 %
```

when nothing is currently transcoding.

The important point is that the NVIDIA driver is installed and
`nvidia-smi` works.

------------------------------------------------------------------------

# 9. Install the NVIDIA Container Toolkit

Jellyfin is running inside Docker, so the Docker daemon needs a way to
expose the NVIDIA GPU to containers.

First, configure NVIDIA's official container repository.

Remove an incorrectly configured repository file if necessary:

``` bash
sudo rm -f /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Install the repository signing key:

``` bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
```

For an amd64 Ubuntu VM, create the repository:

``` bash
echo "deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://nvidia.github.io/libnvidia-container/stable/deb/amd64 /" | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Update APT:

``` bash
sudo apt update
```

Verify that the package is visible:

``` bash
apt-cache policy nvidia-container-toolkit
```

Install it:

``` bash
sudo apt install -y nvidia-container-toolkit
```

------------------------------------------------------------------------

# 10. Configure Docker for NVIDIA

Run:

``` bash
sudo nvidia-ctk runtime configure --runtime=docker
```

This configures Docker to use the NVIDIA container runtime.

It normally creates or modifies:

``` text
/etc/docker/daemon.json
```

Then restart the Docker daemon:

``` bash
sudo systemctl restart docker
```

## What does restarting Docker do?

It restarts the Docker daemon so it loads the new NVIDIA runtime
configuration.

Containers using:

``` yaml
restart: unless-stopped
```

should automatically come back after Docker restarts, but expect a short
service interruption.

You can verify running containers afterward:

``` bash
docker ps
```

------------------------------------------------------------------------

# 11. Test Docker GPU Access Before Touching Jellyfin

This is an important troubleshooting step.

Run:

``` bash
docker run --rm --gpus all nvidia/cuda:13.0.2-base-ubuntu24.04 nvidia-smi
```

The command:

-   Creates a temporary CUDA container.
-   Gives it access to NVIDIA GPUs with `--gpus all`.
-   Runs `nvidia-smi` inside the container.
-   Removes the temporary container when finished because of `--rm`.

A successful result should show:

``` text
NVIDIA GeForce RTX 2080 SUPER
Driver Version: ...
CUDA Version: ...
```

If this test fails, troubleshoot NVIDIA Container Toolkit before
changing Jellyfin.

------------------------------------------------------------------------

# 12. Configure the Jellyfin Docker Compose File

Example LinuxServer.io Jellyfin Compose file:

``` yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
      - JELLYFIN_PublishedServerUrl=http://192.168.1.236
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility

    volumes:
      - ./config:/config
      - /data:/data

    ports:
      - 8096:8096
      - 7359:7359/udp
      - 1900:1900/udp

    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

    restart: unless-stopped
```

The important NVIDIA parts are:

``` yaml
environment:
  - NVIDIA_VISIBLE_DEVICES=all
  - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
```

and:

``` yaml
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: 1
          capabilities: [gpu]
```

LinuxServer.io's Jellyfin image uses the NVIDIA container runtime to
expose the NVIDIA driver and GPU to the container.

------------------------------------------------------------------------

# 13. Recreate Jellyfin

From the Jellyfin directory:

``` bash
cd /docker/jellyfin
```

It is a good idea to make a backup of the Compose file first:

``` bash
cp compose.yaml compose.yaml.bak
```

Recreate Jellyfin:

``` bash
docker compose down
docker compose up -d
```

This only recreates the Jellyfin container, not your other Docker
services.

------------------------------------------------------------------------

# 14. Verify the Jellyfin Container Can See the GPU

Run:

``` bash
docker exec jellyfin nvidia-smi
```

You should see:

``` text
NVIDIA GeForce RTX 2080 SUPER
Driver Version: ...
CUDA Version: ...
```

This confirms:

``` text
Proxmox
  ↓
GPU passthrough
  ↓
Ubuntu VM
  ↓
NVIDIA driver
  ↓
NVIDIA Container Toolkit
  ↓
Docker
  ↓
Jellyfin container
  ↓
RTX 2080 SUPER
```

At this point the container has GPU access.

------------------------------------------------------------------------

# 15. Configure Hardware Acceleration in Jellyfin

Open the Jellyfin web interface.

Go to:

**Dashboard → Playback → Transcoding**

Set:

**Hardware acceleration:**

``` text
NVIDIA NVENC
```

Enable the codecs supported by the GPU.

For an RTX 2080 SUPER, typical useful codecs include:

-   H.264
-   HEVC
-   MPEG-2
-   VP9

Do **not** enable AV1 hardware encoding/decoding just because the
Jellyfin interface lists it. The RTX 2080 SUPER does not have AV1
hardware encode/decode support.

Enable:

``` text
Hardware encoding
```

For modern Jellyfin versions, **Enhanced NVDEC** can be enabled when
appropriate.

Save the configuration.

------------------------------------------------------------------------

# 16. Test an Actual Transcode

Start playing media in a client that would normally Direct Play the
file.

To force transcoding, select a lower playback quality/bitrate from the
Jellyfin player.

For example:

``` text
1080p → 720p
```

Then check the Jellyfin dashboard to confirm that the session says:

``` text
Transcoding
```

------------------------------------------------------------------------

# 17. Verify NVENC/NVDEC in Jellyfin Logs

Check the Jellyfin logs:

``` bash
docker logs jellyfin 2>&1 | grep -iE "nvenc|nvdec|cuda|transcod"
```

A successful NVIDIA transcode should contain entries similar to:

``` text
-init_hw_device cuda=cu:0
-hwaccel cuda
-hwaccel_output_format cuda
```

For NVIDIA hardware encoding, look for:

``` text
h264_nvenc
```

or:

``` text
hevc_nvenc
```

For CUDA hardware processing, you may also see filters such as:

``` text
scale_cuda
```

A real example of a successful HEVC transcode looks like:

``` text
-init_hw_device cuda=cu:0
-hwaccel cuda
-hwaccel_output_format cuda
-codec:v:0 hevc_nvenc
-vf "...scale_cuda..."
```

This proves Jellyfin is actually using the NVIDIA hardware rather than
merely detecting the GPU.

------------------------------------------------------------------------

# 18. Monitor the GPU During Transcoding

While a video is actively transcoding:

``` bash
docker exec jellyfin nvidia-smi
```

You may see:

``` text
GPU-Util: 10%
```

or higher GPU utilization depending on the workload.

You can continuously monitor it:

``` bash
watch -n 1 'docker exec jellyfin nvidia-smi'
```

GPU utilization and power consumption may increase during transcoding.

Do not expect the GPU to show high utilization for every transcode.
Video encoding/decoding uses dedicated hardware engines and the workload
varies by codec, resolution, bitrate, scaling, and other processing.

------------------------------------------------------------------------

# 19. Troubleshooting Checklist

## Proxmox says IOMMU is unavailable

Check:

``` bash
cat /proc/cmdline
```

Confirm:

``` text
intel_iommu=on
```

Then:

``` bash
dmesg | grep -e DMAR -e IOMMU
```

Also verify Intel VT-d is enabled in BIOS.

------------------------------------------------------------------------

## VM does not see the GPU

On Proxmox:

``` bash
lspci -nn | grep -E "VGA|3D|Display"
```

Check the IOMMU group:

``` bash
find /sys/kernel/iommu_groups/ -type l | grep '01:00'
```

Then check the VM's PCI hardware configuration in Proxmox.

------------------------------------------------------------------------

## Ubuntu sees the GPU but `nvidia-smi` does not work

Check:

``` bash
lspci -nnk | grep -A3 -E "NVIDIA|VGA|3D"
```

Then:

``` bash
nvidia-smi
```

If the GPU is using `nouveau` instead of the proprietary NVIDIA driver,
install/configure the NVIDIA driver.

------------------------------------------------------------------------

## Docker says:

``` text
could not select device driver "" with capabilities: [[gpu]]
```

The NVIDIA Container Toolkit is missing or Docker has not been
configured to use it.

Check:

``` bash
apt-cache policy nvidia-container-toolkit
```

Then:

``` bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Test:

``` bash
docker run --rm --gpus all nvidia/cuda:13.0.2-base-ubuntu24.04 nvidia-smi
```

------------------------------------------------------------------------

## Docker can see the GPU but Jellyfin cannot

Check:

``` bash
docker exec jellyfin nvidia-smi
```

If that fails, verify the Jellyfin Compose GPU configuration and
recreate the container:

``` bash
docker compose down
docker compose up -d
```

------------------------------------------------------------------------

## Jellyfin sees the GPU but uses the CPU

Check the Jellyfin logs:

``` bash
docker logs jellyfin 2>&1 | grep -iE "nvenc|nvdec|cuda|transcod"
```

Look for:

``` text
hwaccel cuda
```

and:

``` text
h264_nvenc
```

or:

``` text
hevc_nvenc
```

Also verify the current playback session is actually **Transcoding**,
not Direct Play or Direct Stream.

------------------------------------------------------------------------

# 20. Important Notes

## Direct Play does not use the GPU

If the client can play the original file directly, Jellyfin may use:

``` text
Direct Play
```

In that case, there may be little or no GPU activity.

Hardware transcoding is only relevant when Jellyfin needs to
decode/encode/process the media.

------------------------------------------------------------------------

## HDMI/DP audio is not required for Jellyfin

The RTX 2080 SUPER has:

``` text
01:00.0  GPU
01:00.1  HD Audio
01:00.2  USB
01:00.3  USB-C/UCSI
```

For a headless Jellyfin transcoding server, only:

``` text
01:00.0
```

is required to start.

The HD Audio function is useful when the VM needs the GPU's
HDMI/DisplayPort audio output.

------------------------------------------------------------------------

## Do not install NVIDIA drivers inside the Jellyfin container

The NVIDIA driver belongs on the Ubuntu VM.

The NVIDIA Container Toolkit provides the mechanism for Docker to expose
the GPU and driver libraries to the container.

The architecture should be:

``` text
Proxmox host
    │
    ├── Intel UHD 630 → Proxmox
    │
    └── RTX 2080 SUPER
            │
            │ PCI passthrough
            ▼
      Ubuntu 24.04 VM
            │
            ├── NVIDIA driver
            │
            ├── NVIDIA Container Toolkit
            │
            └── Docker
                    │
                    ▼
               Jellyfin
                    │
                    ▼
              NVDEC / CUDA
                    │
                    ▼
                  NVENC
```

------------------------------------------------------------------------

# 21. Final Verification

A complete working installation should pass all of these tests.

### Proxmox

``` bash
dmesg | grep -e DMAR -e IOMMU
```

Expected:

``` text
DMAR: IOMMU enabled
```

### VM

``` bash
nvidia-smi
```

Expected:

``` text
NVIDIA GeForce RTX 2080 SUPER
```

### Docker

``` bash
docker run --rm --gpus all nvidia/cuda:13.0.2-base-ubuntu24.04 nvidia-smi
```

Expected:

``` text
NVIDIA GeForce RTX 2080 SUPER
```

### Jellyfin container

``` bash
docker exec jellyfin nvidia-smi
```

Expected:

``` text
NVIDIA GeForce RTX 2080 SUPER
```

### Jellyfin transcoding

``` bash
docker logs jellyfin 2>&1 | grep -iE "nvenc|nvdec|cuda|transcod"
```

Look for:

``` text
-hwaccel cuda
```

and:

``` text
h264_nvenc
```

or:

``` text
hevc_nvenc
```

If all of these work, NVIDIA hardware transcoding is functioning
end-to-end.

------------------------------------------------------------------------

# Official References

-   Jellyfin --- NVIDIA Hardware Acceleration
-   Jellyfin --- Hardware Acceleration Overview
-   Jellyfin --- Container Installation
-   LinuxServer.io --- Jellyfin Docker Image
-   NVIDIA --- NVIDIA Container Toolkit Documentation
