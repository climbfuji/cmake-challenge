cmake_minimum_required(VERSION 3.10)

project(cmake_challenge
        VERSION 1.0.0
        LANGUAGES Fortran)

#------------------------------------------------------------------------------
set(PACKAGE "cmake-challenge")
set(AUTHORS "Dom Heinzeller" )

#------------------------------------------------------------------------------
find_package(MPI REQUIRED Fortran)
if(NOT MPI_Fortran_HAVE_F08_MODULE)
  message(FATAL_ERROR "MPI implementation does not support the Fortran 2008 mpi_f08 interface")
endif()

#------------------------------------------------------------------------------
if (OPENMP)
  find_package(OpenMP REQUIRED)
endif()

#------------------------------------------------------------------------------
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "Coverage")
endif()

#------------------------------------------------------------------------------
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  add_definitions(-DDEBUG)
endif()

#------------------------------------------------------------------------------
option(BUILD_SHARED_LIBS "Build a shared library" OFF)

#------------------------------------------------------------------------------
set(TYPEDEFS $ENV{CCPP_TYPEDEFS})
if(TYPEDEFS)
  message(STATUS "Got CCPP TYPEDEFS from environment variable")
else(TYPEDEFS)
  include(${CMAKE_CURRENT_BINARY_DIR}/CCPP_TYPEDEFS.cmake)
  message(STATUS "Got CCPP TYPEDEFS from cmakefile include file")
endif(TYPEDEFS)
list(REMOVE_DUPLICATES TYPEDEFS)

foreach(typedef_module ${TYPEDEFS})
    list(APPEND MODULES_F90 ${CMAKE_CURRENT_BINARY_DIR}/${typedef_module})
endforeach()

#------------------------------------------------------------------------------
set(SCHEMES $ENV{CCPP_SCHEMES})
if(SCHEMES)
  message(STATUS "Got CCPP SCHEMES from environment variable")
else(SCHEMES)
  include(${CMAKE_CURRENT_BINARY_DIR}/CCPP_SCHEMES.cmake)
  message(STATUS "Got CCPP SCHEMES from cmakefile include file")
endif(SCHEMES)
list(REMOVE_DUPLICATES SCHEMES)

set(CAPS $ENV{CCPP_CAPS})
if(CAPS)
  message(STATUS "Got CCPP CAPS from environment variable")
else(CAPS)
  include(${CMAKE_CURRENT_BINARY_DIR}/CCPP_CAPS.cmake)
  message(STATUS "Got CCPP CAPS from cmakefile include file")
endif(CAPS)
list(REMOVE_DUPLICATES CAPS)

get_filename_component(FULL_PATH_TO_CMAKELISTS CMakeLists.txt REALPATH BASE_DIR ${LOCAL_CURRENT_SOURCE_DIR})
get_filename_component(LOCAL_CURRENT_SOURCE_DIR ${FULL_PATH_TO_CMAKELISTS} DIRECTORY)

#------------------------------------------------------------------------------

set(SCHEMES_OPENMP_OFF ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rrtmgp/mo_gas_optics.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rrtmgp/mo_rrtmgp_constants.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rrtmgp/mo_gas_concentrations.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rrtmgp/mo_rrtmgp_util_string.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rrtmgp/kernels/mo_gas_optics_kernels.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rrtmgp/mo_gas_optics_rrtmgp.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/mo_rrtmgp_clr_all_sky.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/mo_fluxes_byband.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/solar_variability/mo_solar_variability.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/mo_heating_rates.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/mo_fluxes_bygpoint.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/mo_compute_bc.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/cloud_optics/mo_cloud_sampling.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/extensions/cloud_optics/mo_cloud_optics.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_rte_config.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_source_functions.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_rte_sw.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_fluxes.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_rte_lw.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_rte_util_array.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/kernels/mo_rte_solver_kernels.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/kernels/mo_optical_props_kernels.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/kernels/mo_fluxes_broadband_kernels.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_rte_kind.F90
                       ${LOCAL_CURRENT_SOURCE_DIR}/physics/Radiation/RRTMGP/rte-rrtmgp/rte/mo_optical_props.F90)

set(SCHEMES_DYNAMICS)

