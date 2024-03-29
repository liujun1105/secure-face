cmake_minimum_required(VERSION 2.8)

if (POLICY CMP0048)
    cmake_policy (SET CMP0048 NEW)
endif ()

if (POLICY CMP0077)
    cmake_policy (SET CMP0077 NEW)
endif ()

set(CMAKE_PROJECT_VERSION 0.1.1)

project(NSecureFace 
	VERSION ${CMAKE_PROJECT_VERSION} 
	DESCRIPTION "Nomura Hackthon 2019 - Nomura Secure Face Project"
	LANGUAGES CXX
)


if(NOT MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
else()
    add_definitions(-D_WIN32_WINNT=0xA00)
endif()


add_subdirectory(deps)

find_package(AWSSDK REQUIRED COMPONENTS rekognition)
message("Found AWSSDK => ${AWSSDK_FOUND}")

if(${AWSSDK_FOUND})
	add_compile_definitions(AWS_FACE_RECOGNITION)

	message("AWSSDK Libraries => ${AWSSDK_LIB_DIR}")
	message("AWSSDK Include Directory => ${AWSSDK_INCLUDE_DIR}")
	message("AWSSDK Bin Directory => ${AWSSDK_BIN_DIR}")

	include_directories(${AWSSDK_INCLUDE_DIR})
endif()

find_package(OpenCV)
message("Found OpenCV => ${OpenCV_FOUND}")
if(${OpenCV_FOUND})
	message("OpenCV Libraries => ${OpenCV_LIBRARIES}")
	message("OpenCV Include Directory => ${OpenCV_INCLUDE_DIRS}")


	set(NSECURE_FACE_HEADERS 
		nsecureface.h
		simple-authorization.hpp
		face-processing.hpp
		common.hpp
        client.hpp
        socket.hpp
		image-sender.hpp
	)

	set(NSECURE_FACE_SOURCES
		nsecureface-service.cpp
		face-processing.cpp
		client.cpp
	)
	set(NSECURE_FACE_RESOURCES config.json)

	include_directories(
	    ${OpenCV_INCLUDE_DIRS} 
	    ${NSECURE_FACE_HEADERS}	
	    deps/nlohmann-json-3.7.0
	    deps/cppfs-1.3.0/source/cppfs/include
	    deps/nng-1.1.1/src
        deps/cpp-httplib-0.2.1
		deps/cpp-base64/include
	)

	add_executable(NSecureFaceService 
	    ${NSECURE_FACE_HEADERS} 
	    ${NSECURE_FACE_SOURCES}
	    ${NSECURE_FACE_RESOURCES}
	)

	if(${AWSSDK_FOUND})
		target_link_libraries(NSecureFaceService ${OpenCV_LIBRARIES} cppfs nng ${AWSSDK_LIB_DIR}/*.lib base64)
	else()
		target_link_libraries(NSecureFaceService ${OpenCV_LIBRARIES} cppfs nng base64)
	endif()

else()
	message("OpenCV is required for this project")
endif(${OpenCV_FOUND})
