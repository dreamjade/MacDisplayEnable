# MacDisplayEnable

This utility forces macOS to re-recognize and enable attached monitors. It is particularly useful for fixing issues where external monitors, KVM switches, or docks fail to wake up from sleep or get stuck on a black screen.

Under the hood, it uses Python and `ctypes` to interact with a private Apple `CoreGraphics` API (`CGSConfigureDisplayEnabled`) to manually cycle and enable display ports.

## Prerequisites

Because this script interacts with core macOS developer libraries, you must accept the Xcode developer license before running it. If you skip this step, macOS will block the script from executing.

1. Open your **Terminal**.
2. Run the following command:
   ```bash
   sudo xcodebuild -license
   ```

3. Enter your Mac login password when prompted and agree the license.

## Installation: Creating a Clickable App

To make this script easy to use without opening Terminal every time, you can wrap it into a standalone macOS application using Automator.

1. Open **Automator** (search for it using Spotlight: `Command + Space`).
2. Click **New Document** and select **Application** as the document type.
3. In the top-left search bar, search for **Run Shell Script** and drag it into the main workflow area on the right.
4. Set the **Shell** dropdown to `/bin/zsh` or `/bin/bash`.
5. Leave **Pass input** set to `to stdin`.
6. Copy the code block below and paste it into the text box, replacing any default text (like `cat`):

```bash
/usr/bin/env python3 << 'EOF'
from ctypes import (CDLL, util, c_void_p, c_uint32, c_int, c_bool, POINTER, byref)

cg = CDLL(util.find_library('CoreGraphics'))

cg.CGBeginDisplayConfiguration.argtypes = [POINTER(c_void_p)]
cg.CGBeginDisplayConfiguration.restype = c_int
cg.CGCompleteDisplayConfiguration.argtypes = [c_void_p, c_int]
cg.CGCompleteDisplayConfiguration.restype = c_int
cg.CGCancelDisplayConfiguration.argtypes = [c_void_p]
cg.CGCancelDisplayConfiguration.restype = c_int
cg.CGSConfigureDisplayEnabled.argtypes = [c_void_p, c_uint32, c_bool]
cg.CGSConfigureDisplayEnabled.restype = c_int

def enable_display(display_id):
    config_ref = c_void_p()
    if cg.CGBeginDisplayConfiguration(byref(config_ref)) != 0:
        return
    if cg.CGSConfigureDisplayEnabled:
        if cg.CGSConfigureDisplayEnabled(config_ref, display_id, True) != 0:
            cg.CGCancelDisplayConfiguration(config_ref)
            return
    cg.CGCompleteDisplayConfiguration(config_ref, 0)

def reset_displays():
    for display_id in range(1, 11):
        enable_display(display_id)

if __name__ == "__main__":
    reset_displays()
EOF

```

7. Press **Command + S** to save your new application. Name it something memorable (e.g., "Wake Monitors") and save it to your `Applications` folder.

## Usage

Whenever your monitors fail to wake up, simply double-click the app you just created. It will execute the reset sequence in the background and force macOS to ping the display ports.

Here is the updated Markdown file with the source link added in an Acknowledgments section at the bottom. You can copy and paste this directly into your GitHub repository.

## Source & Acknowledgments

The core Python script and approach were originally shared and discussed in [displayplacer issue #137 on GitHub](https://github.com/jakehilborn/displayplacer/issues/137#issuecomment-2787781728).
