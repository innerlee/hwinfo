cmake_minimum_required(VERSION 3.16)
project(hwinfo VERSION 1.0.0 LANGUAGES CXX)

# set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")

if (NOT DEFINED NO_OCL)
    set(CMAKE_CXX_STANDARD 20)
else ()
    set(CMAKE_CXX_STANDARD 11)
endif ()
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(MAIN_PROJECT OFF)
if (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    set(MAIN_PROJECT ON)
endif()


if (NOT DEFINED NO_OCL)
    set(OPENCL_SDK_BUILD_SAMPLES OFF CACHE BOOL "" FORCE)
    include_directories(external/miss-opencl/include)
    include_directories(external/miss-opencl/external/OpenCL/external/OpenCL-CLHPP/include)
    if (NOT TARGET miss-opencl_static)
        add_subdirectory(external/miss-opencl)
    endif ()

    add_compile_definitions(USE_OCL)
    add_compile_definitions(NOMINMAX)
endif ()

include_directories({PROJECT_SOURCE_DIR}/include)
add_subdirectory(src)
target_include_directories(HWinfo PUBLIC ${PROJECT_SOURCE_DIR}/include/)

if (CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
    # OLD STORAGE LOCATION (DEPRECATED)
    #file(COPY "scripts/pci.ids" DESTINATION "$ENV{HOME}/.hwinfo/")

    set(PCI_IDS_SOURCE "scripts/pci.ids")
    set(PCI_IDS_DESTINATION "${CMAKE_BINARY_DIR}/.hwinfo")

    configure_file("${PCI_IDS_SOURCE}" "${PCI_IDS_DESTINATION}/pci.ids" COPYONLY)

    install(FILES "${PCI_IDS_SOURCE}" DESTINATION ${CMAKE_BINARY_DIR}/bin)
endif ()

if (${MAIN_PROJECT})
    add_subdirectory(examples)

    include(CTest)
    add_subdirectory(test)

    add_test(Example examples/Example)
endif()
