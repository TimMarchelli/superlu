# Required version
cmake_minimum_required(VERSION 3.22)

# Project Version
project(SuperLU C)

set(VERSION_MAJOR "7")
set(VERSION_MINOR "0")
set(VERSION_BugFix "0")
set(PROJECT_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_BugFix})

# set up options
option(enable_single    "Enable single precision library" ON)
option(enable_double    "Enable double precision library" ON)
option(enable_complex   "Enable single precision complex library" ON)
option(enable_complex16 "Enable double precision complex (complex16) library" ON)
option(enable_matlabmex "Build the Matlab mex library" OFF)
option(enable_doc       "Add target 'doc' to build Doxygen documentation" OFF)
option(enable_examples  "Build examples" ON)
option(enable_fortran   "Build Fortran interface" OFF)
option(enable_tests     "Build tests" ON)
option(enable_compatibility_complex "Provide typdef 'complex' for compatibility with older SuperLU version" OFF)

#-- BLAS
option(TPL_ENABLE_INTERNAL_BLASLIB  "Build the CBLAS library" OFF)
option(TPL_BLAS_LIBRARIES "List of absolute paths to blas libraries [].")
#-- METIS
option(TPL_ENABLE_METISLIB   "Build the METIS library" OFF)
option(TPL_METIS_LIBRARIES "List of absolute paths to METIS link libraries [].")
option(TPL_METIS_INCLUDE_DIRS "List of absolute paths to METIS include directories [].")

# global modules
include(CTest)
include(GNUInstallDirs)
include(FeatureSummary)

# set install targets
set(INSTALL_RUNTIME_DIR ${CMAKE_INSTALL_BINDIR})
set(INSTALL_CONFIG_DIR  ${CMAKE_INSTALL_LIBDIR}/cmake/superlu)
set(INSTALL_PKGCONFIG_DIR  ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
set(INSTALL_LIBRARY_DIR ${CMAKE_INSTALL_LIBDIR})
set(INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR})


######################################################################
#
# Usual initialization stuff
#
######################################################################

# export all symbols
if(BUILD_SHARED_LIBS)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()

#---- set RPATH handling
if(NOT APPLE)
  set(CMAKE_INSTALL_RPATH $origin)
endif()

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# set a debug postfix
set(CMAKE_DEBUG_POSTFIX "d")

if (enable_fortran)
  enable_language(Fortran)
endif()
set(SUPERLU_VERSION "${PROJECT_VERSION}")
set(SUPERLU_REV "${PROJECT_REV}")

if(enable_compatibility_complex)
  add_compile_definitions(SUPERLU_TYPEDEF_COMPLEX)
endif()

if(MSVC)
  add_compile_definitions(NOMINMAX _COMPLEX_DEFINED)
  # suppress compiler warnings that sprintf and similar functions are unsafe (e.g. C4996)
  add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
endif()

######################################################################
#
# Find packages
#
######################################################################
#
#--------------------- BLAS ---------------------
if(NOT TPL_ENABLE_INTERNAL_BLASLIB)
  # Use explicit supplied BLAS library
  if (TPL_BLAS_LIBRARIES)
    set(BLAS_FOUND TRUE)
    message("-- Using TPL_BLAS_LIBRARIES='${TPL_BLAS_LIBRARIES}'")
    set(BLAS_LIB ${TPL_BLAS_LIBRARIES})

  # use cmake to find BLAS
  else()
    find_package(BLAS)

    # if found, set the BLAS_LIB for linking into the library later
    if (BLAS_FOUND)
      
      set(BLAS_LIB BLAS::BLAS)

      # logic to determine type of BLAS and import it directly if possible. Importing directly populates more 
      # information about the import library than FindBLAS does. This is important in windows where we need to 
      # know the location of the imported DLL.
      if(BLAS_LIBRARIES MATCHES "[Oo][Pp][Ee][Nn][Bb][Ll][Aa][Ss]")
        message(STATUS "Using OpenBLAS import library.")
        find_package(OpenBLAS REQUIRED)
        set(BLAS_LIB OpenBLAS::OpenBLAS)

      # add more logic to directly import other BLAS implementations
      else()
        message(STATUS "Using cmake found BLAS.")
      endif()
    endif()

  endif()
endif()

# Build the internal BLAS as static library
if(NOT BLAS_FOUND)
  message("-- Did not find or specify BLAS so configure to build internal CBLAS ...")
  add_subdirectory(CBLAS)
  set(BLAS_LIB blas)
endif()

#--------------------- METIS ---------------------
if(TPL_ENABLE_METISLIB)
  # first check cmake
  find_package(METIS)
  if(METIS_FOUND)
    message(STATUS "Using cmake found METIS.")
    set(METIS_LIB METIS::metis)
    set(METIS_INCLUDE "")
  endif()

  # check options
  if(NOT METIS_FOUND)
    # verify lib and include set
    if(NOT TPL_METIS_LIBRARIES)
      message(FATAL_ERROR "TPL_METIS_LIBRARIES option should be set for METIS support to be enabled.")
    endif()
    if(NOT TPL_METIS_INCLUDE_DIRS)
      message(FATAL_ERROR "TPL_METIS_INCLUDE_DIRS option be set for METIS support to be enabled.")
    endif()
    if (EXISTS ${TPL_METIS_INCLUDE_DIRS})
      message(FATAL_ERROR "METIS include directory not found: ${TPL_METIS_INCLUDE_DIRS}")
    endif()

    # set METIS vars
    set(METIS_FOUND TRUE)
    set(METIS_LIB ${TPL_METIS_LIBRARIES})
    set(METIS_INCLUDE ${TPL_METIS_INCLUDE_DIRS})
  endif()

  message("-- Enabled support for METIS.")
else()
  message("-- Will not link with METIS.")
endif()

if(METIS_FOUND)
  set(HAVE_METIS TRUE)
endif()

if(MSVC AND (enable_tests OR enable_examples))
  # try to find wingetopt
  find_package(wingetopt QUIET)
endif()


######################################################################
#
# Include directories
#
######################################################################
include_directories(${CMAKE_BINARY_DIR}/SRC) # For superlu_config.h
if (TPL_METIS_INCLUDE_DIRS)
  include_directories(${TPL_METIS_INCLUDE_DIRS})  ## metis
endif ()

# configure and install package config
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/superlu.pc.in ${CMAKE_CURRENT_BINARY_DIR}/superlu.pc @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/superlu.pc
  DESTINATION ${INSTALL_PKGCONFIG_DIR}
)

# Configure header file
configure_file(${SuperLU_SOURCE_DIR}/SRC/superlu_config.h.in ${SuperLU_BINARY_DIR}/SRC/superlu_config.h)

# Following is to configure a header file for FORTRAN code
configure_file(${SuperLU_SOURCE_DIR}/SRC/superlu_config.h.in ${SuperLU_BINARY_DIR}/FORTRAN/superlu_config.h)

######################################################################
#
# Add subdirectories
#
######################################################################

add_subdirectory(SRC)

if(enable_matlabmex)
  add_subdirectory(MATLAB)
endif()

if(enable_tests)
  add_subdirectory(TESTING)
endif()

if (enable_examples)
  add_custom_target(examples)
  add_subdirectory(EXAMPLE)
endif()

if (enable_fortran)
  add_subdirectory(FORTRAN)
endif()

if (enable_doc)
   add_subdirectory(DOC)
endif()
