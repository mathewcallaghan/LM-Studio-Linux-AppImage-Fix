# LM Studio Linux AppImage Fix

-----

## Overview

This repository provides a simple, robust workaround for a known issue with the **LM Studio AppImage** on Linux, where the application's processes may not fully terminate after the GUI window is closed via its internal "Quit" menu option.

The solution consists of two independent Bash scripts:

  * **`lmstudio-launch`**: A simple launcher script that starts the LM Studio AppImage.
  * **`lmstudio-shutdown`**: A separate script designed to forcefully terminate any lingering LM Studio processes.

This approach allows users to continue using LM Studio normally and then explicitly run the shutdown script (e.g., via a separate desktop shortcut) when they want to ensure all LM Studio processes are stopped.

-----

## The Problem

When closing the LM Studio AppImage on Linux (e.g., by clicking "Quit" in the application menu or closing the window), the main graphical interface disappears, but underlying processes (often related to the LLM server or other Electron components) may continue to run in the background, consuming system resources. This is a known bug within the LM Studio AppImage itself.

-----

## The Solution

Instead of relying on LM Studio's internal quit mechanism, we provide two dedicated scripts:

  * **Launch Script**: Uses `exec` to hand over control directly to the LM Studio AppImage, ensuring a clean launch.
  * **Shutdown Script**: Uses `pkill -f "lm-studio"` to reliably find and terminate all processes associated with LM Studio. This script is run *after* you've closed the LM Studio GUI.

-----

## Usage

### 1\. Place the LM Studio AppImage

Download the LM Studio AppImage from the [official LM Studio website](https://lmstudio.ai/) and place it in a dedicated directory within your home folder.
By default, these scripts expect it in:
`$HOME/lmstudio/` (e.g., `/home/yourusername/lmstudio/LM-Studio-*.AppImage`)

### 2\. Create the Launch Script (`lmstudio-launch`)

Create a new file, for example, at `~/lmstudio/scripts/lmstudio-launch`, and add the following content:

```bash
#!/bin/bash

# Define the expected subdirectory for AppImages relative to the user's home
APPIMAGE_DIR="$HOME/lmstudio"

# Path to LM Studio AppImage
# Look for LM-Studio*.AppImage in the specified directory
APPIMAGE=$(ls "$APPIMAGE_DIR"/LM-Studio*.AppImage 2>/dev/null | head -n 1)

# Check if AppImage exists
if [ -z "$APPIMAGE" ]; then
    echo "Error: No LM-Studio AppImage found in '$APPIMAGE_DIR'."
    echo "Please ensure the LM Studio AppImage is placed in this directory."
    exit 1
fi

# Optional: unset LD_PRELOAD to avoid memory library conflicts
unset LD_PRELOAD

# Launch with --no-sandbox (required for Electron in AppImage)
exec "$APPIMAGE" --no-sandbox
```

Make it executable:

```bash
chmod +x ~/lmstudio/scripts/lmstudio-launch
```

### 3\. Create the Shutdown Script (`lmstudio-shutdown`)

Create another new file, for example, at `~/lmstudio/scripts/lmstudio-shutdown`, and add the following content:

```bash
#!/bin/bash

echo "Attempting to terminate all LM Studio processes..."
pkill -f "lm-studio"

if [ $? -eq 0 ]; then
    echo "LM Studio processes successfully terminated."
else
    echo "No LM Studio processes found or an error occurred during termination."
fi
```

Make it executable:

```bash
chmod +x ~/lmstudio/scripts/lmstudio-shutdown
```

### 4\. Create Desktop Launchers (Optional but Recommended)

For convenience, you can create `.desktop` files to easily launch and shut down LM Studio from your application menu or desktop.

**`lmstudio.desktop` (for launching):**

```desktop
[Desktop Entry]
Name=LM Studio
Comment=Discover, download, and run local LLMs
Exec=/home/YOUR_USERNAME/lmstudio/scripts/lmstudio-launch
Icon=/home/YOUR_USERNAME/lmstudio/icons/lm-studio-icon.svg
Terminal=false
Type=Application
Categories=Utility;Development;AI;
```

Replace `/home/YOUR_USERNAME/` with your actual home directory path.

**`lmstudio-quit.desktop` (for shutting down):**

```desktop
[Desktop Entry]
Name=Quit LM Studio
Comment=Forcefully terminate all LM Studio processes
Exec=/home/YOUR_USERNAME/lmstudio/scripts/lmstudio-shutdown
Icon=/home/YOUR_USERNAME/lmstudio/icons/lm-studio-shutdown-icon.svg
Terminal=false
Type=Application
Categories=Utility;
```

Replace `/home/YOUR_USERNAME/` with your actual home directory path.

Place these `.desktop` files in `~/.local/share/applications/` to make them appear in your application menu. You might need to log out and back in, or refresh your desktop environment's menu cache for them to appear.

-----

## Disclaimer

The `pkill -f "lm-studio"` command forcefully terminates processes. While effective for this known bug, use it with caution and ensure you understand its implications. This script is provided as a workaround for a specific software issue.

-----

## Contributing

Feel free to open issues or pull requests if you have improvements or find further issues.

-----

## License

This project is open-source under the [MIT License](https://www.google.com/search?q=LICENSE).
