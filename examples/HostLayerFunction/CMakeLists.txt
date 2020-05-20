project(HostLayerFunction LANGUAGES C CXX CUDA)
include(CheckLanguage)
check_language(CUDA)

# Toggle build functions template when functions.c doesn't exist
set(build_functions 1)

# Toggle build visualisation
set(build_visualisation 0)

################################
# PRE-BUILD: TEMPLATE GENERATION
################################

# Listing all dynamic output files
set(dynamic_header_file "${CMAKE_CURRENT_SOURCE_DIR}/src/dynamic/header.h")
set(dynamic_main_file "${CMAKE_CURRENT_SOURCE_DIR}/src/dynamic/main.cu")
set(dynamic_io_file "${CMAKE_CURRENT_SOURCE_DIR}/src/dynamic/io.cu")
set(dynamic_functions_file "${CMAKE_CURRENT_SOURCE_DIR}/src/model/functions.c")
set(dynamic_fgpukernels_file "${CMAKE_CURRENT_SOURCE_DIR}/src/dynamic/FLAMEGPU_kernals.cu")
set(dynamic_simulation_file "${CMAKE_CURRENT_SOURCE_DIR}/src/dynamic/simulation.cu")
set(dynamic_visualisation_file "${CMAKE_CURRENT_SOURCE_DIR}/src/dynamic/visualisation.cu")


set(TEMPLATES_DIR "${CMAKE_SOURCE_DIR}/FLAMEGPU/templates")
set(MODEL_FILE "${CMAKE_CURRENT_SOURCE_DIR}/src/model/XMLModelFile.xml")



# Generate header file command
add_custom_command(OUTPUT ${dynamic_header_file}
    COMMAND xsltproc -o ${dynamic_header_file} "${TEMPLATES_DIR}/header.xslt" ${MODEL_FILE}
    MAIN_DEPENDENCY ${MODEL_FILE}
    COMMENT "Generating ${dynamic_header_file}"
    )


# Add functions.c if doesn't already exist
set(kernel_deps  ${dynamic_header_file})
if(${build_functions})
    if(EXISTS ${dynamic_functions_file})
        message(STATUS "Skip generating functions.c template as it already exists")
    else()
        add_custom_command(OUTPUT ${dynamic_functions_file}
                COMMAND xsltproc -o ${dynamic_functions_file} "${TEMPLATES_DIR}/functions.xslt" ${MODEL_FILE}
                COMMENT "Generating ${dynamic_functions_file} template"
            )
        list(APPEND kernel_deps ${dynamic_functions_file})
    endif()
endif()

# Generate fgpukernels file command
add_custom_command(OUTPUT ${dynamic_fgpukernels_file}
    COMMAND xsltproc -o ${dynamic_fgpukernels_file} "${TEMPLATES_DIR}/FLAMEGPU_kernals.xslt" ${MODEL_FILE}
    MAIN_DEPENDENCY ${MODEL_FILE}
    DEPENDS ${kernel_deps}
    COMMENT "Generating ${dynamic_fgpukernels_file}"
    )

# Generate simulation file command
add_custom_command(OUTPUT ${dynamic_simulation_file}
    COMMAND xsltproc -o ${dynamic_simulation_file} "${TEMPLATES_DIR}/simulation.xslt" ${MODEL_FILE}
    MAIN_DEPENDENCY ${MODEL_FILE}
    DEPENDS ${dynamic_header_file} ${dynamic_fgpukernels_file}
    COMMENT "Generating ${dynamic_simulation_file}"
    )

# Generate IO file command
add_custom_command(OUTPUT ${dynamic_io_file}
    COMMAND xsltproc -o ${dynamic_io_file} "${TEMPLATES_DIR}/io.xslt" ${MODEL_FILE}
    MAIN_DEPENDENCY ${MODEL_FILE}
    DEPENDS ${dynamic_header_file}
    COMMENT "Generating ${dynamic_io_file}"
    )

# Generate main file command
add_custom_command(OUTPUT ${dynamic_main_file}
    COMMAND xsltproc -o ${dynamic_main_file} "${TEMPLATES_DIR}/main.xslt" ${MODEL_FILE}
    MAIN_DEPENDENCY ${MODEL_FILE}
    DEPENDS ${dynamic_header_file} ${dynamic_simulation_file}
    COMMENT "Generating ${dynamic_main_file}"
    )

set (build_src 
    ${dynamic_main_file} 
    ${dynamic_io_file} 
    ${dynamic_simulation_file}
    )

# Add visualisation file for compilation
if(${build_visualisation})
    message(STATUS "Will build visualisation")
    list(APPEND build_src ${dynamic_visualisation_file})
endif()

##############################
# BUILD: GENERATE EXECUTABLE
##############################
add_executable(${PROJECT_NAME} ${build_src})
set_target_properties(${PROJECT_NAME} PROPERTIES
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED YES
    CUDA_STANDARD 11
    CUDA_STANDARD_REQUIRED ON
)

target_include_directories(${PROJECT_NAME} PUBLIC ../../include src/model src/dynamic src/visualisation)

