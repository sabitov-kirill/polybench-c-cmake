cmake_minimum_required(VERSION 3.5)
project(polybench)

set(POLYBENCH_DATASET_SIZE "MEDIUM" CACHE STRING "Polybench dataset size (MINI, SMALL, MEDIUM, LARGE, EXTRALARGE)")
set_property(CACHE POLYBENCH_DATASET_SIZE PROPERTY STRINGS MINI SMALL MEDIUM LARGE EXTRALARGE)
string(TOLOWER ${POLYBENCH_DATASET_SIZE} POLYBENCH_DATASET_SIZE_LOWER)

add_compile_definitions(${POLYBENCH_DATASET_SIZE}_DATASET)

add_library(polybench STATIC utilities/polybench.c)
target_include_directories(polybench INTERFACE utilities)

add_library(polybench_timed STATIC utilities/polybench.c)
target_include_directories(polybench_timed INTERFACE utilities)
target_compile_definitions(polybench_timed PRIVATE POLYBENCH_TIME)

function(add_polybench_child_targets parent_folder)
    file(GLOB children RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/${parent_folder}/*)

    foreach (child ${children})
        if (IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${child})
            add_polybench_child_targets(${child})
        elseif (${child} MATCHES ".*\\.c$")
            get_filename_component(child_name ${child} NAME_WE)
            set(target polybench_${child_name}_${POLYBENCH_DATASET_SIZE_LOWER})

            add_executable(${target} ${child})
            target_link_libraries(${target} polybench m)
            set_property(GLOBAL APPEND PROPERTY polybench_targets ${target})

            add_executable(${target}_timed ${child})
            target_link_libraries(${target}_timed polybench_timed m)
            target_compile_definitions(${target}_timed PRIVATE POLYBENCH_TIME)
            set_property(GLOBAL APPEND PROPERTY polybench_targets ${target}_timed)
        endif ()
    endforeach ()
endfunction()

add_polybench_child_targets(datamining)
add_polybench_child_targets(linear-algebra)
add_polybench_child_targets(medley)
add_polybench_child_targets(stencils)

get_property(polybench_targets GLOBAL PROPERTY polybench_targets)
add_custom_target(polybench_all DEPENDS ${polybench_targets})
