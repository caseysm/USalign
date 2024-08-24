cmake_minimum_required(VERSION 3.10)
project(TMalign_suite)

# Check for the highest C++ standard supported by the compiler
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    set(CMAKE_CXX_STANDARD 17)
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 8.0)
        set(CMAKE_CXX_STANDARD 14)
    endif()
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 5.0)
        set(CMAKE_CXX_STANDARD 11)
    endif()
else()
    set(CMAKE_CXX_STANDARD 11)  # Default to C++11 for other compilers
endif()

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

message(STATUS "Using C++ standard: C++${CMAKE_CXX_STANDARD}")

# Set compiler flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -ffast-math")

# Define the list of all possible executables
set(ALL_EXECUTABLES
        qTMclust
        USalign
        TMalign
        TMscore
        MMalign
        se
        pdb2xyz
        xyz_sfetch
        pdb2fasta
        pdb2ss
        NWalign
        HwRMSD
        cif2pdb
        pdbAtomName
        addChainID
)

# Option to build all executables (off by default)
option(BUILD_ALL_EXECUTABLES "Build all executables" OFF)

# Set the executables to build
if(BUILD_ALL_EXECUTABLES)
    set(EXECUTABLES ${ALL_EXECUTABLES})
else()
    set(EXECUTABLES USalign)
endif()

# Add executable targets
foreach(EXECUTABLE ${EXECUTABLES})
    add_executable(${EXECUTABLE} ${EXECUTABLE}.cpp)
endforeach()

# Special case for USalign.exe (for MinGW)
if(MINGW)
    add_executable(USalign.exe USalign.cpp)
    set_target_properties(USalign.exe PROPERTIES LINK_FLAGS "-static")
endif()

# Add include directories
include_directories(${CMAKE_SOURCE_DIR})

# Set specific dependencies for each executable
if(TARGET USalign)
    target_sources(USalign PRIVATE SOIalign.h MMalign.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h se.h NWalign.h BLOSUM.h flexalign.h)
endif()

if(BUILD_ALL_EXECUTABLES)
    target_sources(qTMclust PRIVATE HwRMSD.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h NWalign.h BLOSUM.h)
    target_sources(TMalign PRIVATE param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h NWalign.h BLOSUM.h)
    target_sources(TMscore PRIVATE TMscore.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h NWalign.h BLOSUM.h)
    target_sources(MMalign PRIVATE MMalign.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h NWalign.h BLOSUM.h)
    target_sources(se PRIVATE se.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h NWalign.h BLOSUM.h)
    target_sources(pdb2ss PRIVATE se.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h)
    target_sources(pdb2xyz PRIVATE basic_fun.h pstream.h)
    target_sources(pdb2fasta PRIVATE basic_fun.h pstream.h)
    target_sources(NWalign PRIVATE NWalign.h basic_fun.h pstream.h BLOSUM.h)
    target_sources(HwRMSD PRIVATE HwRMSD.h NWalign.h BLOSUM.h se.h param_set.h basic_fun.h Kabsch.h NW.h TMalign.h pstream.h se.h)
    target_sources(cif2pdb PRIVATE pstream.h)
    target_sources(pdbAtomName PRIVATE pstream.h)
    target_sources(addChainID PRIVATE pstream.h)
endif()

# Optional: Set output directory for all targets
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Add install rules
install(TARGETS ${EXECUTABLES}
        RUNTIME DESTINATION bin
)

# If using MinGW, also install USalign.exe
if(MINGW)
    install(TARGETS USalign.exe
            RUNTIME DESTINATION bin
    )
endif()