# Copyright (C) 1995-2019, Rene Brun and Fons Rademakers.
# All rights reserved.
#
# For the licensing terms see $ROOTSYS/LICENSE.
# For the list of contributors see $ROOTSYS/README/CREDITS.

#######################################################################################
# CMakeLists.txt file for building JupyROOT bindings/pyroot_experimental/PyROOT package
#######################################################################################

set(py_sources
  JupyROOT/__init__.py
  JupyROOT/helpers/__init__.py
  JupyROOT/helpers/cppcompleter.py
  JupyROOT/helpers/handlers.py
  JupyROOT/helpers/utils.py
  JupyROOT/html/__init__.py
  JupyROOT/html/cpphighlighter.py
  JupyROOT/kernel/__init__.py
  JupyROOT/kernel/rootkernel.py
  JupyROOT/kernel/utils.py
  JupyROOT/kernel/magics/__init__.py
  JupyROOT/kernel/magics/cppmagic.py
  JupyROOT/kernel/magics/jsrootmagic.py
  JupyROOT/magics/__init__.py
  JupyROOT/magics/cppmagic.py
  JupyROOT/magics/jsrootmagic.py
)

set(JupyROOTPySrcDir python/JupyROOT)
file(COPY ${JupyROOTPySrcDir} DESTINATION ${localruntimedir})

# Install .py source files
install(DIRECTORY ${JupyROOTPySrcDir} DESTINATION ${runtimedir})

foreach(val RANGE ${how_many_pythons})
  list(GET python_under_version_strings ${val} python_under_version_string)
  list(GET python_include_dirs ${val} python_include_dir)
  list(GET python_executables ${val} python_executable)
  list(GET python_libraries ${val} python_library)

  set(libname JupyROOT${python_under_version_string})

  add_library(${libname} SHARED src/IOHandler.cxx)

  # libJupyROOT uses ROOT headers from binary directory
  add_dependencies(${libname} move_headers)
  target_include_directories(${libname} PRIVATE ${CMAKE_BINARY_DIR}/include)
  target_include_directories(${libname} PRIVATE ${python_include_dir})

  target_link_libraries(${libname} ${python_library} Core ${CMAKE_DL_LIBS})

  # Disables warnings originating from deprecated register keyword in Python
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" AND CMAKE_CXX_STANDARD GREATER_EQUAL 11)
    target_compile_options(${libname} PRIVATE -Wno-register)
  endif()
  if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang" AND CMAKE_CXX_STANDARD GREATER_EQUAL 11)
    target_compile_options(${libname} PRIVATE -Wno-register)
    target_compile_options(${libname} PRIVATE -Wno-deprecated-register)
  endif()

  # Install compiled .py files
  foreach(py_source ${py_sources})
    install(CODE "execute_process(COMMAND ${python_executable} -m py_compile ${d}/${py_source})")
    install(CODE "execute_process(COMMAND ${python_executable} -O -m py_compile ${d}/${py_source})")
  endforeach()

  # Install library
  install(TARGETS ${libname} EXPORT ${CMAKE_PROJECT_NAME}Exports DESTINATION ${runtimedir})

endforeach()
