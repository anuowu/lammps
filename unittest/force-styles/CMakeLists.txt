
find_package(YAML)
if(NOT YAML_FOUND)
  message(STATUS "Skipping tests because libyaml is not found")
  return()
endif()

set(TEST_INPUT_FOLDER ${CMAKE_CURRENT_SOURCE_DIR}/tests)
add_library(style_tests STATIC yaml_writer.cpp error_stats.cpp test_config_reader.cpp test_main.cpp)
target_compile_definitions(style_tests PRIVATE TEST_INPUT_FOLDER=${TEST_INPUT_FOLDER})
target_include_directories(style_tests PRIVATE ${LAMMPS_SOURCE_DIR})
target_link_libraries(style_tests PUBLIC GTest::GTest GTest::GMock Yaml::Yaml)
if(BUILD_MPI)
  target_link_libraries(style_tests PUBLIC MPI::MPI_CXX)
else()
  target_link_libraries(style_tests PUBLIC mpi_stubs)
endif()
# propagate sanitizer options to test tools
if (NOT ENABLE_SANITIZER STREQUAL "none")
  if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.13)
    target_compile_options(style_tests PUBLIC -fsanitize=${ENABLE_SANITIZER})
    target_link_options(style_tests PUBLIC -fsanitize=${ENABLE_SANITIZER})
  endif()
endif()

# unit test for error stats class
add_executable(test_error_stats test_error_stats.cpp)
target_include_directories(test_error_stats PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${LAMMPS_SOURCE_DIR})
target_link_libraries(test_error_stats PRIVATE GTest::GTestMain GTest::GTest)
add_test(NAME ErrorStats COMMAND test_error_stats)

# pair style tester
add_executable(pair_style pair_style.cpp)
target_link_libraries(pair_style PRIVATE lammps style_tests)

# tests for a molecular systems and related pair styles
file(GLOB MOL_PAIR_TESTS LIST_DIRECTORIES false ${TEST_INPUT_FOLDER}/mol-pair-*.yaml)
foreach(TEST ${MOL_PAIR_TESTS})
  string(REGEX REPLACE "^.*mol-pair-(.*)\.yaml" "MolPairStyle:\\1" TNAME ${TEST})
  add_test(NAME ${TNAME}
    COMMAND ${CMAKE_COMMAND} -DTEST_EXECUTABLE=$<TARGET_FILE:pair_style>
    -DTEST_INPUT=${TEST} -DTEST_NAME=${TNAME} -P ${CMAKE_CURRENT_SOURCE_DIR}/TestRunner.cmake
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endforeach()

# tests for metal-like atomic systems and related pair styles
file(GLOB ATOMIC_PAIR_TESTS LIST_DIRECTORIES false ${TEST_INPUT_FOLDER}/atomic-pair-*.yaml)
foreach(TEST ${ATOMIC_PAIR_TESTS})
  string(REGEX REPLACE "^.*atomic-pair-(.*)\.yaml" "AtomicPairStyle:\\1" TNAME ${TEST})
  add_test(NAME ${TNAME}
    COMMAND ${CMAKE_COMMAND} -DTEST_EXECUTABLE=$<TARGET_FILE:pair_style>
    -DTEST_INPUT=${TEST} -DTEST_NAME=${TNAME} -P ${CMAKE_CURRENT_SOURCE_DIR}/TestRunner.cmake
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  set_tests_properties(${TNAME} PROPERTIES ENVIRONMENT "LAMMPS_POTENTIALS=${LAMMPS_POTENTIALS_DIR}")
endforeach()

# tests for Si-like manybody systems and related pair styles
file(GLOB MANYBODY_PAIR_TESTS LIST_DIRECTORIES false ${TEST_INPUT_FOLDER}/manybody-pair-*.yaml)
foreach(TEST ${MANYBODY_PAIR_TESTS})
  string(REGEX REPLACE "^.*manybody-pair-(.*)\.yaml" "ManybodyPairStyle:\\1" TNAME ${TEST})
  add_test(NAME ${TNAME}
    COMMAND ${CMAKE_COMMAND} -DTEST_EXECUTABLE=$<TARGET_FILE:pair_style>
    -DTEST_INPUT=${TEST} -DTEST_NAME=${TNAME} -P ${CMAKE_CURRENT_SOURCE_DIR}/TestRunner.cmake
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  set_tests_properties(${TNAME} PROPERTIES ENVIRONMENT "LAMMPS_POTENTIALS=${LAMMPS_POTENTIALS_DIR}")
endforeach()

# bond style tester
add_executable(bond_style bond_style.cpp)
target_link_libraries(bond_style PRIVATE lammps style_tests)

file(GLOB BOND_TESTS LIST_DIRECTORIES false ${TEST_INPUT_FOLDER}/bond-*.yaml)
foreach(TEST ${BOND_TESTS})
  string(REGEX REPLACE "^.*bond-(.*)\.yaml" "BondStyle:\\1" TNAME ${TEST})
  add_test(NAME ${TNAME}
    COMMAND ${CMAKE_COMMAND} -DTEST_EXECUTABLE=$<TARGET_FILE:bond_style>
    -DTEST_INPUT=${TEST} -DTEST_NAME=${TNAME} -P ${CMAKE_CURRENT_SOURCE_DIR}/TestRunner.cmake
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endforeach()

# angle style tester
add_executable(angle_style angle_style.cpp)
target_link_libraries(angle_style PRIVATE lammps style_tests)

file(GLOB ANGLE_TESTS LIST_DIRECTORIES false ${TEST_INPUT_FOLDER}/angle-*.yaml)
foreach(TEST ${ANGLE_TESTS})
  string(REGEX REPLACE "^.*angle-(.*)\.yaml" "AngleStyle:\\1" TNAME ${TEST})
  add_test(NAME ${TNAME}
    COMMAND ${CMAKE_COMMAND} -DTEST_EXECUTABLE=$<TARGET_FILE:angle_style>
    -DTEST_INPUT=${TEST} -DTEST_NAME=${TNAME} -P ${CMAKE_CURRENT_SOURCE_DIR}/TestRunner.cmake
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endforeach()
