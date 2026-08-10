# Open HiBy Player

**TODOS**: look at [TODO.md](TODO.md) to see what has to be done

**Terminology**:
- "host device", "host": This refers to the device you are developing on, such as your laptop or PC.
- "target device", "target": This refers to the device you are developing for, such as the R1 or R3Pro II.

**Supported Devices:**
As I only have an R3Pro II, that's what's currently supported. But very very little goes into "porting" it to other devices, such as the R1. The only notable differences are the screen size and lack of the previous track button.

## 1. Local Development (Linux Host Simulation)

Running the GUI simulated on the host system (i.e. your laptop).

### Requirements
The following packages are required:
- `sdl2`
- `make`
- `gcc`
- `pkg-config`
- `git`

I do not use Windows, so I'm not sure how developing this on Windows would work. I think that just running everything like you would on linux through WSL (Windows Subsystem Linux) should work?

### Running the Simulator
Run the default Makefile build target:
```bash
make host
```
This command automatically clones the LVGL repository (if not already done) and compiles the project for your host architecture.

Launch the compiled executable:
```bash
./open_hiby_player_host
```
This will open an SDL2 graphical window. Clicking and stuff works. Scrolling is handled through holding click and dragging.

Note that the battery display doesnt work. That's intended. No real point in hooking that up for host testing.

---

## 2. Cross-Compiling for the HiBy Device (MIPS Target)

Building a binary that can be placed on the HiBy OS device.

### Build for Target
Build the target-specific binary:
```bash
make target
```
This builds the cross compilers, then generates the binary `open_hiby_player_target`. This generated binary can be transfered to the device, and run from anywhere.

> ![TIP]
> You can use `-j$(nproc)` to speed up builds significantly. Or you can manually set the number of compile threads. `-j4` sets it to 4 threads, for example.
> 
> So in practice, that would be running `make target -j$(nproc)` to make the target build with multithreading.

## My Workflow

### Host Testing

To quickly compile and run the host binary, i use: `make host -j $(nproc) && ./open_hiby_player_host`. This compiles using all CPU cores then runs the generated binary in one go.

### Target Testing

To quickly compile and transfer to target i use: `make target -j$(nproc) && adb push ./open_hiby_player_target /usr/data/mnt/sd_0/open_hiby_player`. For this to work you need your target device to have ADB turned on and for it to be plugged into your host device. This command will compile the target binary using all CPU cores then transfer it to the target's SD card.

Putting it on the SD card limits wear and tear on the device's built-in memory.

Then to run the program on the device is use `adb shell` to connect to it, then do the following:
1. Restart the device if it was playing music. I'm not sure how to safely unbind/rebind the audio backend from a program that was using it before.
2. Close whatever's currently running on the target device to make sure no other programs are trying to use the device's screen 
    - `killall -9 hiby_player.sh && killall -9 hiby_player` (if it was running the stock UI)
    - `killall -9 bootloader.r3proii` or `killall -9 bootloader.r1` (depending on the device, if it was running the rockbox bootloader)
3. Run the open hiby player `/usr/data/mnt/sd_0/open_hiby_player`

### Modifying Unpacked Firmware
(TODO, instructions on how to add open_hiby_player to a `.upt` file or something)
