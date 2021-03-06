cmake_minimum_required (VERSION 2.6)

project (hsm_book_samples)

# Pass "-DHSM_DEBUG=ON" to CMake to enable debug features
# Note that this gets automatically defined in MSVC Debug configurations because it defines _DEBUG
option(HSM_DEBUG "Enable HSM debug features" OFF)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP") # Multiprocessor build
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /WX") # Max warning level and error on warnings
	if(MSVC_VERSION LESS 1900) # Starting from MSVC 14 (2015), STL needs language extensions enabled
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Za") # Disable language extensions
	endif()
elseif ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
endif()

function(add_hsm_lib)
	message(STATUS "Adding hsm lib")
	# Add a header-only project (an INTERFACE library in CMake)
	add_library(hsm INTERFACE)
	target_include_directories(hsm INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../../include>)
endfunction(add_hsm_lib)

# Adds a project (exe) per source file found in source/${CHAPTER_NAME}
# Note: won't work if a sample spans multiple source files
function(add_chapter_samples CHAPTER_NAME)
	file(GLOB SAMPLES_SRC "source/${CHAPTER_NAME}/*.cpp")

	foreach(SRC_FILE ${SAMPLES_SRC})
		get_filename_component(SRC_NAME ${SRC_FILE} NAME_WE)
		set(PROJ_NAME "${CHAPTER_NAME}_${SRC_NAME}")
		message(STATUS "Adding project ${PROJ_NAME}")
		add_executable("${PROJ_NAME}" ${SRC_FILE})
		target_link_libraries("${PROJ_NAME}" hsm) # link against hsm lib
		if(HSM_DEBUG)
			target_compile_definitions("${PROJ_NAME}" PRIVATE HSM_DEBUG=1)
		endif()			
	endforeach(SRC_FILE)
endfunction(add_chapter_samples)

# add hsm library
add_hsm_lib()

# add sample exes
add_chapter_samples("ch2")
add_chapter_samples("ch3")
add_chapter_samples("ch4")
add_chapter_samples("ch5")
