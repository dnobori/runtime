cmake_minimum_required(VERSION 3.14.5)

project(mono-wasm-runtime C)

add_executable(dotnet corebindings.c driver.c pinvoke.c)

target_include_directories(dotnet PUBLIC ${MONO_INCLUDES} ${MONO_OBJ_INCLUDES})
target_compile_options(dotnet PUBLIC @${NATIVE_BIN_DIR}/src/emcc-default.rsp ${CONFIGURATION_EMCC_FLAGS} -DCORE_BINDINGS -DGEN_PINVOKE=1)
target_link_libraries(dotnet
    ${ICU_LIB_DIR}/libicuuc.a
    ${ICU_LIB_DIR}/libicui18n.a
    ${MONO_ARTIFACTS_DIR}/libmono-component-hot_reload-static.a
    ${MONO_ARTIFACTS_DIR}/libmono-component-debugger-static.a
    ${MONO_ARTIFACTS_DIR}/libmono-component-diagnostics_tracing-stub-static.a
    ${MONO_ARTIFACTS_DIR}/libmono-ee-interp.a
    ${MONO_ARTIFACTS_DIR}/libmonosgen-2.0.a
    ${MONO_ARTIFACTS_DIR}/libmono-ilgen.a
    ${MONO_ARTIFACTS_DIR}/libmono-icall-table.a
    ${NATIVE_BIN_DIR}/libSystem.Native.a
    ${NATIVE_BIN_DIR}/libSystem.IO.Compression.Native.a)

set_target_properties(dotnet PROPERTIES
    CMAKE_EXECUTABLE_SUFFIX ".js"
    LINK_DEPENDS "${NATIVE_BIN_DIR}/src/emcc-default.rsp;${NATIVE_BIN_DIR}/src/runtime.iffe.js;${SOURCE_DIR}/library_mono.js;${SOURCE_DIR}/binding_support.js;${SOURCE_DIR}/dotnet_support.js;${SYSTEM_NATIVE_DIR}/pal_random.js"
    LINK_FLAGS "@${NATIVE_BIN_DIR}/src/emcc-default.rsp ${CONFIGURATION_EMCC_FLAGS} -DENABLE_NETCORE=1 --pre-js ${NATIVE_BIN_DIR}/src/runtime.iffe.js --js-library ${SOURCE_DIR}/library_mono.js --js-library ${SOURCE_DIR}/binding_support.js --js-library ${SOURCE_DIR}/dotnet_support.js --js-library ${SYSTEM_NATIVE_DIR}/pal_random.js"
    RUNTIME_OUTPUT_DIRECTORY "${NATIVE_BIN_DIR}")

add_custom_command(TARGET dotnet POST_BUILD COMMAND ${EMSDK_PATH}/upstream/bin/wasm-opt --strip-dwarf ${NATIVE_BIN_DIR}/dotnet.wasm -o ${NATIVE_BIN_DIR}/dotnet.wasm)
