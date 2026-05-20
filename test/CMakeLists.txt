#------------------------------------------------------------------------------
# Include source and build directories
#------------------------------------------------------------------------------
include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
)

#------------------------------------------------------------------------------
# External dependencies
#------------------------------------------------------------------------------
find_package(MPI REQUIRED)
if(MPI_C_FOUND)
  set(HDF5_VFD_GDS_EXT_LIB_DEPENDENCIES
    ${HDF5_VFD_GDS_EXT_LIB_DEPENDENCIES}
    ${MPI_C_LIBRARIES}
  )
  include_directories(
    ${MPI_C_INCLUDE_DIRS}
  )
endif()

#-----------------------------------------------------------------------------
# Define Sources and tests
#-----------------------------------------------------------------------------
set(vfd_gds_tests
  gds_test
)

foreach(vfd_test ${vfd_gds_tests})
  add_executable(${vfd_test}
    ${CMAKE_CURRENT_SOURCE_DIR}/${vfd_test}.c
  )
  target_include_directories(${vfd_test}
    PUBLIC  "$<BUILD_INTERFACE:${HDF5_VFD_GDS_BUILD_INCLUDE_DEPENDENCIES}>"
            $<INSTALL_INTERFACE:${HDF5_VFD_GDS_INSTALL_INCLUDE_INTERFACE}>
  )
  target_include_directories(${vfd_test}
    SYSTEM PUBLIC ${HDF5_VFD_GDS_EXT_INCLUDE_DEPENDENCIES}
  )
  target_link_libraries(${vfd_test}
      ${HDF5_VFD_GDS_EXPORTED_LIBS}
      ${HDF5_VFD_GDS_EXT_LIB_DEPENDENCIES}
      ${HDF5_VFD_GDS_EXT_PKG_DEPENDENCIES}
      CUDA::cudart
  )
endforeach()

#-----------------------------------------------------------------------------
# Tests
# Run via mpiexec so MPI initialises correctly on compute nodes.
# CUDA_VISIBLE_DEVICES=0 pins the single rank to GPU 0, which is required
# before cuFileDriverOpen() is called inside the VFD.
#-----------------------------------------------------------------------------
set(HDF5_VFD_GDS_CUFILE_JSON "" CACHE STRING
  "Path to cufile.json passed as CUFILE_ENV_PATH_JSON to the test (optional)")

foreach(vfd_test ${vfd_gds_tests})
  add_test(NAME "${vfd_test}"
    COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} 1 $<TARGET_FILE:${vfd_test}>
  )
  set(_test_env
    "HDF5_PLUGIN_PATH=${CMAKE_RUNTIME_OUTPUT_DIRECTORY}"
    "CUDA_VISIBLE_DEVICES=0"
  )
  if(HDF5_VFD_GDS_CUFILE_JSON)
    list(APPEND _test_env "CUFILE_ENV_PATH_JSON=${HDF5_VFD_GDS_CUFILE_JSON}")
  endif()
  set_tests_properties(${vfd_test} PROPERTIES
    ENVIRONMENT "${_test_env}"
  )
endforeach()
