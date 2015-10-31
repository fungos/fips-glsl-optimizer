#
# project: fips-glsl-optimizer
#

if (NOT FIPS_IMPORT)
    cmake_minimum_required(VERSION 2.8)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    fips_setup()
    fips_project(fips-glsl-optimizer)
endif()

include_directories(glsl-optimizer/include)
include_directories(glsl-optimizer/src/mesa)
include_directories(glsl-optimizer/src/mapi)
include_directories(glsl-optimizer/src/glsl)
include_directories(glsl-optimizer/src)

fips_begin_lib(mesa)
    file(GLOB mesa_sources glsl-optimizer/src/mesa/program/*.c glsl-optimizer/src/mesa/main/*.c)
    fips_files(${mesa_sources})
fips_end_lib()

fips_begin_lib(glcpp_library)
    file(GLOB glcpp_sources glsl-optimizer/src/glsl/glcpp/*.c glsl-optimizer/src/util/*.c)
    include_directories(glsl-optimizer/src/glsl/glcpp)
    fips_files(${glcpp_sources})
fips_end_lib()

fips_begin_lib(glsl_optimizer)
    file(GLOB glsl_sources glsl-optimizer/src/glsl/*.cpp glsl-optimizer/src/glsl/*.c)
    file(GLOB glsl_sources_remove glsl-optimizer/src/glsl/main.cpp glsl-optimizer/src/glsl/builtin_stubs.cpp)
    list(REMOVE_ITEM glsl_sources ${glsl_sources_remove})
    fips_files(${glsl_sources})
    fips_deps(glcpp_library mesa)
fips_end_lib()

if (NOT FIPS_IMPORT)
    #
    # There is an error on mainstream: duplicated symbol for _mesa_error_no_memory
    #
    #fips_begin_app(glsl_compiler cmdline)
    #    fips_dir(glsl-optimizer/src/glsl)
    #    fips_files(main.cpp)
    #    fips_deps(glsl_optimizer)
    #fips_end_app()

    fips_begin_app(glslopt cmdline)
        file(GLOB glslopt_sources glsl-optimizer/contrib/glslopt/*.cpp)
        include_directories(glsl-optimizer/contrib/glslopt)
        fips_files(${glslopt_sources})
        fips_deps(glsl_optimizer)
    fips_end_app()

    fips_finish()
endif()

if(FIPS_MSVC)
    set_target_properties(glcpp_library PROPERTIES COMPILE_FLAGS "/wd4005 /wd4090 /wd4291")
    set_target_properties(glsl_optimizer PROPERTIES COMPILE_FLAGS "/wd4005 /wd4311 /wd4312 /wd4291 /wd4351 /wd4715 /wd4805")
endif()
