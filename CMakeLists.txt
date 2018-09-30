cmake_minimum_required(VERSION 2.8.3)
project(serial)

set(SERIAL_INCLUDE_INSTALL_DIR "include")
set(SERIAL_LIB_INSTALL_DIR "lib")

# Find catkin
find_package(catkin)

if(APPLE)
	find_library(IOKIT_LIBRARY IOKit)
	find_library(FOUNDATION_LIBRARY Foundation)
endif()

if(UNIX AND NOT APPLE)
    # If Linux, add rt and pthread
    set(rt_LIBRARIES rt)
    set(pthread_LIBRARIES pthread)
    # Check if catkin has been founf
    if (catkin_FOUND)
        catkin_package(
            LIBRARIES ${PROJECT_NAME}
            INCLUDE_DIRS include
            DEPENDS rt pthread
        )
    endif()
else()
    # Otherwise normal call
    # Check if catkin has been founf
    if (catkin_FOUND)
        catkin_package(
            LIBRARIES ${PROJECT_NAME}
            INCLUDE_DIRS include
        )
    endif()
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(APPLE)
	# If OSX
	list(APPEND serial_SRCS src/impl/unix.cc)
	list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(APPLE)
	target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
	target_link_libraries(${PROJECT_NAME} rt pthread)
else()
	target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
add_dependencies(serial_example ${PROJECT_NAME})
target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

# Check if catkin has been founf
if (catkin_FOUND)
    ## Install executable
    install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    )

    ## Install headers
    install(FILES include/serial/serial.h include/serial/v8stdint.h
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial
    )

    ## Tests
    if(CATKIN_ENABLE_TESTING)
        add_subdirectory(tests)
    endif()
else()
    ## Install executable
    install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${SERIAL_LIB_INSTALL_DIR}
        LIBRARY DESTINATION ${SERIAL_LIB_INSTALL_DIR}
    )

    ## Install headers
    install(FILES include/serial/serial.h include/serial/v8stdint.h
        DESTINATION ${SERIAL_INCLUDE_INSTALL_DIR}/serial
    )
endif()
