# CMakeList.txt: 顶层 CMake 项目文件，在此处执行全局配置
# 并包含子项目。
#
# 设置cmake的最低版本 Qt5 需要 CMake 3.16+，Qt6 需要 3.18+
cmake_minimum_required (VERSION 3.16)

# 如果支持，请为 MSVC 编译器启用热重载。
if (POLICY CMP0141)
  cmake_policy(SET CMP0141 NEW)
  set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<IF:$<AND:$<C_COMPILER_ID:MSVC>,$<CXX_COMPILER_ID:MSVC>>,$<$<CONFIG:Debug,RelWithDebInfo>:EditAndContinue>,$<$<CONFIG:Debug,RelWithDebInfo>:ProgramDatabase>>")
endif()

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION_BUILD 0)  # 可选，用于构建版本

# 组合完整版本字符串
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}.${PROJECT_VERSION_BUILD}")

# 定义项目名称、项目版本、语言 
project (CMakeQt5 VERSION ${PROJECT_VERSION} DESCRIPTION "" LANGUAGES CXX)

# 打开全局uic，非必须
set(CMAKE_AUTOUIC ON)
# 打开全局moc,设置自动生成moc文件，一定要设置
set(CMAKE_AUTOMOC ON)
# 打开全局rcc，非必须，如需打开，注意修改相应的qrc文件名
set(CMAKE_AUTORCC ON)  

# 指定 C++ 语言标准版本       
set(CMAKE_CXX_STANDARD 17)
# 强制要求指定的标准必须被满足。如果不支持C++17，CMake 配置阶段会直接报错并终止，不会尝试降级编译
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# 设置工程包含当前目录，非必须
set(CMAKE_INCLUDE_CURRENT_DIR ON) 
# 可选：关闭编译器特定扩展，保证代码的跨平台可移植性
set(CMAKE_CXX_EXTENSIONS OFF)
#显示CMake详细 的执行过程，默认是OFF
#set(CMAKE_VERBOSE_MAKEFILE ON)
                                                                       
# 设置CMP0074策略为NEW  
cmake_policy(SET CMP0074 NEW)

# MSVC UTF-8 源码支持
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")  

# ==================== 第三方库地址 ====================
# Boost库的路径
#set(BOOST_ROOT "E:/libs/boost_1_90_0")

# ==================== 平台检测 ====================
if(WIN32)
    set(QT_INSTALL_DIR "E:/Qt/5.15.2")
    if(MSVC)
        if(CMAKE_SIZEOF_VOID_P EQUAL 4) #32位系统
            set(Qt5_ROOT "${QT_INSTALL_DIR}/msvc2019")
        elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)#64位系统
            set(Qt5_ROOT "${QT_INSTALL_DIR}/msvc2019_64")
        endif ()
    elseif(MINGW)
        if(CMAKE_SIZEOF_VOID_P EQUAL 4)
            set(Qt5_ROOT "${QT_INSTALL_DIR}/mingw81_32")
        elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)
            set(Qt5_ROOT "${QT_INSTALL_DIR}/mingw81_64")
        endif ()
    endif()     

    # 让 CMake 能找到 Qt 的 cmake 配置
    set(CMAKE_PREFIX_PATH "${Qt5_ROOT}")
endif()

# ==================== 第三方库路径（按需取消注释）====================
#qt库文件
link_directories("${Qt5_ROOT}/lib")

# 添加Boost库的头文件路径

# 添加Log4Qt库的头文件路径

# 添加Qwt库的头文件路径

# 添加QXls库
# 添加libevent库的头文件路径

#=============================项目文件 ====================#
set(APP_SOURCE_DIR "${CMAKE_SOURCE_DIR}/apps") 
#aux_source_directory("${CMAKE_SOURCE_DIR}/apps" APP_SOURCE_DIR)
set(APP_UI_DIR "${CMAKE_SOURCE_DIR}/ui")

file(GLOB APP_SOURCE_FILES "${APP_SOURCE_DIR}/*.h" "${APP_SOURCE_DIR}/*.cpp" "${CMAKE_SOURCE_DIR}/*.cpp")
file(GLOB APP_UI_FILES "${APP_UI_DIR}/*.ui")

message(${PROJECT_NAME} " build type:" ${CMAKE_BUILD_TYPE})
message("CMAKE_PREFIX_PATH:" ${CMAKE_PREFIX_PATH})
message("CXX Compiler:" ${CMAKE_CXX_COMPILER})
message("CXX Standard:" ${CMAKE_CXX_STANDARD})
message("APP_SOURCE_DIR:" ${APP_SOURCE_DIR})
message("CMAKE_CURRENT_SOURCE_DIR" ${CMAKE_CURRENT_SOURCE_DIR} )
message("CMAKE_SIZEOF_VOID_P: " ${CMAKE_SIZEOF_VOID_P})
message("CMAKE_SOURCE_DIR: " ${CMAKE_SOURCE_DIR})

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
if(QT_VERSION_MAJOR EQUAL 6)
    find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Gui Widgets)
