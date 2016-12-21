cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
cmake_policy(VERSION 2.8)




set(NAME vkExamples)

project(${NAME})

#set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake ${CMAKE_SOURCE_DIR}/cmake/rpavlik-cmake-modules-fe2273")
SET(CMAKE_MODULE_PATH 
	${PROJECT_SOURCE_DIR}/cmake/
	${PROJECT_SOURCE_DIR}/cmake/rpavlik-cmake-modules-fe2273 
	${CMAKE_MODULE_PATH}
)

include(CreateLaunchers)
include(MSVCMultipleProcessCompile) # /MP


if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
	set ( MACHINEBIT	64)
else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
	set ( MACHINEBIT	32)
endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )

if(MSVC)
# Instruct CMake where to find cmake related stuffs
SET( BINDIR binvc-${MACHINEBIT})
ELSE(MSVC)
SET( BINDIR bin-MACHINEBIT)
endif(MSVC)

set(PROJECT_OUTPUT_DIR ${PROJECT_SOURCE_DIR}/../${BINDIR})
set (RUN_ENV "PATH=${PROJECT_OUTPUT_DIR}") 
MESSAGE ("-- PROJECT_OUTPUT_DIR: ${PROJECT_OUTPUT_DIR}") 

include_directories(external)
include_directories(external/glm)
include_directories(external/gli)
include_directories(external/assimp)
include_directories(base)

OPTION(USE_D2D_WSI "Build the project using Direct to Display swapchain" OFF)



# Use FindVulkan module added with CMAKE 3.7
if (NOT CMAKE_VERSION VERSION_LESS 3.7.0)
	message(STATUS "Using module to find Vulkan")
	#find_package(Vulkan REQUIRED)
	find_package(Vulkan )
endif()

IF(WIN32)
	IF (NOT Vulkan_FOUND)
		set(Vulkan_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/external/vulkan)
		find_library(Vulkan_LIBRARY NAMES vulkan-1 vulkan PATHS ${CMAKE_SOURCE_DIR}/libs/vulkan)
		IF (Vulkan_LIBRARY)
			set(Vulkan_FOUND ON)
		ENDIF()
	ENDIF()
	find_library(ASSIMP_LIBRARIES NAMES assimp libassimp.dll.a PATHS ${CMAKE_SOURCE_DIR}/libs/assimp)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DVK_USE_PLATFORM_WIN32_KHR")
ELSE(WIN32)
	IF (NOT Vulkan_FOUND)
		find_library(Vulkan_LIBRARY NAMES vulkan HINTS "$ENV{VULKAN_SDK}/lib" "${CMAKE_SOURCE_DIR}/libs/vulkan" REQUIRED)
		IF (Vulkan_LIBRARY)
			set(Vulkan_FOUND ON)
		ENDIF()
	ENDIF()
	find_package(ASSIMP REQUIRED)
	find_package(Threads REQUIRED)
IF(USE_D2D_WSI)
	MESSAGE("Using direct to display extension...")
	add_definitions(-D_DIRECT2DISPLAY)
ELSE(USE_D2D_WSI)
	find_package(XCB REQUIRED)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DVK_USE_PLATFORM_XCB_KHR")
ENDIF(USE_D2D_WSI)
	# Todo : android?
ENDIF(WIN32)

IF (NOT Vulkan_FOUND)
	message(FATAL_ERROR "Could not find Vulkan library!")
ELSE()
	message(STATUS ${Vulkan_LIBRARY})
ENDIF()

MESSAGE ("-- CMAKE_MODULE_PATH: ${CMAKE_MODULE_PATH}")
MESSAGE ("-- Vulkan_INCLUDE_DIR: ${Vulkan_INCLUDE_DIR}")

# Set preprocessor defines
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNOMINMAX -D_USE_MATH_DEFINES")

add_definitions(-D_CRT_SECURE_NO_WARNINGS)
add_definitions(-std=c++11)

file(GLOB SOURCE *.cpp base/*.cpp)

# Function for building single example
function(buildExample EXAMPLE_NAME)
	file(GLOB SOURCE *.cpp base/*.cpp ${EXAMPLE_NAME}/*.cpp)
	if(WIN32)
		add_executable(${EXAMPLE_NAME} WIN32 ${EXAMPLE_NAME}/${EXAMPLE_NAME}.cpp ${SOURCE})
		target_link_libraries(${EXAMPLE_NAME} ${Vulkan_LIBRARY} ${ASSIMP_LIBRARIES} ${WINLIBS})
	else(WIN32)
		add_executable(${EXAMPLE_NAME} ${EXAMPLE_NAME}/${EXAMPLE_NAME}.cpp ${SOURCE})
		target_link_libraries(${EXAMPLE_NAME} ${Vulkan_LIBRARY} ${ASSIMP_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
	endif(WIN32)
endfunction(buildExample)

# Build all examples
function(buildExamples)
	foreach(EXAMPLE ${EXAMPLES})
		buildExample(${EXAMPLE})
	endforeach(EXAMPLE)
endfunction(buildExamples)

# Compiler specific stuff
IF(MSVC)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
ENDIF(MSVC)

IF(WIN32)
	# Nothing here (yet)
ELSE(WIN32)
	link_libraries(${XCB_LIBRARIES} ${Vulkan_LIBRARY})
ENDIF(WIN32)

#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/")

#set(EXAMPLES 
#	bloom
#	computecullandlod
#)

#buildExamples()


add_subdirectory (basicTest)