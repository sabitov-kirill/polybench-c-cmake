cmake_minimum_required(VERSION 3.5)
project(polybench)

add_compile_definitions(MEDIUM_DATASET)

add_library(polybench STATIC ${CMAKE_CURRENT_SOURCE_DIR}/utilities/polybench.c)
target_include_directories(polybench INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/utilities)

add_library(polybench_timed STATIC ${CMAKE_CURRENT_SOURCE_DIR}/utilities/polybench.c)
target_include_directories(polybench_timed INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/utilities)
target_compile_definitions(polybench_timed PRIVATE POLYBENCH_TIME)

# create empty list of polybench targets
set(polybench_targets "")

function(add_polybench_child_targets parent_folder)
    file(GLOB children RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/${parent_folder}/*)

    foreach (child ${children})
        if (IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${child})
            add_polybench_child_targets(${child})
        elseif (${child} MATCHES ".*\\.c$")
            get_filename_component(child_name ${child} NAME_WE)
            set(child_target_name polybench_${child_name})
            set(child_timed_target_name polybench_${child_name}_timed)

            add_executable(${child_target_name} ${child})
            target_link_libraries(${child_target_name} polybench m)
            add_push_and_build_for_target(${child_target_name})
            list(APPEND polybench_targets ${child_target_name})

            add_executable(${child_timed_target_name} ${child})
            target_link_libraries(${child_timed_target_name} polybench_timed m)
            target_compile_definitions(${child_timed_target_name} PRIVATE POLYBENCH_TIME)
            add_push_and_build_for_target(${child_timed_target_name})
        endif ()

        set(polybench_targets ${polybench_targets} PARENT_SCOPE)
    endforeach ()
endfunction()

add_polybench_child_targets(datamining)
add_polybench_child_targets(linear-algebra)
add_polybench_child_targets(medley)
add_polybench_child_targets(stencils)

add_custom_target(polybench_all DEPENDS ${polybench_targets})
