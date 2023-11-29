cmake_minimum_required(VERSION 3.10)

project(my-sample-lib)

include(GNUInstallDirs)

# Find the fmt library
find_package(fmt CONFIG REQUIRED)

# Add your library
add_library(my-sample-lib my_sample_lib.cpp)

if (BUILD_SHARED_LIBS AND MSVC)
    target_compile_definitions(my-sample-lib PRIVATE MYLIB_EXPORTS)
endif()

# Link your library to fmt
target_link_libraries(my-sample-lib PRIVATE fmt::fmt)


# Add include directories
target_include_directories(my-sample-lib PUBLIC 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>   # for headers when building
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>  # for client in install mode
)
# Install the library and its headers
install(TARGETS my-sample-lib
        EXPORT my-sample-lib_targets
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")

install(FILES my_sample_lib.h DESTINATION include)


# Generate and install *-targets.cmake 
install(EXPORT my-sample-lib_targets
        FILE my-sample-lib-targets.cmake
        NAMESPACE my-sample-lib::
        DESTINATION share/my-sample-lib)

# Generate the config file in the current binary dir (this ensures it's not placed directly in source)
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/my-sample-lib-config.cmake"
"include(CMakeFindDependencyMacro)\n"
"find_dependency(fmt CONFIG REQUIRED)\n"
"include(\"\${CMAKE_CURRENT_LIST_DIR}/my-sample-lib-targets.cmake\")\n"
)

# Install the generated config file
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/my-sample-lib-config.cmake"
        DESTINATION share/my-sample-lib)
