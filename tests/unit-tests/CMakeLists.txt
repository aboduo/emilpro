cmake_minimum_required (VERSION 2.6)

# ====================================
# project name and version
# ====================================
project (emilpro-ut)
set (PROJECT_VERSION "1.0.0")

enable_language(ASM)

# Dependencies
set (CMAKE_MODULE_PATH  ${CMAKE_MODULE_PATH}
                        ${CMAKE_CURRENT_SOURCE_DIR}/../../cmake)
set (BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../)


find_package (LibElf REQUIRED)
find_package (LibGeoIP REQUIRED)
find_package (LibCRPCUT REQUIRED)
find_package (PkgConfig REQUIRED)
include (${CMAKE_CURRENT_SOURCE_DIR}/../../cmake/BuildBinutils.cmake)

pkg_check_modules(GTKMM gtkmm-3.0)
pkg_check_modules(LIBXMLPP libxml++-2.6)
pkg_check_modules(LIBCURL libcurl)

set (TGT ut)

set (CMAKE_CXX_FLAGS "-std=c++0x -Wall -D_GLIBCXX_USE_NANOSLEEP -DPACKAGE=emilpro -DPACKAGE_VERSION=1")
LINK_DIRECTORIES(${CMAKE_BINARY_DIR}/install-binutils/lib ${CMAKE_BINARY_DIR}/install-binutils/lib64 /home/ska/local/lib)

if (HAVE_BFD_MULTIARCH)
	set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_BFD_MULTIARCH")
endif (HAVE_BFD_MULTIARCH)

set (CMAKE_BUILD_TYPE debug)

set (${TGT}_SRCS
	../../src/addresshistory.cc
	../../src/architecturefactory.cc
	../../src/bfd-disassembly.cc
	../../src/bfd-provider.cc
	../../src/configuration.cc
	../../src/emilpro.cc
	../../src/gtk/hexview.cc
	../../src/jumptargetdisplay.cc
	../../src/namemangler.cc
	../../src/network-listener.cc
	../../src/preferences.cc
	../../src/server/html-generator.cc
	../../src/symbolfactory.cc
	../../src/ui-helpers.cc
	../../src/utils.cc
	../../src/xmlfactory.cc
	../../src/xmlstring.cc
	main.cc
	tests/tests-addresshistory.cc
	tests/tests-cgi-server.cc
	tests/tests-disassembly.cc
	tests/tests-hexview.cc
	tests/tests-html-generator.cc
	tests/tests-instructionfactory.cc
	tests/tests-jumptargetdisplay.cc
	tests/tests-model.cc
	tests/tests-namemangler.cc
	tests/tests-network-listener.cc
	tests/tests-preferences.cc
	tests/tests-server-connection.cc
	tests/tests-symbol-provider.cc
	tests/tests-ui-helpers.cc
	tests/tests-utils.cc
	tests/tests-xmlfactory.cc
	tests/tests-xmlstring.cc
	)

include_directories(
	../../src/gtk/include/
	../../src/include/
	${CMAKE_BINARY_DIR}
	${LIBELF_INCLUDE_DIRS}
	${CMAKE_BINARY_DIR}/install-binutils/include/
	${LIBCRPCUT_INCLUDE_DIRS}
	${GTKMM_INCLUDE_DIRS}
	${LIBXMLPP_INCLUDE_DIRS}
	${LIBCURL_INCLUDE_DIRS}
	${LibGeoIP_INCLUDE_DIR}
	)

add_executable (file-to-source
	../../src/utils.cc
	../../tools/file-to-source.cc
)

add_custom_command (OUTPUT built_in_instruction_models.hh
  COMMAND ${CMAKE_BINARY_DIR}/file-to-source ${CMAKE_BINARY_DIR}/built_in_instruction_models.hh built_in_instruction_models_xml
  	${CMAKE_SOURCE_DIR}/../../data/instruction-models/i386.xml 
  	${CMAKE_SOURCE_DIR}/../../data/instruction-models/mips.xml
  DEPENDS
  	${CMAKE_SOURCE_DIR}/../../data/instruction-models/i386.xml 
  	${CMAKE_SOURCE_DIR}/../../data/instruction-models/mips.xml
  	file-to-source
)

add_custom_target(emilpro_built_in_instruction_models ALL
	DEPENDS built_in_instruction_models.hh
)
set_property(SOURCE ../../src/emilpro.cc APPEND PROPERTY OBJECT_DEPENDS ${CMAKE_BINARY_DIR}/built_in_instruction_models.hh)

add_executable (${TGT} ${${TGT}_SRCS})
target_link_libraries(${TGT}
	${LIBELF_LIBRARIES}
	opcodes.a
	bfd.a
	iberty.a
	${GTKMM_LIBRARIES}
	${LIBXMLPP_LIBRARIES}
	${LIBCURL_LIBRARIES}
	${LibGeoIP_LIBRARY}
	crpcut
	gtest
	gmock
	z
	pthread
	dl
	)

set_property(TARGET ${TGT} APPEND PROPERTY OBJECT_DEPENDS binutils)


set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS)
set(CMAKE_EXE_LINK_DYNAMIC_CXX_FLAGS)
set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS) 
set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS) 
    
add_executable (test-binary
	elf-assembly.S
	elf-example-source.cc)

add_custom_target (test-binary-asm-only ALL
  COMMAND ${CMAKE_C_COMPILER} -nostdlib ${CMAKE_CURRENT_SOURCE_DIR}/elf-assembly-start.S -o ${CMAKE_BINARY_DIR}/test-binary-asm-only
)

add_custom_target (test-binary.exe ALL
  COMMAND i586-mingw32msvc-g++ ${CMAKE_CURRENT_SOURCE_DIR}/elf-example-source.cc -o ${CMAKE_BINARY_DIR}/test-binary.exe
)

add_custom_target (test-binary-asm-only-32 ALL
  COMMAND ${CMAKE_C_COMPILER} -m32 -nostdlib ${CMAKE_CURRENT_SOURCE_DIR}/elf-assembly-start.S -o ${CMAKE_BINARY_DIR}/test-binary-asm-only-32
)

add_custom_target (jump-test ALL
  COMMAND ${CMAKE_C_COMPILER} -m32 -nostdlib ${CMAKE_CURRENT_SOURCE_DIR}/jump-test-source.S -o ${CMAKE_BINARY_DIR}/jump-test
)

#add_custom_target (test-binary.mach-o ALL
#  DEPENDS test-binary
#  COMMAND objcopy -O mach-o-i386 ${CMAKE_BINARY_DIR}/test-binary ${CMAKE_BINARY_DIR}/test-binary.mach-o
#)


add_custom_target (test-binary-stripped ALL
  DEPENDS test-binary
  COMMAND strip -o ${CMAKE_BINARY_DIR}/test-binary-stripped ${CMAKE_BINARY_DIR}/test-binary
)
