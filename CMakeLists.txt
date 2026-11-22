cmake_minimum_required(VERSION 3.22)
project(Aulos VERSION 0.1.4 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)

# JUCE is fetched at configure time. Bump GIT_TAG to update the framework.
FetchContent_Declare(JUCE
    GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
    GIT_TAG        9.0.5
    GIT_SHALLOW    ON)
FetchContent_MakeAvailable(JUCE)

juce_add_plugin(Aulos
    COMPANY_NAME               "duddex"
    COMPANY_WEBSITE            "https://github.com/duddex"
    PLUGIN_MANUFACTURER_CODE   Ddex
    PLUGIN_CODE                Auls
    DESCRIPTION                "Aulos - physical-model wind synth (engine by Cody Geary / CVfunk)"
    FORMATS                    VST3 Standalone
    VST3_CATEGORIES            Instrument Synth
    IS_SYNTH                   TRUE
    NEEDS_MIDI_INPUT           TRUE
    NEEDS_MIDI_OUTPUT          FALSE
    IS_MIDI_EFFECT             FALSE
    EDITOR_WANTS_KEYBOARD_FOCUS FALSE
    COPY_PLUGIN_AFTER_BUILD    FALSE
    PRODUCT_NAME               "Aulos")

target_sources(Aulos PRIVATE
    src/PluginProcessor.cpp
    src/PluginEditor.cpp
    dsp/AulosEngine.cpp)

target_include_directories(Aulos PRIVATE dsp)

target_compile_definitions(Aulos PUBLIC
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_CAN_REPLACE_VST2=0
    JUCE_DISPLAY_SPLASH_SCREEN=0)

target_link_libraries(Aulos
    PRIVATE
        juce::juce_audio_utils
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_warning_flags)

# Offline WAV render harness for the JUCE-free engine.
add_executable(render_wav
    test/render_wav.cpp
    dsp/AulosEngine.cpp)
target_include_directories(render_wav PRIVATE dsp)
if(MSVC)
    target_compile_options(render_wav PRIVATE /O2 /fp:fast)
else()
    target_compile_options(render_wav PRIVATE -O2 -ffast-math)
endif()