if(${LOCAL_CURRENT_SOURCE_DIR}/physics/MP/GFDL/fv_sat_adj.F90 IN_LIST SCHEMES)
  list(APPEND SCHEMES_DYNAMICS ${LOCAL_CURRENT_SOURCE_DIR}/physics/MP/GFDL/fv_sat_adj.F90)
endif()

if(SCHEMES_DYNAMICS)
  SET_PROPERTY(SOURCE ${SCHEMES_DYNAMICS}
               APPEND_STRING PROPERTY COMPILE_FLAGS " ${CMAKE_Fortran_FLAGS_DYNAMICS} ${OpenMP_Fortran_FLAGS}")
  list(REMOVE_ITEM SCHEMES ${SCHEMES_DYNAMICS})
endif()

if(SCHEMES_OPENMP_OFF)
  SET_PROPERTY(SOURCE ${SCHEMES_OPENMP_OFF}
               APPEND_STRING PROPERTY COMPILE_FLAGS " ${CMAKE_Fortran_FLAGS_PHYSICS}")
  list(REMOVE_ITEM SCHEMES ${SCHEMES_OPENMP_OFF})
endif()

SET_PROPERTY(SOURCE ${SCHEMES} ${CAPS}
             APPEND_STRING PROPERTY COMPILE_FLAGS " ${CMAKE_Fortran_FLAGS_PHYSICS} ${OpenMP_Fortran_FLAGS}")

if(CMAKE_BUILD_TYPE STREQUAL "Release" AND ${CMAKE_Fortran_COMPILER_ID} STREQUAL "Intel")
  set(SCHEME_NAMES_LOWER_OPTIMIZATION module_sf_mynn.F90
                                      mynnedmf_wrapper.F90
                                      gcycle.F90)
  foreach(SCHEME_NAME IN LISTS SCHEME_NAMES_LOWER_OPTIMIZATION)
    set(SCHEMES_TMP ${SCHEMES})
    list(FILTER SCHEMES_TMP INCLUDE REGEX ".*${SCHEME_NAME}$")
    SET_SOURCE_FILES_PROPERTIES(${SCHEMES_TMP}
                                APPEND_STRING PROPERTY COMPILE_FLAGS
                                " ${CMAKE_Fortran_FLAGS_PHYSICS} ${OpenMP_Fortran_FLAGS} -O1")
  endforeach()
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release" AND ${CMAKE_Fortran_COMPILER_ID} STREQUAL "Intel")
  set(SCHEME_NAMES_NO_OPTIMIZATION GFS_typedefs.F90)
  foreach(SCHEME_NAME IN LISTS SCHEME_NAMES_NO_OPTIMIZATION)
    set(SCHEMES_TMP ${SCHEMES})
    list(FILTER SCHEMES_TMP INCLUDE REGEX ".*${SCHEME_NAME}$")
    SET_SOURCE_FILES_PROPERTIES(${SCHEMES_TMP}
                                APPEND_STRING PROPERTY COMPILE_FLAGS " ${CMAKE_Fortran_FLAGS_PHYSICS} ${OpenMP_Fortran_FLAGS} -O0")
  endforeach()
endif()

#------------------------------------------------------------------------------

add_library(ccpp_physics STATIC ${SCHEMES} ${SCHEMES_OPENMP_OFF} ${SCHEMES_DYNAMICS} ${CAPS})
foreach(source_f90 ${CAPS})
    get_filename_component(tmp_source_f90 ${source_f90} NAME)
    string(REGEX REPLACE ".F90" ".mod" tmp_module_f90 ${tmp_source_f90})
    string(TOLOWER ${tmp_module_f90} module_f90)
    list(APPEND MODULES_F90 ${CMAKE_CURRENT_BINARY_DIR}/${module_f90})
endforeach()

set_target_properties(ccpp_physics PROPERTIES VERSION ${PROJECT_VERSION}
                                     SOVERSION ${PROJECT_VERSION_MAJOR})

target_include_directories(ccpp_physics PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

target_link_libraries(ccpp_physics PUBLIC w3emc::w3emc_d
                                          sp::sp_d
                                          NetCDF::NetCDF_Fortran)

install(TARGETS ccpp_physics
        EXPORT ccpp_physics-targets
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION lib
)
install(EXPORT ccpp_physics-targets
        FILE ccpp_physics-config.cmake
        DESTINATION lib/cmake
)
#install(FILES ${HEADERS_C} DESTINATION include)
install(FILES ${MODULES_F90} DESTINATION include)
