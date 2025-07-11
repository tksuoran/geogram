vor_reset_warning_level()

if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
  add_definitions(-D_SILENCE_STDEXT_HASH_DEPRECATION_WARNINGS)
  add_definitions(/wd4005) # libMeshb redefines _CRT_SECURE_NO_WARNINGS
  add_definitions(/wd4244) # libMeshb has int64->int conversion (calls to GmfStatKw)
else()
  add_definitions(-Wno-deprecated) # PoissonRecon uses hash map
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    add_definitions(-Wno-maybe-uninitialized)
    add_definitions(-Wno-clobbered)    # libMeshb
    add_definitions(-Wno-sign-compare) # libMeshb
  endif()
  if(
      "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR
      "${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang"
  )
    add_definitions(-Wno-unknown-warning-option)
    add_definitions(-Wno-zero-as-null-pointer-constant)
    add_definitions(-Wno-sign-compare)             # libMeshb
    add_definitions(-Wno-string-plus-int)          # lua
    add_definitions(-Wno-unused-but-set-variable)  # xatlas
    add_definitions(-Wno-deprecated-non-prototype) # triangle
  endif()
  if(ANDROID)
    add_definitions(-Wno-unused-but-set-variable)
    add_definitions(-Wno-implicit-function-declaration)
    add_definitions(-Wno-unused-comparison)
  endif()
endif()

include_directories(
    ${PROJECT_SOURCE_DIR}/src/lib/geogram/third_party/libMeshb/sources
)

# Since we are in a CMake "object library" (i.e.,
# a set of object files compiled with same options),
# the automatically generated geogram_EXPORTS
# pre-processor definition of the parent library
# is not automatically inherited (this is why
# I set it manually)
if(BUILD_SHARED_LIBS)
   add_definitions(-Dgeogram_EXPORTS)
   if(WIN32)
# For Windows, nothing to do, because
# LUA_DL_DLL is by-default activated in luaconf.h
   else()
      add_definitions(-DLUA_USE_DLOPEN)   
   endif()
endif()

aux_source_directories(SOURCES "Source Files"               .)
aux_source_directories(SOURCES "Source Files\\libMeshb"     libMeshb/sources)
aux_source_directories(SOURCES "Source Files\\rply"         rply)
aux_source_directories(SOURCES "Source Files\\shewchuk"     shewchuk)
aux_source_directories(SOURCES "Source Files\\zlib"         zlib)
aux_source_directories(SOURCES "Source Files\\PoissonRecon" PoissonRecon)
aux_source_directories(SOURCES "Source Files\\xatlas"       xatlas)

if(GEOGRAM_WITH_LUA)
   aux_source_directories(SOURCES "Source Files\\lua"          lua)
endif()

if(GEOGRAM_WITH_HLBFGS)
    aux_source_directories(SOURCES "Source Files\\HLBFGS"       HLBFGS)
endif()

if(GEOGRAM_WITH_TETGEN)
    aux_source_directories(SOURCES "Source Files\\tetgen" tetgen)
endif()

if(GEOGRAM_WITH_TRIANGLE)
    aux_source_directories(SOURCES "Source Files\\triangle" triangle)
endif()

add_library(geogram_third_party OBJECT ${SOURCES})

if(ANDROID)
  target_include_directories(geogram_third_party PRIVATE
    ${ANDROID_NDK}/sources/android/native_app_glue
  )
  message(STATUS "building for Android")
endif()  


set_target_properties(
    geogram_third_party PROPERTIES
    FOLDER "GEOGRAM"
)
