# Copyright (C) 1995-2019, Rene Brun and Fons Rademakers.
# All rights reserved.
#
# For the licensing terms see $ROOTSYS/LICENSE.
# For the list of contributors see $ROOTSYS/README/CREDITS.

set(py_sources
  cppyy_backend/__init__.py
  cppyy_backend/_cling_config.py
  cppyy_backend/_cppyy_generator.py
  cppyy_backend/_genreflex.py
  cppyy_backend/_rootcling.py
  cppyy_backend/bindings_utils.py
  cppyy_backend/loader.py
  cppyy_backend/_get_cppflags.py
)

set(cppyy_backendPySrcDir cling/python/cppyy_backend)
file(COPY ${cppyy_backendPySrcDir} DESTINATION ${localruntimedir})

# Install .py source files
foreach(py_source ${py_sources})
  install(FILES cling/python/${py_source} DESTINATION ${runtimedir}/cppyy_backend)
endforeach()

foreach(val RANGE ${how_many_pythons})
  list(GET python_under_version_strings ${val} python_under_version_string)
  list(GET python_executables ${val} python_executable)

  set(libname cppyy_backend${python_under_version_string})

  add_library(${libname} SHARED clingwrapper/src/clingwrapper.cxx)
  target_include_directories(${libname} PRIVATE clingwrapper/src)
  target_link_libraries(${libname} Core ${CMAKE_DL_LIBS})

  # cppyy uses ROOT headers from binary directory
  add_dependencies(${libname} move_headers)
  target_include_directories(${libname} PRIVATE ${CMAKE_BINARY_DIR}/include)

  set_property(GLOBAL APPEND PROPERTY ROOT_EXPORTED_TARGETS ${libname})

  # Install library
  install(TARGETS ${libname} EXPORT ROOTExports DESTINATION ${CMAKE_INSTALL_LIBDIR})

  # Install compiled .py files
  foreach(py_source ${py_sources})
    install(CODE "execute_process(COMMAND ${python_executable} -m py_compile ${d}/${py_source})")
    install(CODE "execute_process(COMMAND ${python_executable} -O -m py_compile ${d}/${py_source})")
  endforeach()
endforeach()
