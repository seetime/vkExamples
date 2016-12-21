cmake_minimum_required(VERSION 2.8)

# set project name from current directory
get_filename_component(TARGET ${CMAKE_CURRENT_SOURCE_DIR} NAME) 
project( ${TARGET} )

SET(${TARGET}_HEADERS
)

SET(${TARGET}_SOURCES 
  main.cpp
)

file(GLOB KERNEL_SRC "kernels/*" )
#set(KERNEL_SRC
#	kernels/simple.vert
#	kernels/simple.frag
#)
foreach(KERNEL ${KERNEL_SRC})
	get_filename_component(barename "${KERNEL}" NAME)
	list(APPEND KERNEL_SRC_COMPILED "${PROJECT_OUTPUT_DIR}/${barename}.spv")
endforeach()

#set(KERNEL_SRC_COMPILED "${KERNEL_SRC}.spv")
#set(COMPILED_KERNELS
#	${PROJECT_OUTPUT_DIR}/mesh.vert.spv
#	${PROJECT_OUTPUT_DIR}/mesh.frag.spv
#)

SOURCE_GROUP(shaders REGULAR_EXPRESSION ".*/kernels/.*" )

INCLUDE_DIRECTORIES(
  ../common/
    ${Vulkan_INCLUDE_DIR}
)
link_directories(${GLUT_LIBRARY_DIR})



MESSAGE ("-- KERNEL_SRC: ${KERNEL_SRC}")
MESSAGE ("-- KERNEL_SRC_COMPILED: ${KERNEL_SRC_COMPILED}")

add_executable( ${TARGET} WIN32 ${${TARGET}_SOURCES} ${${TARGET}_HEADERS} ${KERNEL_SRC} ${KERNEL_SRC_COMPILED})
target_link_libraries( ${TARGET} ${ALL_LIBS}  ${Vulkan_LIBRARY} )

foreach(KERNEL ${KERNEL_SRC})
	get_filename_component(barename "${KERNEL}" NAME)
	set(SPVFILE ${PROJECT_OUTPUT_DIR}/${barename}.spv)
	add_custom_command(OUTPUT ${SPVFILE}
		COMMAND ${PROJECT_OUTPUT_DIR}/glslc ${KERNEL} -g -o ${SPVFILE}
		DEPENDS ${KERNEL}
		COMMENT "Rebuilding ${SPVFILE}" )
		message(STATUS "Generating build commands for ${SPVFILE}")
endforeach()

#create_target_launcher(${TARGET} ARGS "-w 6 -h 9 -s 0.025 -o camera.yml -op -oe" WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}/../../bin/")
create_target_launcher(${TARGET} ARGS "" WORKING_DIRECTORY "${PROJECT_OUTPUT_DIR}" ENVIRONMENT "${RUN_ENV}")

add_custom_command(
   TARGET ${TARGET} POST_BUILD
   #COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/${TARGET}${CMAKE_EXECUTABLE_SUFFIX}" "${PROJECT_SOURCE_DIR}/../colortest"
   COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/${TARGET}${CMAKE_EXECUTABLE_SUFFIX}" "${PROJECT_OUTPUT_DIR}"
   WORKING_DIRECTORY ${PROJECT_OUTPUT_DIR}
  )