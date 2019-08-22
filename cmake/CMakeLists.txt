project(capnproto-java)

cmake_minimum_required(VERSION 3.6)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR})

include(FindCapnProto)
include_directories(${CAPNP_INCLUDE_DIRS})
link_directories(${CAPNP_LIBRARY_DIRS})
add_definitions(${CAPNP_DEFINITIONS})

get_filename_component(CAPNP_DIR ${CAPNP_EXECUTABLE} DIRECTORY)
get_filename_component(CAPNP_PREFIX ${CAPNP_DIR} DIRECTORY)
set(CAPNP_INCLUDE ${CAPNP_PREFIX}/include)

set(CMAKE_CXX_STANDARD 11)

add_executable(capnpc-java ../compiler/src/main/cpp/capnpc-java.c++)
target_link_libraries(capnpc-java ${CAPNP_LIBRARIES_LITE})

get_filename_component(ROOT ${CMAKE_CURRENT_SOURCE_DIR} DIRECTORY)

file(READ ${CAPNP_INCLUDE}/capnp/schema.capnp SCHEMA_ORIGINAL_CONTENTS)
file(MAKE_DIRECTORY ${ROOT}/runtime/src/main/schema)
file(WRITE ${ROOT}/runtime/src/main/schema/schema.capnp "using Java = import \"/capnp/java.capnp\";
$Java.package(\"org.capnproto\");
$Java.outerClassname(\"Schema\");
${SCHEMA_ORIGINAL_CONTENTS}")

if(WIN32)
    set(PATH_SEP "$<SEMICOLON>")
    set(CAPNPC_JAVA ./capnpc-java.exe)
else()
    set(PATH_SEP ":")
    set(CAPNPC_JAVA ./capnpc-java)
endif()

add_custom_target(addressbook ALL ${CMAKE_COMMAND} -E make_directory ${ROOT}/examples/target/generated-sources/capnp
                  COMMAND ${CAPNP_EXECUTABLE} compile -I${CAPNP_INCLUDE} -I${ROOT}/compiler/src/main/schema --src-prefix=${ROOT}/examples/src/main/schema -o${CAPNPC_JAVA}${PATH_SEP}${ROOT}/examples/target/generated-sources/capnp ${ROOT}/examples/src/main/schema/addressbook.capnp
                  DEPENDS capnpc-java)

add_custom_target(schema ALL ${CMAKE_COMMAND} -E make_directory ${ROOT}/runtime/target/generated-sources/capnp
                  COMMAND ${CAPNP_EXECUTABLE} compile -I${CAPNP_INCLUDE} -I${ROOT}/compiler/src/main/schema --src-prefix=${ROOT}/runtime/src/main/schema -o${CAPNPC_JAVA}${PATH_SEP}${ROOT}/runtime/target/generated-sources/capnp ${ROOT}/runtime/src/main/schema/schema.capnp
                  DEPENDS capnpc-java)

add_custom_target(benchmark ALL ${CMAKE_COMMAND} -E make_directory ${ROOT}/benchmark/target/generated-sources/capnp
                  COMMAND ${CAPNP_EXECUTABLE} compile -I${CAPNP_INCLUDE} -I${ROOT}/compiler/src/main/schema --src-prefix=${ROOT}/benchmark/src/main/schema -o${CAPNPC_JAVA}${PATH_SEP}${ROOT}/benchmark/target/generated-sources/capnp ${ROOT}/benchmark/src/main/schema/eval.capnp ${ROOT}/benchmark/src/main/schema/carsales.capnp ${ROOT}/benchmark/src/main/schema/catrank.capnp
                  DEPENDS capnpc-java)
