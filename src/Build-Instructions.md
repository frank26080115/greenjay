Operating system being used is `Windows 11 Pro 64-bit`

IDE and toolchain is `Simplicity Studio V5` from `Silicon Labs`: https://www.silabs.com/developers/simplicity-studio

Obtain and activate the license for `Keil PK51 Developer's Kit`: https://www.silabs.com/developers/keil-pk51

Example installation path is `SIMPLICITY_PATH ?= C:/SiliconLabs/SimplicityStudio/v5` and it is specified in the makefile

Keil path is specified as `$(SIMPLICITY_PATH)/developer/toolchains/keil_8051/9.60/BIN` check that the version number is correct

Assembler binary is specified as `AX51_BIN = $(KEIL_PATH)/AX51.exe`

`make` executable is found at: `C:\SiliconLabs\SimplicityStudio\v5\support\common\build\msys\1.0\bin\make.exe` make sure the system environment variables has this included in the list of paths. Version used is `GNU Make 4.3 Built for x86_64-pc-msys`

Modify `VARIANT`, `MCU`, and `FETON_DELAY` for the target. Obtain target information by reading it using BLHeliSuite https://github.com/bitdump/BLHeli/releases

Use an ordinary Windows command prompt to execute `make` on the makefile, build artifacts are in `src\build`, use BLheliSuite to bootload it into the ESC
