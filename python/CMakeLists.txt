########################################################################
# Project setup
########################################################################
cmake_minimum_required(VERSION 2.8)
project(SoapySDR CXX)
enable_testing()

find_package(SoapySDR CONFIG REQUIRED)

########################################################################
# Find SWIG
########################################################################
include(FindSWIG)

if (NOT SWIG_FOUND)
    message(WARNING "Python bindings require SWIG, skipping...")
    return()
endif ()

include(UseSWIG)

########################################################################
# Find python interp
########################################################################
include(FindPythonInterp)
if (NOT PYTHONINTERP_FOUND)
    message(WARNING "Python bindings require python exe, skipping...")
endif ()
message(STATUS "PYTHON_EXECUTABLE: ${PYTHON_EXECUTABLE}")

########################################################################
# Find Python libs
########################################################################
include(FindPythonLibs)

if (NOT PYTHONLIBS_FOUND)
    message(WARNING "Python bindings require python dev, skipping...")
    return()
endif ()

message(STATUS "PYTHON_INCLUDE_DIRS ${PYTHON_INCLUDE_DIRS}")
message(STATUS "PYTHON_LIBRARIES ${PYTHON_LIBRARIES}")

########################################################################
# Build Module
########################################################################
include_directories(${SoapySDR_INCLUDE_DIRS})
include_directories(${PYTHON_INCLUDE_DIRS})

set(CMAKE_SWIG_FLAGS -c++)
set_source_files_properties(SoapySDR.i PROPERTIES CPLUSPLUS ON)

SWIG_ADD_MODULE(SoapySDR python SoapySDR.i)
SWIG_LINK_LIBRARIES(SoapySDR ${SoapySDR_LIBRARIES} ${PYTHON_LIBRARIES})

########################################################################
# Determine install directory
########################################################################
execute_process(
    COMMAND ${PYTHON_EXECUTABLE} -c "import distutils.sysconfig; print(distutils.sysconfig.get_python_lib(prefix='${CMAKE_INSTALL_PREFIX}'))"
    OUTPUT_STRIP_TRAILING_WHITESPACE
    OUTPUT_VARIABLE PYTHON_MODULE_DIR
)
message(STATUS "PYTHON_MODULE_DIR: ${PYTHON_MODULE_DIR}")

########################################################################
# Install Module
########################################################################
install(
    TARGETS ${SWIG_MODULE_SoapySDR_REAL_NAME}
    DESTINATION ${PYTHON_MODULE_DIR}
)

install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/SoapySDR.py
    DESTINATION ${PYTHON_MODULE_DIR}
)