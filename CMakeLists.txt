cmake_minimum_required(VERSION 3.8)
project(tbb CXX)

if(NOT TBB_LINKAGE)
  set(TBB_LINKAGE SHARED)
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
configure_file(cmake/version_string.ver.in version_string.ver)

add_library(tbb ${TBB_LINKAGE}
    src/tbb/concurrent_hash_map.cpp
    src/tbb/concurrent_queue.cpp
    src/tbb/concurrent_vector.cpp
    src/tbb/dynamic_link.cpp
    src/tbb/itt_notify.cpp
    src/tbb/cache_aligned_allocator.cpp
    src/tbb/pipeline.cpp
    src/tbb/queuing_mutex.cpp
    src/tbb/queuing_rw_mutex.cpp
    src/tbb/reader_writer_lock.cpp
    src/tbb/spin_rw_mutex.cpp
    src/tbb/x86_rtm_rw_mutex.cpp
    src/tbb/spin_mutex.cpp
    src/tbb/critical_section.cpp
    src/tbb/mutex.cpp
    src/tbb/recursive_mutex.cpp
    src/tbb/condition_variable.cpp
    src/tbb/tbb_thread.cpp
    src/tbb/concurrent_monitor.cpp
    src/tbb/semaphore.cpp
    src/tbb/private_server.cpp
    src/tbb/tbb_misc.cpp
    src/tbb/tbb_misc_ex.cpp
    src/tbb/task.cpp
    src/tbb/task_group_context.cpp
    src/tbb/governor.cpp
    src/tbb/market.cpp
    src/tbb/arena.cpp
    src/tbb/scheduler.cpp
    src/tbb/observer_proxy.cpp
    src/tbb/tbb_statistics.cpp
    src/tbb/tbb_main.cpp

    src/rml/client/rml_tbb.cpp
)

target_include_directories(tbb
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>)

target_compile_features(tbb PUBLIC cxx_std_17)
target_compile_definitions(tbb
  PRIVATE
    __TBB_BUILD=1
)

if(UNIX)
  target_compile_definitions(tbb PRIVATE USE_PTHREAD)
else()
  target_compile_definitions(tbb PRIVATE USE_WINTHREAD)
endif()

if(UNIX AND NOT APPLE)
  target_compile_definitions(tbb PRIVATE DO_ITT_NOTIFY)
  target_link_libraries(tbb PUBLIC ${CMAKE_DL_LIBS})
endif()

if(WIN32)
  target_compile_definitions(tbb PUBLIC __TBB_NO_IMPLICIT_LINKAGE=1)
endif()

include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-flifetime-dse=1" LIFETIME_DSE)
if(LIFETIME_DSE)
  target_compile_options(tbb PRIVATE -flifetime-dse=1)
endif()

check_cxx_compiler_flag("-mrtm" RTM)
if(RTM)
  target_compile_options(tbb PRIVATE -mrtm)
endif()

install(
  TARGETS tbb
  EXPORT tbb-exports
  ARCHIVE DESTINATION lib/static
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib)

install(
  DIRECTORY include/tbb
  DESTINATION include)

install(
  EXPORT tbb-exports
  DESTINATION lib/cmake/tbb)

export(
  EXPORT tbb-exports
  NAMESPACE tbb)
