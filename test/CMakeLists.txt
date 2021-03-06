# CMakeLists.txt file for unit testing Archer runtime library.
include(CheckFunctionExists)
include(CheckLibraryExists)

# Some tests use math functions
check_library_exists(m sqrt "" LIBARCHER_HAVE_LIBM)
# When using libgcc, -latomic may be needed for atomics
# (but when using compiler-rt, the atomics will be built-in)
# Note: we can not check for __atomic_load because clang treats it
# as special built-in and that breaks CMake checks
check_function_exists(__atomic_load_1 LIBARCHER_HAVE_BUILTIN_ATOMIC)
if(NOT LIBARCHER_HAVE_BUILTIN_ATOMIC)
  check_library_exists(atomic __atomic_load_1 "" LIBARCHER_HAVE_LIBATOMIC)
else()
  # not needed
  set(LIBARCHER_HAVE_LIBATOMIC 0)
endif()

set(LIBARCHER_TEST_PATH ${CMAKE_CURRENT_SOURCE_DIR})
set(LIBARCHER_HAVE_ARCHER_LIBRARY ${LIBARCHER_LIB_PATH})
set(LIBARCHER_HAVE_ARCHER_RUNTIME ${LIBARCHER_RUNTIME_PATH})

macro(pythonize_bool var)
  if (${var})
    set(${var} True)
  else()
    set(${var} False)
  endif()
endmacro()

pythonize_bool(LIBARCHER_HAVE_ARCHER_LIBRARY)
pythonize_bool(LIBARCHER_HAVE_ARCHER_RUNTIME)
pythonize_bool(LIBARCHER_OMPT_SUPPORT)
pythonize_bool(LIBARCHER_OMPT_OPTIONAL)
pythonize_bool(LIBARCHER_HAVE_LIBM)
pythonize_bool(LIBARCHER_HAVE_LIBATOMIC)

add_archer_testsuite(check-libarcher "Running libarcher tests" ${CMAKE_CURRENT_BINARY_DIR} DEPENDS archer LLVMArcher)

# Configure the lit.site.cfg.in file
set(AUTO_GEN_COMMENT "## Autogenerated by libarcher configuration.\n# Do not edit!")
configure_file(lit.site.cfg.in lit.site.cfg @ONLY)