else()
    find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets)
endif()
   
#============================= 添加翻译=========================#
set(TS_FILES
    ${CMAKE_SOURCE_DIR}/translations/zh_CN.ts
    ${CMAKE_SOURCE_DIR}/translations/en_US.ts
)


find_program(LUPDATE_EXECUTABLE lupdate)
find_program(LRELEASE_EXECUTABLE lrelease)

# -----------------------创建版本头文件模板----------------------------------------------
# 配置版本头文件
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/config.h
    @ONLY
)

# Add the forms directory to the AUTOUIC search paths
set(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_AUTOUIC_SEARCH_PATHS} ${APP_UI_DIR})

# Windows平台：设置应用程序详细信息（文件属性）
if(WIN32)
    # 生成一个包含版本信息的RC文件
    #当前应用图标
    set(APP_RESOURCE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/resources/app.rc)
    
    # 使用configure_file创建一个RC文件模板
    configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/version.rc.in
        ${APP_RESOURCE_FILE}
        @ONLY
    )
    
    # 将RC文件添加到源文件列表
    list(APPEND EXTRA_SOURCE_FILES ${APP_RESOURCE_FILE})
endif()


#qt资源文件
#set(QRC_SOURCE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/resources/main.qrc")

# 创建可执行文件
add_executable(${PROJECT_NAME} WIN32 
    ${APP_SOURCE_FILES}
    ${APP_UI_FILES}
    ${EXTRA_SOURCE_FILES}
#   ${QRC_SOURCE_FILE}
 )  
    
 # Add the target includes for MY_PROJECT 
target_include_directories(${PROJECT_NAME} 
    PRIVATE 
        ${CMAKE_SOURCE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}
        ${APP_SOURCE_DIR}
        ${APP_UI_DIR}
)

  
# 把库链接到目标执行程序上  
target_link_libraries(${PROJECT_NAME} 
    PRIVATE
	Qt${QT_VERSION_MAJOR}::Core
	Qt${QT_VERSION_MAJOR}::Gui
	Qt${QT_VERSION_MAJOR}::Widgets
)

# 为构建目标设置特定的属性(发布专业、规范的软件时，必不可少)  
# TRUE = 窗口应用程序，FALSE = 控制台应用程序
set_target_properties(${PROJECT_NAME}  PROPERTIES
    AUTOMOC ON
    AUTORCC ON
    AUTOUIC ON
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
    WIN32_EXECUTABLE TRUE
)

# ==================== 翻译更新 ====================
#foreach(_ts_file ${TS_FILES})
#    execute_process(   
#        COMMAND ${LUPDATE_EXECUTABLE} -recursive ${CMAKE_SOURCE_DIR} -ts ${_ts_file})
#    execute_process(
#        COMMAND ${LRELEASE_EXECUTABLE} ${_ts_file})
#endforeach()

add_custom_target(lupdate
    COMMAND ${LUPDATE_EXECUTABLE} -recursive ${CMAKE_SOURCE_DIR} -ts ${TS_FILES}
    COMMENT "Updating translation files..."
)

add_custom_target(lrelease
    COMMAND ${LRELEASE_EXECUTABLE} ${TS_FILES}
    DEPENDS ${TS_FILES}
    COMMENT "Releasing translation files..."
)

# ==================== INSTALL 部署配置 ====================
# 安装可执行文件
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin  # exe 放在 bin 目录
    BUNDLE DESTINATION .     # macOS 用
)


# Windows 平台：自动调用 windeployqt 部署 Qt 运行环境
if(WIN32)
    find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${Qt5_ROOT}/bin")

    if(WINDEPLOYQT_EXECUTABLE)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E echo "Deploying Qt libraries using windeployqt..."
            COMMAND ${WINDEPLOYQT_EXECUTABLE}
                --verbose 1
                --no-compiler-runtime
                --no-opengl-sw
                $<TARGET_FILE:${PROJECT_NAME}>
            COMMENT "Running windeployqt..."
        )

        # install 时也自动部署完整依赖
        install(CODE "
            execute_process(
                COMMAND ${WINDEPLOYQT_EXECUTABLE}
                    --verbose 0
                    --no-compiler-runtime
                    --no-opengl-sw
                    \${CMAKE_INSTALL_PREFIX}/bin/\$<TARGET_FILE_NAME:${PROJECT_NAME}>
            )
        ")
    endif()

endif()    
