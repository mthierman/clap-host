cmake_minimum_required(VERSION 3.20)
project(TestHost VERSION 0.0.1 LANGUAGES C CXX ASM)
set(CMAKE_CXX_STANDARD 17)
add_subdirectory(libs/JUCE)

math(EXPR BITNESS "${CMAKE_SIZEOF_VOID_P} * 8" OUTPUT_FORMAT DECIMAL)
message(STATUS "${PROJECT_NAME} ${BITNESS}-bit, version ${PROJECT_VERSION}, ${CMAKE_BUILD_TYPE} build")
message(STATUS "CMake version ${CMAKE_VERSION}, Compiler version ${CMAKE_CXX_COMPILER_VERSION}")

juce_add_gui_app(${PROJECT_NAME}
  PRODUCT_NAME "Test Host"
  COMPANY_NAME "Test Company"
  ICON_BIG "${CMAKE_SOURCE_DIR}/resources/images/logo.png"
  MICROPHONE_PERMISSION_ENABLED TRUE
  PLUGINHOST_AU TRUE
)

target_sources(${PROJECT_NAME}
  PRIVATE
    src/Main.cpp
    src/MainComponent.cpp)

target_compile_definitions(${PROJECT_NAME}
  PRIVATE
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_APPLICATION_NAME_STRING="$<TARGET_PROPERTY:${PROJECT_NAME},JUCE_PRODUCT_NAME>"
    JUCE_APPLICATION_VERSION_STRING="$<TARGET_PROPERTY:${PROJECT_NAME},JUCE_VERSION>"
    JUCE_DISPLAY_SPLASH_SCREEN=0)

if(DEFINED ENV{ASIOSDK_DIR} OR POWERSHELL_ASIO)
  if(POWERSHELL_ASIO)
    execute_process(COMMAND pwsh -NoProfile -Command "& {Import-Module .\\scripts\\ASIO && asio}")
    target_compile_definitions(${PROJECT_NAME} PUBLIC JUCE_ASIO=1)
    target_include_directories(${PROJECT_NAME} PUBLIC libs/asio/common)
  else()
    message(STATUS "Checking for Steinberg ASIO SDK")
    if(EXISTS "$ENV{ASIOSDK_DIR}/readme.txt")
      file(TO_CMAKE_PATH "$ENV{ASIOSDK_DIR}" ASIOSDK_DIR)
      message(STATUS "Steinberg ASIO SDK location: ${ASIOSDK_DIR}")
      message(STATUS "The resulting standalone executables are not licensed for distribution")
      target_compile_definitions(${PROJECT_NAME} PUBLIC JUCE_ASIO=1)
      target_include_directories(${PROJECT_NAME} PUBLIC ${ASIOSDK_DIR}/common)
    else()
      message(STATUS "Steinberg ASIO SDK not found")
    endif()
  endif()
endif()

target_link_libraries(${PROJECT_NAME}
  PRIVATE
    juce::juce_gui_extra
  PUBLIC
    juce::juce_recommended_config_flags
    juce::juce_recommended_lto_flags
    juce::juce_recommended_warning_flags)
