# Nordic-nRF52-VS-Code-template

In this guide, all necessary steps are documented to program and debug nRF52 series chips with VS Code.


# Quick Start Guide
- Follow Install Guide
- Run: ```vscode.bat```
- Including other directories via: ```c_cpp_properties:includePath``` and in ```Makefile```.
- Building: ```CTRL```+```ALT```+```T```.
- Debugging: ```F5```.

# Install Guide

## 1. Define locations

Run ```vscode.bat``` to start VS Code with predefined locations of SDK, GCC, JLINK, etc.

Change "username" to your windows user name.

Create a workspace in vs code ```workspace.code-workspace``` passing it when starting vs code.

Example setup config:

```
rem Location of Nordic SDK
set NRF_SDK=C:/Users/username/Box/NOMADe/SOFTWARE/nRF5_SDK_17.0.2_d674dde

rem Location of Nordic Command Line tools (nrfjprog) 
set NRF_TOOLS=C:/Program Files (x86)/Nordic Semiconductor/nrf-command-line-tools/bin

rem location of GCC Cross-compiler https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads
set GNU_GCC=C:/Program Files (x86)/GNU Arm Embedded Toolchain/10 2020-q4-major/bin

rem Location of Gnu Tools (make) https://github.com/gnu-mcu-eclipse/windows-build-tools/releases
set GNU_TOOLS=C:/Users/username/Box/NOMADe/SOFTWARE/nRF5_SDK_17.0.2_d674dde/xpack-windows-build-tools-4.2.1-2/bin

rem Location of SEGGER JLink tools
set SEGGER_TOOLS=C:/Program Files (x86)/SEGGER/JLink

rem Location of java
set JAVA=C:/Program Files/Java/jre1.8.0_251/bin/java.exe

rem Serial numbers of nRF development boards
set PCA10056_SN=000000000
set PCA10040_SN=682864789

start "C:/Users/u0139004/AppData/Local/Programs/Microsoft VS Code/Code.exe" workspace.code-workspace
exit
```


## 2. Setup VS Code: "c_cpp_properties.json"

Example configuration for pca10040 with Soft Device 132.

Preprocessor defines can be added under "defines".
Include paths can be added under "includePath".


```
{
    "configurations": [
        {
            "name": "pca10040/s132",
            "includePath": [
                "${workspaceFolder}/pca10040/s132/config",
                "${env:NRF_SDK}/**"
            ],
            "defines": [
                "BOARD_PCA10040",
                "CONFIG_GPIO_AS_PINRESET",
                "FLOAT_ABI_HARD",
                "NRF52",
                "NRF52832_XXAA",
                "NRF52_PAN_74",
                "NRF_SD_BLE_API_VERSION=6",
                "S132",
                "SOFTDEVICE_PRESENT",
                "SWI_DISABLE0"
            ],
            "compilerPath": "${env:GNU_GCC}/arm-none-eabi-gcc.exe",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64"
        }
    ],
    "version": 4
}
```

## 3. Setup build options: "tasks.json"

Caution: set "cwd": ```${workspaceFolder}/pca10040/s132/armgcc``` correctly to link to the ```armgcc``` folder for calling "make".
```
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "${env:GNU_TOOLS}/make",
            "options": {
                "cwd": "${workspaceFolder}/pca10040/s132/armgcc"
            },
            "problemMatcher": []
        }
    ]
}
```

## 4. Setup debugging: "launch.json"


Caution: set ```"program": "pca10040/s132/armgcc/_build/nrf52832_xxaa.out"``` to location where ```_build/nrf52832_xxaa.out``` is generated.
```"-file-exec-and-symbols pca10040/s132/armgcc/_build/nrf52832_xxaa.out"```should also be set correctly.


```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "pca10040/s132/armgcc/_build/nrf52832_xxaa.out",
            "cwd": "${workspaceFolder}",
            "MIMode": "gdb",
            "miDebuggerPath": "${env:GNU_GCC}/arm-none-eabi-gdb.exe",
            "targetArchitecture": "arm",
            "customLaunchSetupCommands": [
                { "text": "-environment-cd ${workspaceFolder}", "description": "set cwd", "ignoreFailures": false },
                { "text": "-file-exec-and-symbols pca10040/s132/armgcc/_build/nrf52832_xxaa.out", "description": "set executable", "ignoreFailures": false },
                { "text": "-interpreter-exec console \"set pagination off\"", "description": "set pagination off", "ignoreFailures": false },
                { "text": "-target-select remote localhost:2331", "description": "connect target", "ignoreFailures": false },
                { "text": "-break-insert main", "description": "break on main", "ignoreFailures": false },
                { "text": "-interpreter-exec console \"monitor reset\"", "description": "reset target", "ignoreFailures": false },
            ],
            "stopAtEntry": true,
            "miDebuggerServerAddress": "localhost:2331",
            "debugServerPath": "${env:SEGGER_TOOLS}/JLinkGDBServerCL.exe",
            "debugServerArgs": "-select USB=${env:PCA10040_SN} -device nRF52840_xxAA -if SWD -speed 1000 -noir",
            "serverStarted": "Connected to target",
        }
    ]
}
```

