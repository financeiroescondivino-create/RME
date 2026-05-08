cmake_minimum_required(VERSION 3.19)

project(rme)

if(POLICY CMP0072)
    cmake_policy(SET CMP0072 NEW)
endif()

if(POLICY CMP0167)
    cmake_policy(SET CMP0167 NEW)
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

if(MSVC)
    set(CMAKE_CXX_FLAGS                "/W3 /EHsc /D_CRT_SECURE_NO_WARNINGS")
    set(CMAKE_CXX_FLAGS_DEBUG          "/Od /Zi /D__DEBUG__")
    set(CMAKE_CXX_FLAGS_MINSIZEREL     "/O1 /DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELEASE        "/O2 /DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "/O2 /Zi")
else()
    set(CMAKE_CXX_FLAGS                "-Wall -Wextra -Wno-unused-parameter -Wno-unused-variable -Wno-deprecated-declarations -Wno-overloaded-virtual -Wno-strict-aliasing -Wno-sign-compare -Wno-unused-function -Wunused-result")
    set(CMAKE_CXX_FLAGS_DEBUG          "-O0 -g -D__DEBUG__")
    set(CMAKE_CXX_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELEASE        "-O3 -DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
endif()

find_package(OpenGL REQUIRED)
find_package(LibArchive REQUIRED)

if(WIN32)
    set(Boost_THREADAPI win32)
endif()
find_package(Boost 1.34.0 COMPONENTS thread system REQUIRED)

find_package(wxWidgets COMPONENTS html aui gl adv core net base REQUIRED)
find_package(GLUT REQUIRED)
find_package(ZLIB REQUIRED)

if(MSVC)
    find_package(lua CONFIG REQUIRED)
    find_package(sol2 CONFIG REQUIRED)
    find_package(cpr CONFIG REQUIRED)
    find_package(nlohmann_json CONFIG REQUIRED)

    if(TARGET lua::lua)
        set(LUA_LIB_TARGET lua::lua)
    elseif(TARGET lua)
        set(LUA_LIB_TARGET lua)
    else()
        message(FATAL_ERROR "Lua target not found")
    endif()

    get_target_property(LUA_INCLUDE_DIRS ${LUA_LIB_TARGET} INTERFACE_INCLUDE_DIRECTORIES)

    add_compile_definitions(
        SOL_USING_CXX_LUA_HPP=1
        SOL_LUA_VERSION=504
        SOL_LUAJIT=0
    )

    set(LIBS_TO_LINK
        ${LUA_LIB_TARGET}
        sol2::sol2
        cpr::cpr
        nlohmann_json::nlohmann_json
    )

else()
    find_package(Lua REQUIRED)

    include(FetchContent)

    FetchContent_Declare(
        sol2
        GIT_REPOSITORY https://github.com/ThePhD/sol2.git
        GIT_TAG v3.3.0
    )
    FetchContent_MakeAvailable(sol2)

    FetchContent_Declare(
        cpr
        GIT_REPOSITORY https://github.com/libcpr/cpr.git
        GIT_TAG 1.10.5
    )
    FetchContent_MakeAvailable(cpr)

    FetchContent_Declare(
        json
        GIT_REPOSITORY https://github.com/nlohmann/json.git
        GIT_TAG v3.11.3
    )
    FetchContent_MakeAvailable(json)

    set(LIBS_TO_LINK
        ${LUA_LIBRARIES}
        sol2::sol2
        cpr::cpr
        nlohmann_json::nlohmann_json
    )
endif()

include(${wxWidgets_USE_FILE})
include(source/CMakeLists.txt)

add_executable(rme ${rme_H} ${rme_SRC})

target_include_directories(rme PRIVATE
    ${CMAKE_SOURCE_DIR}/source
    ${Boost_INCLUDE_DIRS}
    ${LibArchive_INCLUDE_DIRS}
    ${OPENGL_INCLUDE_DIR}
    ${GLUT_INCLUDE_DIRS}
    ${ZLIB_INCLUDE_DIR}
    ${LUA_INCLUDE_DIR}
    ${LUA_INCLUDE_DIRS}
)

target_link_libraries(rme PRIVATE
    ${wxWidgets_LIBRARIES}
    ${Boost_LIBRARIES}
    ${LibArchive_LIBRARIES}
    ${OPENGL_LIBRARIES}
    ${GLUT_LIBRARIES}
    ${ZLIB_LIBRARIES}
    ${LIBS_TO_LINK}
)
