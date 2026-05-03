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

# 定义项目名称、项目版本、语言 
project (CMakeQt5  VERSION 0.1 LANGUAGES CXX)

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
                                                                       
# 设置CMP0074策略为NEW  
cmake_policy(SET CMP0074 NEW)

# MSVC UTF-8 源码支持
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")  

# ==================== 第三方库地址 ====================
#set(BOOST_ROOT "E:/libs/boost_1_90_0")


# ==================== 平台检测 ====================
if (WIN32 AND MSVC)
    message("MSVC")
    if(CMAKE_SIZEOF_VOID_P EQUAL 4) #32位系统
        set(Qt5_ROOT "E:/Qt/5.15.2/msvc2019")
    elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)#64位系统
        set(Qt5_ROOT "E:/Qt/5.15.2/msvc2019_64")
    endif ()
    message(STATUS "Qt5_ROOT: ${Qt5_ROOT}")
endif ()


if (WIN32 AND MINGW)
    message(STATUS "正在使用 MinGW 编译器")
    if(CMAKE_SIZEOF_VOID_P EQUAL 4)
        set(Qt5_ROOT "E:/Qt/5.15.2/mingw81_32")
    elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(Qt5_ROOT "E:/Qt/5.15.2/mingw81_64")
    else()
        message(FATAL_ERROR "无法确定 MinGW 的位数")
    endif()
endif ()


# ==================== 第三方库路径（按需取消注释）====================
#qt库文件
link_directories("${CMAKE_PREFIX_PATH}/lib")

# 添加Boost库的头文件路径

# 添加Log4Qt库的头文件路径

# 添加Qwt库的头文件路径

# 添加QXls库
# 添加libevent库的头文件路径

#=============================项目文件 ====================#
set(APP_SOURCE_DIR "${CMAKE_SOURCE_DIR}/apps") 
set(APP_UI_DIR "${CMAKE_SOURCE_DIR}/ui")

file(GLOB APP_SOURCE_FILES "${APP_SOURCE_DIR}/*.h" "${APP_SOURCE_DIR}/*.cpp" "${CMAKE_SOURCE_DIR}/*.cpp")
file(GLOB APP_UI_FILES "${APP_UI_DIR}/*.ui")

#=============================国际化=========================#
set(TS_FILES
    ${CMAKE_SOURCE_DIR}/translations/zh_CN.ts
    ${CMAKE_SOURCE_DIR}/translations/en_US.ts
)

message(${PROJECT_NAME} " build type:" ${CMAKE_BUILD_TYPE})
message("CMAKE_PREFIX_PATH:" ${CMAKE_PREFIX_PATH})
message("CXX Compiler:" ${CMAKE_CXX_COMPILER})
message("CXX Standard:" ${CMAKE_CXX_STANDARD})
message("APP_SOURCE_DIR:" ${APP_SOURCE_DIR})
message("CMAKE_CURRENT_SOURCE_DIR" ${CMAKE_CURRENT_SOURCE_DIR} )
message("CMAKE_SIZEOF_VOID_P: " ${CMAKE_SIZEOF_VOID_P})
message("CMAKE_SOURCE_DIR: " ${CMAKE_SOURCE_DIR})

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Gui Widgets)
   
# 添加翻译
find_program(LUPDATE_EXECUTABLE lupdate)
find_program(LRELEASE_EXECUTABLE lrelease)

foreach(_ts_file ${TS_FILES})
    execute_process(   
        COMMAND ${LUPDATE_EXECUTABLE} -recursive ${CMAKE_SOURCE_DIR} -ts ${_ts_file})
    execute_process(
        COMMAND ${LRELEASE_EXECUTABLE} ${_ts_file})
endforeach()
                             
#qt资源文件
#set(QRC_SOURCE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/resources/main.qrc")
  
#当前应用图标
#set(APP_RESOURCE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/resources/app.rc")

# Add the forms directory to the AUTOUIC search paths
set(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_AUTOUIC_SEARCH_PATHS} ${APP_UI_DIR})

add_executable(${PROJECT_NAME} WIN32 
    ${APP_SOURCE_FILES}
    ${APP_UI_FILES}
    ${APP_RESOURCE_FILE}
#   ${QRC_SOURCE_FILE}
 )  
    
 # Add the target includes for MY_PROJECT 
target_include_directories(${PROJECT_NAME} 
    PRIVATE 
        ${CMAKE_SOURCE_DIR}
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
    WIN32_EXECUTABLE TRUE
)

 # ==================== Deployment ====================
# Handle Windows deployment
if (WIN32 AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    set(DEBUG_SUFFIX)
    if (MSVC AND CMAKE_BUILD_TYPE MATCHES "Debug")
        set(DEBUG_SUFFIX "d")
    endif ()
    set(QT_INSTALL_PATH "${Qt5_ROOT}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()

    message("QT_INSTALL_PATH: ${QT_INSTALL_PATH}")
    if (EXISTS "${QT_INSTALL_PATH}/plugins/styles/qwindowsvistastyle${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/styles/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/styles/qwindowsvistastyle${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/styles/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/bearer/qgenericbearer${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/bearer/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/bearer/qgenericbearer${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/bearer/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/iconengines/qsvgicon${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/iconengines/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/iconengines/qsvgicon${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/iconengines/")
    endif ()
        
    #自定义添加
    if (EXISTS "${QT_INSTALL_PATH}/plugins/mediaservice/dsengine${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/mediaservice/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/mediaservice/dsengine${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/mediaservice/")
    endif ()     
    if (EXISTS "${QT_INSTALL_PATH}/plugins/mediaservice/qtmedia_audioengine${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/mediaservice/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/mediaservice/qtmedia_audioengine${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/mediaservice/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/mediaservice/wmfengine${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/mediaservice/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/mediaservice/wmfengine${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/mediaservice/")
    endif ()

    # qgif qicns qico qjpeg 
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qgif${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qgif${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qicns${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qicns${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qico${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qico${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qjpeg${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qjpeg${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()

    # qsvg qtga qtiff qwbmp qwebp
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qsvg${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qsvg${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qtga${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qtga${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qtiff${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qtiff${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qwbmp${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qwbmp${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/imageformats/qwebp${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/imageformats/qwebp${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/imageformats/")
    endif ()
       
    #数据库
    if (EXISTS "${QT_INSTALL_PATH}/plugins/sqldrivers/qsqlmysql${DEBUG_SUFFIX}.dll")   
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/sqldrivers/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/sqldrivers/qsqlmysql${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/sqldrivers/")
    endif () 
          
    foreach (QT_LIB Core Gui Widgets)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt5${QT_LIB}${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)
endif ()
  
