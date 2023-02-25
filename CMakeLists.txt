cmake_minimum_required(VERSION 3.16)

# Set some basic project attributes
project (RPI_X_Compile
	VERSION 0.1
	DESCRIPTION "RPI_X_Compile")

add_subdirectory(	
	${PROJECT_SOURCE_DIR}/odas
)

set (SRC src)
# This project will output an executable file
add_executable(${PROJECT_NAME} ${SRC}/main.cpp
	odas/demo/odaslive/configs.c
    odas/demo/odaslive/objects.c
    odas/demo/odaslive/parameters.c
    odas/demo/odaslive/profiler.c
    odas/demo/odaslive/threads.c
    odas
)

target_include_directories(${PROJECT_NAME} PRIVATE .
	${PROJECT_NAME} PRIVATE ./include
	${PROJECT_NAME} PRIVATE ./odas/include
	${PROJECT_NAME} PRIVATE ./odas/include/odas	
	${PROJECT_NAME} PRIVATE ./odas/demo/odaslive
)

target_link_libraries(${PROJECT_NAME}
    odas
)

SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}")
SET(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} ${GCC_COVERAGE_LINK_FLAGS}")