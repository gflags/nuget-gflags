Packaging gflags for Nuget
--------------------------

### Requirements
To build the nuget package you need:

* CoApp tools: http://downloads.coapp.org/files/Development.CoApp.Tools.Powershell.msi 
* Visual Studio 2010, 2012 and 2013


### Creating the nuget package
The CMakeLists.txt file defines a project via `ExternalProject_Add` for each generator defined in the
CMAKE_GENERATORS cache variable. By default Visual Studio 2010, 2012 and 2013 for both 32 bit and 64 bit are
used. To create the package:
```    
    cmd> git clone https://github.com/gflags/gflags.git gflags
    cmd> git clone https://github.com/gflags/nuget-gflags.git nuget-gflags
    cmd> mkdir nuget-gflags-build
    cmd> cd nuget-gflags-build    
    cmd> cmake -G "Visual Studio 12 2013 Win64" ..\nuget-gflags
    cmd> cmake --build --config Debug
    cmd> cmake --build --config Release
    cmd> powershell
    PS> Write-NuGetPackage -SplitThreshold 10000000 .\gflags.autopkg
```        

### Using the nuget package with a CMake based project on Windows
In powershell execute:
```PowerShell
PS> nuget install gflags -ExcludeVersion
```
    
Then in your CMakeLists.txt:
```CMake
cmake_minimum_required(VERSION 2.8.12)

project(test_gflags)

# make sure CMake finds the nuget installed package
find_package(gflags REQUIRED)

add_executable(test_gflags main.cpp)

# gflags libraries are automatically mapped to the good arch/VS version/linkage combination
target_link_libraries(test_gflags ${gflags_LIBRARIES})
target_include_directories(test_gflags PRIVATE ${gflags_INCLUDE_DIR})

# copy the DLL to the output folder if desired.
if (MSVC AND COMMAND target_copy_shared_libs AND NOT gflags_STATIC)
  target_copy_shared_libs(test_gflags ${gflags_LIBRARIES})
endif ()
```    