# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(camera_aravis)

include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules)
find_package(catkin REQUIRED COMPONENTS roscpp sensor_msgs image_transport camera_info_manager dynamic_reconfigure driver_base tf dynamic_reconfigure)
find_package(GLIB2 REQUIRED)

ExternalProject_Add(Aravis
   DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}/external
   URL "http://ftp.acc.umu.se/pub/GNOME/sources/aravis/0.6/aravis-0.6.4.tar.xz"
   UPDATE_COMMAND ""
   SOURCE_DIR "${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4"
   BUILD_IN_SOURCE 1
   CONFIGURE_COMMAND ./configure --prefix=${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4/install
   BUILD_COMMAND make -j
   INSTALL_COMMAND make install
)

# Include and Link Aravis
set (aravis_install_dir ${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4/install)

set(Aravis_INCLUDE_DIRS ${aravis_install_dir}/include/aravis-0.6 ${GLIB2_INCLUDE_DIRS})


#find_library(ARAVIS_LIBRARY
#	NAMES "aravis-0.6"
#  HINTS ${aravis_install_dir}/lib
#)

#file( GLOB ARAVIS_LIBRARY "${aravis_install_dir}/lib/*.so*" )
set(ARAVIS_LIBRARY ${aravis_install_dir}/lib/libaravis-0.6.so)


generate_dynamic_reconfigure_options(cfg/CameraAravisConfig.cfg)
catkin_package(
    DEPENDS Aravis GLIB2
    CATKIN_DEPENDS roscpp sensor_msgs image_transport camera_info_manager dynamic_reconfigure driver_base tf
    INCLUDE_DIRS
    LIBRARIES
)

include_directories(cfg/cpp ${catkin_INCLUDE_DIRS} ${Aravis_INCLUDE_DIRS} ${GLIB2_INCLUDE_DIRS})

add_executable(camnode src/camnode.cpp)
add_dependencies(camnode Aravis)
add_dependencies(camnode ${PROJECT_NAME}_gencfg)
target_link_libraries(camnode ${catkin_LIBRARIES} ${ARAVIS_LIBRARY} glib-2.0 gmodule-2.0 gobject-2.0)
