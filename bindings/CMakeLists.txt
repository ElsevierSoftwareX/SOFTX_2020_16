# Copyright (C) 1995-2019, Rene Brun and Fons Rademakers.
# All rights reserved.
#
# For the licensing terms see $ROOTSYS/LICENSE.
# For the list of contributors see $ROOTSYS/README/CREDITS.

if(pyroot)
  if(NOT pyroot_experimental)
    add_subdirectory(pyroot)
  else()
    # Machinery necessary to loop in parallel over multiple lists
    # This will be used in the CMakeLists of pythonizations,
    # cppyy, cppyy-backend and CPyCppyy to build and install targets
    # for multiple Python versions (only 2 and 3 for now)
    list(LENGTH python_executables len)
    math(EXPR how_many_pythons "${len} - 1")

    if(IS_ABSOLUTE ${runtimedir})
      set(d $ENV{DESTDIR}/${runtimedir})
    else()
      set(d $ENV{DESTDIR}/${CMAKE_INSTALL_PREFIX}/${runtimedir})
    endif()

    add_subdirectory(pyroot_experimental)
    add_subdirectory(tpython)
    add_subdirectory(jupyroot)
    add_subdirectory(jsmva)
  endif()
endif()

if(r)
  add_subdirectory(r)
endif()
