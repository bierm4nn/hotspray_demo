cmake_minimum_required(VERSION 2.8.3)
project(hotspray_ur5_support)
add_definitions(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
  actionlib
  control_msgs
  godel_msgs
  industrial_robot_simulator_service
  roscpp
  trajectory_msgs
)

find_package(PkgConfig)
pkg_check_modules(PC_MODBUS QUIET libmodbus)

find_path(MODBUS_INCLUDE_DIR modbus/modbus.h
          HINTS ${PC_MODBUS_INCLUDEDIR} ${PC_MODBUS_INCLUDE_DIRS}
)
find_library(MODBUS_LIBRARY NAMES modbus libmodbus
             HINTS ${PC_MODBUS_LIBDIR} ${PC_MODBUS_LIBRARY_DIRS} )

mark_as_advanced(MODBUS_INCLUDE_DIR MODBUS_LIBRARY )

if(NOT MODBUS_INCLUDE_DIR OR NOT MODBUS_LIBRARY)
  message(FATAL_ERROR "modbus library not found")
endif()

catkin_package()

include_directories(${catkin_INCLUDE_DIRS} ${MODBUS_INCLUDE_DIR}
)

add_executable(ur5_hotspray_process_service_node
  src/ur5_hotspray_process_service_node.cpp
)
add_dependencies(ur5_hotspray_process_service_node ${catkin_EXPORTED_TARGETS})

target_link_libraries(ur5_hotspray_process_service_node
  ${catkin_LIBRARIES}
  ${MODBUS_LIBRARY}
)

add_executable(spindle_sim
  src/spindle_sim.cpp
)

target_link_libraries(spindle_sim
  ${MODBUS_LIBRARY}
)

install(TARGETS ur5_hotspray_process_service_node spindle_sim
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
