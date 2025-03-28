cmake_minimum_required(VERSION 3.16)

project(fluentuiplugin LANGUAGES CXX)

#配置通用编译
set(CMAKE_AUTOMOC ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if(APPLE)
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64" CACHE STRING "" FORCE)
endif()

#设置QML插件输出目录->D:\Qt\6.4.3\msvc2019_64\qml\FluentUI
set(QML_PLUGIN_DIRECTORY ${CMAKE_PREFIX_PATH}/qml/FluentUI)

#设置版本号
add_definitions(-DVERSION=1,3,1,0)

find_package(Qt6 REQUIRED COMPONENTS Core Quick Qml ShaderTools)

#遍历所有Cpp文件
file(GLOB_RECURSE CPP_FILES *.cpp *.h)
foreach(filepath ${CPP_FILES})
    string(REPLACE "${CMAKE_CURRENT_SOURCE_DIR}/" "" filename ${filepath})
    list(APPEND sources_files ${filename})
endforeach(filepath)

#遍历所有qml文件
file(GLOB_RECURSE QML_PATHS *.qml)
foreach(filepath ${QML_PATHS})
    string(REPLACE "${CMAKE_CURRENT_SOURCE_DIR}/" "" filename ${filepath})
    list(APPEND qml_files ${filename})
endforeach(filepath)

#遍历所有资源文件
file(GLOB_RECURSE RES_PATHS *.png *.jpg *.svg *.ico *.ttf *.webp *.qsb *.metainfo)
foreach(filepath ${RES_PATHS})
    string(REPLACE "${CMAKE_CURRENT_SOURCE_DIR}/" "" filename ${filepath})
    list(APPEND resource_files ${filename})
endforeach(filepath)

#修改资源文件导出路径
foreach(filepath IN LISTS qml_files resource_files)
    string(REPLACE "imports/FluentUI/" "" filename ${filepath})
    set_source_files_properties(${filepath} PROPERTIES QT_RESOURCE_ALIAS ${filename})
endforeach()

#添加qml模块
qt_add_library(fluentuiplugin SHARED)
qt_add_qml_module(fluentuiplugin
    #没有下面这行代码就会生成fluentuiplugin.dll与fluentuipluginplugin.dll两个动态库
    PLUGIN_TARGET fluentuiplugin
    OUTPUT_DIRECTORY ${QML_PLUGIN_DIRECTORY}
    VERSION 1.0
    URI "FluentUI"
    SOURCES ${sources_files} fluentui.rc
    QML_FILES ${qml_files}
    RESOURCES ${resource_files}
    #支持designer
    DESIGNER_SUPPORTED
)

#链接库
target_link_libraries(fluentuiplugin PUBLIC
    Qt::CorePrivate
    Qt::QuickPrivate
    Qt::QmlPrivate
)

#链接库 win32库 不然mingw会编译错误
if(WIN32)
    target_link_libraries(fluentuiplugin PRIVATE dwmapi user32)
endif()

#如果是debug，则生成的库文件名后面拼接d
# 在MinGW和GCC/Clang中, 默认不会链接带`d`后缀的动态库
if(MSVC)
    set_target_properties(fluentuiplugin PROPERTIES DEBUG_POSTFIX "d")  
endif(MSVC)


