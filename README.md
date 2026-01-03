# Slough - Dev Container - ESP32

A Docker-based development container for ESP32 projects with ESP-IDF v5.5.2, providing a complete and consistent development environment.

## About the Slough Project

This container is part of the **Slough Project** by [Daryl Stark](https://github.com/DarylStark). The Slough project aims to deliver consistent development tooling and environments across different projects using development containers (dev containers). By standardizing the development environment, Slough ensures that all developers work with the same tools, dependencies, and configurations, eliminating "works on my machine" issues.

## Quick Start

### Using as a Dev Container

This container is designed to work seamlessly with Visual Studio Code's Dev Containers extension or any other tool that supports the dev container specification.

#### Image Information

- **Docker Image**: `dast1968/slough-dev-dc-esp32:1.0.0`
- **Base Image**: `dast1986/slough-dev-dc-cpp:1.0.0`
- **ESP-IDF Version**: v5.5.2

#### Setup Instructions

1. **Install Prerequisites**:
   - [Docker](https://www.docker.com/products/docker-desktop)
   - [Visual Studio Code](https://code.visualstudio.com/)
   - [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

2. **Create `.devcontainer/devcontainer.json`** in your ESP32 project:

   ```json
   {
     "name": "ESP32 Development",
     "image": "dast1968/slough-dev-dc-esp32:1.0.0",
     "customizations": {
       "vscode": {
         "extensions": [
           "ms-vscode.cpptools",
           "ms-vscode.cmake-tools"
         ]
       }
     },
     "mounts": [
       "source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind"
     ],
     "runArgs": [
       "--privileged"
     ]
   }
   ```

3. **Open in Dev Container**:
   - Open your project in VS Code
   - Press `F1` or `Ctrl+Shift+P` (Windows/Linux) / `Cmd+Shift+P` (macOS)
   - Select "Dev Containers: Reopen in Container"

4. **Start Developing**:
   - The ESP-IDF environment is automatically activated in the shell
   - All ESP-IDF tools are ready to use

## Working with Dev Containers

### General Tips

- **First Launch**: The first time you open a dev container, Docker needs to pull the image, which may take a few minutes
- **Persistence**: Files in your project directory persist between container sessions
- **Port Forwarding**: VS Code automatically forwards ports from the container to your host
- **Extensions**: Install VS Code extensions inside the container for a consistent experience

### Tips for Microsoft Windows Users

1. **Use WSL 2 Backend**:
   - Docker Desktop on Windows should use the WSL 2 backend for better performance
   - Enable "Use the WSL 2 based engine" in Docker Desktop settings

2. **File Performance**:
   - Store your projects in the Linux filesystem (e.g., `\\wsl$\Ubuntu\home\username\projects`) for better I/O performance
   - Avoid storing projects on Windows drives (`C:\`) when using WSL 2

3. **Line Endings**:
   - Configure Git to use LF line endings: `git config --global core.autocrlf input`
   - This prevents issues with scripts inside the Linux container

4. **Serial Port Access**:
   - To access COM ports from WSL 2, you may need to use [usbipd-win](https://github.com/dorssel/usbipd-win)
   - Alternatively, run Docker Desktop with Hyper-V and pass through USB devices

5. **Permissions**:
   - WSL 2 handles file permissions differently than Windows
   - Files created in the container will have appropriate Linux permissions

## Container Configuration

### User Account

- **Username**: `developer`
- **Home Directory**: `/home/developer`
- **Sudo Access**: Available **without password** - you can run `sudo` commands without entering a password

### ESP-IDF Installation

- **Location**: `/home/developer/esp/esp-idf`
- **Version**: v5.5.2
- **Auto-activation**: The ESP-IDF environment is automatically sourced in `.bashrc.local`

### Shell Environment

- **Shell**: Bash with Starship prompt
- **Prompt Name**: "ESP32" - helps identify the container environment
- **Working Directory**: Opens in your project directory

## Installed Tools

This container comes with a comprehensive set of tools for ESP32 development:

### ESP32 Development Tools

- **ESP-IDF v5.5.2**: Complete Espressif IoT Development Framework
  - All ESP32 chip variants supported (ESP32, ESP32-S2, ESP32-S3, ESP32-C3, ESP32-C6, etc.)
  - Build system (CMake/Ninja)
  - Toolchains for all architectures

### Build Tools

- **CMake**: Build system generator
  - Usage: `idf.py build` (uses CMake internally)
- **Ninja**: Fast build system
  - Automatically used by ESP-IDF build system
- **ccache**: Compiler cache for faster rebuilds
  - Automatically configured with ESP-IDF

### Programming & Flashing Tools

- **dfu-util**: Device Firmware Upgrade utility
  - Usage: Flash firmware via DFU mode
- **libusb-1.0-0**: USB device access library
  - Required for serial and JTAG communication

### Development Utilities

- **Git**: Version control
  - Usage: `git clone`, `git commit`, etc.
- **Python 3**: Programming language with pip and venv
  - ESP-IDF uses Python for build scripts
  - Usage: `python3 -m venv myenv` to create virtual environments
- **wget**: File downloader
  - Usage: `wget https://example.com/file.bin`
- **flex & bison**: Parser generators
  - Used by some ESP-IDF components
- **gperf**: Perfect hash function generator
  - Used for efficient lookup tables

### Serial Communication

- **screen**: Terminal emulator for serial communication
  - Usage: `screen /dev/ttyUSB0 115200`
  - Exit: Press `Ctrl+A` then `K` then `Y`
  - Alternative: Use `idf.py monitor`

### Compiler Tools

- **psmisc**: Process management utilities
  - Includes `killall`, `pstree`, etc.

## Using ESP-IDF

The ESP-IDF environment is automatically activated when you open a terminal in the container. You can use all ESP-IDF commands directly:

### Common Commands

```bash
# Create a new project from a template
idf.py create-project my_project

# Or copy an example
cp -r $IDF_PATH/examples/get-started/hello_world ./my_project
cd my_project

# Configure the project
idf.py menuconfig

# Build the project
idf.py build

# Flash to device (requires USB device passthrough)
idf.py flash

# Monitor serial output
idf.py monitor

# Flash and monitor in one command
idf.py flash monitor

# Clean build artifacts
idf.py fullclean
```

### Setting Target Chip

```bash
# Set target to ESP32
idf.py set-target esp32

# Set target to ESP32-S3
idf.py set-target esp32s3

# Set target to ESP32-C3
idf.py set-target esp32c3
```

### Building for Production

```bash
# Build optimized for size
idf.py menuconfig  # Set optimization level in Compiler options
idf.py build

# Generate binary for OTA updates
idf.py build
# Output in build/ directory
```

## Accessing USB Devices

To flash ESP32 devices, you need to pass through USB serial devices to the container:

### Linux Host

Add to your `devcontainer.json`:

```json
{
  "runArgs": [
    "--device=/dev/ttyUSB0",
    "--privileged"
  ]
}
```

### Windows Host (WSL 2)

Use [usbipd-win](https://github.com/dorssel/usbipd-win) to attach USB devices to WSL 2:

```powershell
# In PowerShell (as Administrator)
usbipd list
usbipd bind --busid <BUSID>
usbipd attach --wsl --busid <BUSID>
```

### macOS Host

Add to your `devcontainer.json`:

```json
{
  "runArgs": [
    "--device=/dev/cu.usbserial-*"
  ]
}
```

## Building This Container

If you want to build the container yourself:

```bash
# Clone the repository
git clone https://github.com/DarylStark/slough-dev-dc-esp32.git
cd slough-dev-dc-esp32

# Build the container
docker build -t dast1968/slough-dev-dc-esp32:1.0.0 ./src

# Or use the VS Code task (Ctrl+Shift+B)
```

## License

This project is licensed under the MIT License. See [LICENSE.md](LICENSE.md) for details.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Related Projects

- [ESP-IDF](https://github.com/espressif/esp-idf) - Espressif IoT Development Framework
- [Slough Base Containers](https://github.com/DarylStark?tab=repositories&q=slough-dev-dc) - Other Slough development containers

## Support

For issues specific to this container, please [open an issue](https://github.com/DarylStark/slough-dev-dc-esp32/issues).

For ESP-IDF related questions, refer to the [ESP-IDF documentation](https://docs.espressif.com/projects/esp-idf/en/latest/).
