
set(ddsCppSourcesDir "${CMAKE_CURRENT_SOURCE_DIR}")

set(CMAKE_CXX_STANDARD 17)

find_program(IDLC_EXECUTABLE NAMES idlc PATHS cyclonedds)

if (NOT IDLC_EXECUTABLE)
    message(FATAL_ERROR "The Cyclone DDS IDL compiler was not found. Cannot generate DDS sources.")
endif ()

get_filename_component(IDLC_PARENT_DIR "${IDLC_EXECUTABLE}" DIRECTORY)


set(ENV{LD_LIBRARY_PATH} "${VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/lib")

# generate cpp sources
execute_process(
        WORKING_DIRECTORY ${ddsCppSourcesDir}
        COMMAND "${IDLC_EXECUTABLE}"
        "-l" "cxx"
        "-o" "${ddsCppSourcesDir}"
        "${CMAKE_CURRENT_SOURCE_DIR}/HelloWorldData.idl"
        ERROR_VARIABLE idl_error
        RESULT_VARIABLE idl_result
)
if(${idl_result})
    message("IDL status ${idl_result}: ${idl_error}")
endif()

set(generatedSource "${ddsCppSourcesDir}/HelloWorldData.cpp")

add_executable(dds_publisher publisher.cpp "${generatedSource}")
target_link_libraries(dds_publisher PRIVATE CycloneDDS-CXX::ddscxx)
target_include_directories(dds_publisher PRIVATE ${ddsCppSourcesDir})

add_executable(dds_subscriber subscriber.cpp "${generatedSource}")
target_link_libraries(dds_subscriber PRIVATE CycloneDDS-CXX::ddscxx)
target_include_directories(dds_subscriber PRIVATE ${ddsCppSourcesDir})
