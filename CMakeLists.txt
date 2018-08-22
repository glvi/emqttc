# -*- mode: cmake; coding: utf-8; indent-tabs-mode: nil; -*-
project(emqttc
  VERSION 1.0.0
  DESCRIPTION "EMQTT client"
  LANGUAGES NONE)
set(emqttc_source_dir
  ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(EMQTTC_MODULES
  emqttc
  emqttc_keepalive
  emqttc_message
  emqttc_opts
  emqttc_packet
  emqttc_parser
  emqttc_protocol
  emqttc_reconnector
  emqttc_serialiser
  emqttc_socket
  emqttc_topic)
foreach(module IN LISTS EMQTTC_MODULES)
  list(APPEND EMQTTC_SOURCES    ${emqttc_source_dir}/${module}.erl)
  list(APPEND EMQTTC_OBJECTS    ${CMAKE_CURRENT_BINARY_DIR}/ebin/${module}.beam)
  list(APPEND EMQTTC_MODULES_Q '${module}')
endforeach()
file(RELATIVE_PATH emqttc_include_dir ${emqttc_source_dir} ${CMAKE_CURRENT_SOURCE_DIR}/include)
file(RELATIVE_PATH emqttc_binary_dir  ${emqttc_source_dir} ${CMAKE_CURRENT_BINARY_DIR}/ebin)
string(REPLACE ";" "," emqttc_modules "${EMQTTC_MODULES_Q}")
configure_file(Emakefile.in ${emqttc_source_dir}/Emakefile)
configure_file(${emqttc_source_dir}/emqttc.app.src ebin/emqttc.app)
add_custom_target(EmqttClient
  BYPRODUCTS ${EMQTTC_OBJECTS}
  DEPENDS    ${EMQTTC_SOURCES}
  COMMAND    ${CMAKE_COMMAND} -E make_directory ${emqttc_binary_dir}
  COMMAND    ${CMAKE_COMMAND} -E copy_directory ${emqttc_include_dir} ${CMAKE_CURRENT_BINARY_DIR}
  COMMAND    ${ERLANG_PROGRAM} -pa ${gen_logger_BINARY_DIR}/ebin -sasl errlog_type error -make
  WORKING_DIRECTORY ${emqttc_source_dir}
  VERBATIM
  COMMAND_EXPAND_LISTS)
add_dependencies(EmqttClient ErlangOTP)
set(URBANATM_EMQTTC_OBJECTS ${EMQTTC_OBJECTS} CACHE INTERNAL "UrbanATM Erlang EMQTTC objects" FORCE)
add_custom_target(DialyzeEmqttClient
  COMMAND ${DIALYZER_PROGRAM}
          --fullpath
          --plt ${DIALYZER_PLT}
          -I ${CMAKE_CURRENT_SOURCE_DIR}/include
          -pa ${gen_logger_BINARY_DIR}/ebin
          -pa ${CMAKE_CURRENT_BINARY_DIR}/ebin
          --src ${CMAKE_CURRENT_SOURCE_DIR}/src
  DEPENDS ${DIALYZER_PLT}
  )
add_dependencies(DialyzeEmqttClient
  DialyzerPlt
  EmqttClient)
