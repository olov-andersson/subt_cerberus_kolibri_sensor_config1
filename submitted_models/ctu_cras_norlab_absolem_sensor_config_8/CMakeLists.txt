cmake_minimum_required(VERSION 2.8.3)
project(ctu_cras_norlab_absolem_sensor_config_8)

find_package(catkin REQUIRED)

catkin_package()

install(DIRECTORY launch config DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(FILES model.sdf model.config
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(PROGRAMS
  scripts/print_robot_urdf
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/scripts)

