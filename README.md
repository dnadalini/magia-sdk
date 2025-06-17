# Magia-sdk
This repository contains the WIP development platform for the mesh-based architecture [MAGIA](https://github.com/pulp-platform/MAGIA/tree/main).
It provides useful tools for programming and running software applications on simulated MAGIA architectures for benchmarking, debugging and profiling.

Magia and Magia-SDK are developed as part of the [PULP project](https://pulp-platform.org/index.html), a joint effort between ETH Zurich and the University of Bologna.

## Getting started and Usage

The following *optional* parameters can be specified when running the make command:

`target_platform`: **magia** (**Default**: magia). Selects the target platform to build and run tests on. At the moment, only MAGIA is supported by the SDK.

`build_mode`: **update**|**profile**|**synth** (**Default**: update). Selects the mode that the MAGIA architecture is built.

`fsync_mode`: **stall**|**interrupt** (**Default** stall). Selects the Fractal Sync module synchronization behaviour.

`compiler`: **GCC**|**LLVM** (**Default**: GCC). Selects the compiler to be used. LLVM is currently WIP.

`platform`: **rtl**|**gvsoc** (**Default**: rtl). Selects the simulation platform. GVSoC is currently WIP.

`tiles`: **2**|**4**|**8**|**16** (**Default**: 2). Selects number of rows and columns for the mesh architecture.

`test_name`: Name of the test binary to be run.

0. In case you are using this SDK as non-submodule: Clone the [MAGIA](https://github.com/pulp-platform/MAGIA/tree/main) repository:

    `git clone git@github.com:pulp-platform/MAGIA.git`

    Then modify the magia-sdk **Makefile** to point to the correct paths:

        MAGIA_DIR ?= path/to/MAGIA/repository

        BUILD_DIR ?= $(MAGIA_DIR)//work/sw/tests/$(test).c

2. Build the Magia architecture (*this command may take time and return an error, please be patient.*):
        
    `make MAGIA <target_platform> <tiles> <build_mode> <fsync_mode>`

3. Make sure the RISC-V GCC compiler is installed and visible in the `$PATH` environment variable. You can check if and where the compiler is installed by running the following command on your root (`/`) directory:

    `find . ! -readable -prune -o -name "riscv32-unknown-elf-gcc" -print`

    Then add the compiler to the `$PATH` environment variable with:

    `export PATH=<absolute path to directory containing the compiler binary>:$PATH`

4. To compile and build the test binaries for a desired architecture run:

    `make clean build <target_platform> <tiles> <compiler>`

    To run one of the tests:

    `make run test=<test_name> <platform>`

## Adding your own test

This SDK uses a nested CMakeList mechanism to build and check for dependencies.
To add your own test, you have to integrate a new test folder inside the **tests** directory.

1. Change directory to the desired architecture test folder

    `cd tests/<target_platform>/mesh`

2. Create a new test directory

    `mkdir <test_name>`

3. Modify the *CMakeList.txt* file in the current mesh directory, adding the following line:

    `add_subdirectory(<test_name>)`

    You can also *exclude* the generation of other tests binaries by commenting/deleting the lines for those tests.

4. Add to the *\<test_name\>* directory:

    1. A new CMakeList.txt file following this template:
    
            set(TEST_NAME <test_name>)

            file(GLOB_RECURSE TEST_SRCS
            "src/*.c"
            )

            add_executable(${TEST_NAME} ${TEST_SRCS})
            target_include_directories(${TEST_NAME} PUBLIC include)

            target_compile_options(${TEST_NAME}
            PRIVATE
            -O2
            )
            target_link_libraries(${TEST_NAME} PUBLIC runtime hal)

            add_custom_command(
                    TARGET ${TEST_NAME}
                    POST_BUILD
                    COMMAND ${CMAKE_OBJDUMP} -dhS $<TARGET_FILE:${TEST_NAME}> > $<TARGET_FILE:${TEST_NAME}>.s)
    
    2. An **src** directory containing your test's source (.c) files

    3. An **include** directory containing your test's header (.h) files

## Folder Structure

### README.md
This file.

### Makefile
Makefile script to run the sdk.

### CMakeLists.txt
Root CMakeLists file to compile and build the executable test/application binaries for one of the available architectures.

### targets
This directory contains the *startup routine*, *linker script*, *address map*, *register definitions*, *custom ISA instructions*, *MAGIA mesh and tile util instructions* for each available architecture.

### scripts
Contains scripts to automatize the test building and running.

### hal
Contains the weak definitions of this SDK APIs. These are the API instruction that should be used by the programmer when developing applications to be run on MAGIA. These instructions are then overloaded by the corresponding driver implementation specific for the chosen architecture. The APIs currently available are for controlling and using the *idma*, *redmule* and *fractalsync* modules.

### drivers
Contains the architecture-specific implementation and source code for the HAL APIs. Despite each implementation having different names, thanks to an aliasing system the programmer can use the same name for the same API instruction on different architectures.

### devices
Nothing there. 

If MAGIA ever evolves to have a host-offload mechanism, this folder will contain the trampoline functions.

### cmake
Contains utility files for *cmake* automatic compilation.

