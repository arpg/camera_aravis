# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(camera_aravis)

include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules)
find_package(catkin REQUIRED COMPONENTS roscpp sensor_msgs image_transport camera_info_manager dynamic_reconfigure driver_base tf dynamic_reconfigure)
#find_package(Aravis REQUIRED)
find_package(GLIB2 REQUIRED)

ExternalProject_Add(Aravis
   DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}/external
   URL "http://ftp.acc.umu.se/pub/GNOME/sources/aravis/0.6/aravis-0.6.4.tar.xz"
   UPDATE_COMMAND ""
   #mkdir "${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4/install"
   SOURCE_DIR "${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4"
   BUILD_IN_SOURCE 1
   CONFIGURE_COMMAND ./configure --prefix=${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4/install
   BUILD_COMMAND make -j
   INSTALL_COMMAND make install
)

# Include and Link Aravis
set (aravis_install_dir ${CMAKE_CURRENT_BINARY_DIR}/external/aravis-0.6.4/install)

set(Aravis_INCLUDE_DIRS ${aravis_install_dir}/include/aravis-0.6 ${GLIB2_INCLUDE_DIRS})

#message("ARAVIS INCLUDE DIRS: ${Aravis_INCLUDE_DIRS}") 

find_library(ARAVIS_LIBRARY
  NAMES "aravis-0.6"
  HINTS ${aravis_install_dir}/lib
)

set(Aravis_LIBRARIES ${ARAVIS_LIBRARY} ${GLIB2_LIBRARY} ${GObject_LIBRARY})

generate_dynamic_reconfigure_options(cfg/CameraAravisConfig.cfg)
catkin_package(
    DEPENDS Aravis GLIB2
    CATKIN_DEPENDS roscpp sensor_msgs image_transport camera_info_manager dynamic_reconfigure driver_base tf
    INCLUDE_DIRS
    LIBRARIES
)

include_directories(cfg/cpp ${catkin_INCLUDE_DIRS} ${Aravis_INCLUDE_DIRS} ${GLIB2_INCLUDE_DIRS})

add_executable(camnode src/camnode.cpp)

target_link_libraries(camnode ${catkin_LIBRARIES} ${Aravis_LIBRARIES} glib-2.0 gmodule-2.0 gobject-2.0)
add_dependencies(camnode ${PROJECT_NAME}_gencfg)
add_dependencies(camnode Aravis)
