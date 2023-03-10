include(${TEST_HARNESS_DIR}/test_harness.cmake)

##--------------------------------------------------------------------------##
## Install example files used in tests
##--------------------------------------------------------------------------##
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Inp_DirSolidification.txt
  ${CMAKE_CURRENT_BINARY_DIR}/Inp_DirSolidification.txt COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Inp_DirSolidification.json
  ${CMAKE_CURRENT_BINARY_DIR}/Inp_DirSolidification.json COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Inp_SpotMelt.txt
  ${CMAKE_CURRENT_BINARY_DIR}/Inp_SpotMelt.txt COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Inp_SpotMelt.json
  ${CMAKE_CURRENT_BINARY_DIR}/Inp_SpotMelt.json COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Materials/Inconel625
  ${CMAKE_CURRENT_BINARY_DIR}/Inconel625 COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Materials/Inconel625.json
  ${CMAKE_CURRENT_BINARY_DIR}/Inconel625.json COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Materials/Inconel625_Quadratic.json
  ${CMAKE_CURRENT_BINARY_DIR}/Inconel625_Quadratic.json COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../examples/Materials/SS316.json
  ${CMAKE_CURRENT_BINARY_DIR}/SS316.json COPYONLY)

ExaCA_add_tests_nobackend(PACKAGE ExaCA NAMES Init)

ExaCA_add_tests_nobackend(MPI PACKAGE ExaCA NAMES InitMPI)

ExaCA_add_tests(PACKAGE ExaCA NAMES KokkosInit KokkosUpdate KokkosPrint)

ExaCA_add_tests(MPI PACKAGE ExaCA NAMES KokkosInitMPI KokkosMPI)
