cmake_minimum_required(VERSION 2.8.3)
project(imu_utils)

## Compile as C++11, supported in ROS noetic and newer
set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++14")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g -fPIC -fopenmp")

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED
    roscpp
    std_msgs
    geometry_msgs
    nav_msgs
    )

## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)
find_package(OpenCV REQUIRED)
find_package(Ceres REQUIRED)
find_package(Eigen3 REQUIRED)
include_directories(
        include
        include/imu_utils
        include/imu_utils/cv_utils
        include/imu_utils/cv_utils/dlt
        include/imu_utils/cv_utils/pnp
        include/imu_utils/math_utils
        include/imu_utils/sys_utils
        ${catkin_INCLUDE_DIRS}
        ${EIGEN3_INCLUDE_DIR}
)

catkin_package(
        INCLUDE_DIRS include
        LIBRARIES polynomial cv_utils pnp
        CATKIN_DEPENDS roscpp std_msgs
        #    DEPENDS system_lib
)

add_library(polynomial SHARED
        src/math_utils/Polynomial.cpp)
target_link_libraries(polynomial ${Boost_LIBRARIES} )

add_library(pnp SHARED
        src/cv_utils/dlt/dlt.cpp
        src/cv_utils/pnp/pnp.cpp
        src/cv_utils/pnp/linearpnp.cpp
        src/cv_utils/pnp/nonlinearpnp.cpp)
target_link_libraries(pnp ${Boost_LIBRARIES}  ${OpenCV_LIBS}  ${CERES_LIBRARIES})

add_library(cv_utils SHARED
        src/cv_utils.cc
        )
target_link_libraries(cv_utils ${Boost_LIBRARIES}  ${OpenCV_LIBS} )

add_executable(matIO_test   src/mat_io_test.cpp )
target_link_libraries(matIO_test dw ${OpenCV_LIBS})

add_executable(sumpixel_test   src/sumpixel_test.cpp )
target_link_libraries(sumpixel_test dw ${OpenCV_LIBS})

set(ACC_LIB_SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/src/acc_lib/allan_acc.cpp
    ${PROJECT_SOURCE_DIR}/src/acc_lib/fitallan_acc.cpp
    )
set(GYR_LIB_SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/src/gyr_lib/allan_gyr.cpp
    ${PROJECT_SOURCE_DIR}/src/gyr_lib/fitallan_gyr.cpp
    )

add_executable(imu_an
    src/imu_an.cpp
    ${GYR_LIB_SOURCE_FILES}
    ${ACC_LIB_SOURCE_FILES}
    )

target_link_libraries(imu_an ${catkin_LIBRARIES} ${OpenCV_LIBS} ${CERES_LIBRARIES})
