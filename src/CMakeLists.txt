# Copyright (C) 2015-2025 Jonathan Müller and foonathan/memory contributors
# SPDX-License-Identifier: Zlib

# builds actual library

set(header_path ${FOONATHAN_MEMORY_SOURCE_DIR}/include/foonathan/memory)
set(detail_header
        ${header_path}/detail/align.hpp
        ${header_path}/detail/assert.hpp
        ${header_path}/detail/container_node_sizes.hpp
        ${header_path}/detail/debug_helpers.hpp
        ${header_path}/detail/ebo_storage.hpp
        ${header_path}/detail/free_list.hpp
        ${header_path}/detail/free_list_array.hpp
        ${header_path}/detail/ilog2.hpp
        ${header_path}/detail/lowlevel_allocator.hpp
        ${header_path}/detail/memory_stack.hpp
        ${header_path}/detail/small_free_list.hpp
        ${header_path}/detail/utility.hpp)
set(header
        ${header_path}/aligned_allocator.hpp
        ${header_path}/allocator_storage.hpp
        ${header_path}/allocator_traits.hpp
        ${header_path}/config.hpp
        ${header_path}/container.hpp
        ${header_path}/debugging.hpp
        ${header_path}/default_allocator.hpp
        ${header_path}/deleter.hpp
        ${header_path}/error.hpp
        ${header_path}/fallback_allocator.hpp
        ${header_path}/malloc_allocator.hpp
        ${header_path}/heap_allocator.hpp
        ${header_path}/iteration_allocator.hpp
        ${header_path}/joint_allocator.hpp
        ${header_path}/memory_arena.hpp
        ${header_path}/memory_pool.hpp
        ${header_path}/memory_pool_collection.hpp
        ${header_path}/memory_pool_type.hpp
        ${header_path}/memory_resource_adapter.hpp
        ${header_path}/memory_stack.hpp
        ${header_path}/namespace_alias.hpp
        ${header_path}/new_allocator.hpp
        ${header_path}/segregator.hpp
        ${header_path}/smart_ptr.hpp
        ${header_path}/static_allocator.hpp
        ${header_path}/std_allocator.hpp
        ${header_path}/temporary_allocator.hpp
        ${header_path}/threading.hpp
        ${header_path}/tracking.hpp
        ${header_path}/virtual_memory.hpp
        ${CMAKE_CURRENT_BINARY_DIR}/container_node_sizes_impl.hpp)

set(src
        detail/align.cpp
        detail/debug_helpers.cpp
        detail/assert.cpp
        detail/free_list.cpp
        detail/free_list_array.cpp
        detail/free_list_utils.hpp
        detail/small_free_list.cpp
        debugging.cpp
        error.cpp
        heap_allocator.cpp
        iteration_allocator.cpp
        malloc_allocator.cpp
        memory_arena.cpp
        memory_pool.cpp
        memory_pool_collection.cpp
        memory_stack.cpp
        new_allocator.cpp
        static_allocator.cpp
        temporary_allocator.cpp
        virtual_memory.cpp)

# configure config file
configure_file("config.hpp.in" "${CMAKE_CURRENT_BINARY_DIR}/config_impl.hpp")

# generate container_node_sizes.hpp if necessary
if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/container_node_sizes_impl.hpp)
    get_container_node_sizes(${CMAKE_CURRENT_BINARY_DIR}/container_node_sizes_impl.hpp)
endif()

add_library(foonathan_memory ${detail_header} ${header} ${src})
target_include_directories(foonathan_memory PUBLIC $<BUILD_INTERFACE:${FOONATHAN_MEMORY_SOURCE_DIR}/include/> # for client in subdirectory
                                                   $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}> # for generated files in build mode
                                                   $<INSTALL_INTERFACE:${FOONATHAN_MEMORY_INC_INSTALL_DIR}> # for client in install mode
                                            PRIVATE ${header_path}) # for source files
target_compile_definitions(foonathan_memory PUBLIC
                            FOONATHAN_MEMORY=1
                            FOONATHAN_MEMORY_VERSION_MAJOR=$CACHE{FOONATHAN_MEMORY_VERSION_MAJOR}
                            FOONATHAN_MEMORY_VERSION_MINOR=$CACHE{FOONATHAN_MEMORY_VERSION_MINOR}
                            FOONATHAN_MEMORY_VERSION_PATCH=$CACHE{FOONATHAN_MEMORY_VERSION_PATCH})
if(NOT MSVC)
target_compile_features(foonathan_memory PUBLIC cxx_constexpr)
endif()

if(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
    if("${CMAKE_CXX_SIMULATE_ID}" STREQUAL "MSVC")
        target_compile_options(foonathan_memory PRIVATE /WX /W3 /D _CRT_SECURE_NO_WARNINGS)
    else()
        target_compile_options(foonathan_memory PRIVATE -pedantic-errors -Werror -Wall -Wextra -Wconversion -Wsign-conversion)
    endif()
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
    target_compile_options(foonathan_memory PRIVATE -pedantic-errors -Werror -Wall -Wextra -Wconversion -Wsign-conversion)
elseif(MSVC)
    target_compile_options(foonathan_memory PRIVATE /WX /W3 /D _CRT_SECURE_NO_WARNINGS)
endif()

set_target_properties(foonathan_memory PROPERTIES
                                       OUTPUT_NAME "foonathan_memory-$CACHE{FOONATHAN_MEMORY_VERSION}"
                                       POSITION_INDEPENDENT_CODE ON)

if(NEED_LIBRARY_FOR_CXX_ATOMIC)
    target_link_libraries(foonathan_memory PRIVATE -latomic)
endif()

install(TARGETS foonathan_memory EXPORT foonathan_memoryTargets
        RUNTIME       DESTINATION ${FOONATHAN_MEMORY_RUNTIME_INSTALL_DIR}
        LIBRARY       DESTINATION ${FOONATHAN_MEMORY_LIBRARY_INSTALL_DIR}
        ARCHIVE       DESTINATION ${FOONATHAN_MEMORY_ARCHIVE_INSTALL_DIR}
        FRAMEWORK     DESTINATION ${FOONATHAN_MEMORY_FRAMEWORK_INSTALL_DIR})

# Write/install version file
include(CMakePackageConfigHelpers)
set(version_file "${CMAKE_CURRENT_BINARY_DIR}/cmake/foonathan_memory-config-version.cmake")
write_basic_package_version_file(${version_file}
                                 VERSION $CACHE{FOONATHAN_MEMORY_VERSION}
                                 COMPATIBILITY AnyNewerVersion)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/config_impl.hpp DESTINATION ${FOONATHAN_MEMORY_INC_INSTALL_DIR}/foonathan/memory/)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/container_node_sizes_impl.hpp DESTINATION ${FOONATHAN_MEMORY_INC_INSTALL_DIR}/foonathan/memory/detail)
install(FILES ${header}                                   DESTINATION ${FOONATHAN_MEMORY_INC_INSTALL_DIR}/foonathan/memory)
install(FILES ${detail_header}                            DESTINATION ${FOONATHAN_MEMORY_INC_INSTALL_DIR}/foonathan/memory/detail)
install(FILES ${version_file}                             DESTINATION ${FOONATHAN_MEMORY_CMAKE_CONFIG_INSTALL_DIR})
