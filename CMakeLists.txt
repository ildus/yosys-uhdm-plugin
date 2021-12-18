cmake_minimum_required(VERSION 3.20)
project(uhdm-plugin)

set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CXX_EXTENSIONS ON) # Same as gnu++2a (ON) vs c++2a (OFF): https://cmake.org/cmake/help/latest/prop_tgt/CXX_EXTENSIONS.html
set (CMAKE_CXX_STANDARD_REQUIRED ON)

set(src
	uhdm-plugin/UhdmAst.cc
	uhdm-plugin/uhdmastfrontend.cc
	uhdm-plugin/uhdmsurelogastfrontend.cc
	uhdm-plugin/uhdmastreport.cc
)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
find_package(antlr4-runtime REQUIRED)
find_package(Surelog REQUIRED)

# prepare UHDM library for linking
find_path(UHDM_INCLUDE_DIRS uhdm/uhdm.h)
find_library(UHDM_LIBRARY_PATH uhdm REQUIRED)

add_library(uhdm_lib SHARED IMPORTED)
set_target_properties(uhdm_lib PROPERTIES
	IMPORTED_LOCATION "${UHDM_LIBRARY_PATH}"
	INTERFACE_INCLUDE_DIRECTORIES "${UHDM_INCLUDE_DIRS}"
)

find_program(YOSYS_CONFIG yosys-config REQUIRED)
exec_program(${YOSYS_CONFIG} ARGS --cxx OUTPUT_VARIABLE YOSYS_CXX)
exec_program(${YOSYS_CONFIG} ARGS --cxxflags OUTPUT_VARIABLE YOSYS_CXXFLAGS)
exec_program(${YOSYS_CONFIG} ARGS --ldflags OUTPUT_VARIABLE YOSYS_LDFLAGS)
exec_program(${YOSYS_CONFIG} ARGS --ldlibs OUTPUT_VARIABLE YOSYS_LDLIBS)
exec_program(${YOSYS_CONFIG} ARGS --datdir OUTPUT_VARIABLE YOSYS_DATADIR)
set(YOSYS_PLUGINS_DIR ${YOSYS_DATADIR}/plugins)

set(CMAKE_CXX_FLAGS ${YOSYS_CXXFLAGS})

add_library(uhdm SHARED ${src})
target_include_directories(uhdm PRIVATE
	/home/ildus/dev/yosys
)
target_link_libraries(uhdm PRIVATE
	flatbuffers
	capnp
	kj
	dl
	util
	m
	rt
	Threads::Threads
	antlr4_shared
	uhdm_lib
	surelog
	${YOSYS_LDFLAGS}
	${YOSYS_LDLIBS}
)
install(TARGETS uhdm DESTINATION ${YOSYS_PLUGINS_DIR})
