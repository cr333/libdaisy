CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

#README
# Compile only daisy as a library.
# do not compile the binary because third party libraries are required
# such as png zlib and jpeg

project(daisy)

IF (UNIX OR APPLE)
  include(CheckSymbolExists)
  check_symbol_exists(mmap64 "sys/mman.h" HAVE_MMAP64)
ENDIF ()

if(HAVE_MMAP64)
    message(STATUS "System has large file support.")
    add_definitions(-DHAVE_MMAP64)
else()
    message(STATUS "System hasn't large file support.")
endif()

# Detect OpenMP
FIND_PACKAGE(OpenMP) 
if (OPENMP_FOUND)
  option(USE_OPENMP "Use OpenMP for parallelization" ON)
  if (USE_OPENMP)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    add_definitions(-DUSE_OPENMP)
  endif (USE_OPENMP)
endif (OPENMP_FOUND)

OPTION(WITH_PNG "Enable PNG input/output using libpng." OFF)
if(WITH_PNG)
	find_package(PNG)
	add_definitions(-DWITH_PNG)
	include_directories(${PNG_PNG_INCLUDE_DIR} ${ZLIB_INCLUDE_DIR})
	SET(EXTRA_LIBS "${PNG_LIBRARIES};${EXTRA_LIBS}")
endif()

include_directories(include)

# Make the headers appear in IDEs.
file(GLOB daisy_hdrs include/kutility/*.h include/daisy/*.h)

set(daisy_srcs
    src/daisy.cpp
    src/corecv.cpp
    src/image_io_bmp.cpp
    src/image_io_pnm.cpp
    src/image_io_png.cpp
    src/image_io_jpeg.cpp
    src/image_manipulation.cpp
    src/progress_bar.cpp
    src/general.cpp
    src/interaction.cpp
)

add_library(daisy ${daisy_srcs} ${daisy_hdrs})
target_link_libraries(daisy ${EXTRA_LIBS})

option(DAISY_BUILD_MAIN "Build the libdaisy example application." OFF)
if(DAISY_BUILD_MAIN)
    ADD_EXECUTABLE(main_daisy src/main.cpp)
    TARGET_LINK_LIBRARIES(main_daisy daisy)
endif(DAISY_BUILD_MAIN)
