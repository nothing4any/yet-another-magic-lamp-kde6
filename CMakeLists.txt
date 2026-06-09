cmake_minimum_required(VERSION 3.16)
project(kwin-effects-yaml)

set(KF_MIN_VERSION "6.0")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(ECM ${KF_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH
    ${CMAKE_MODULE_PATH}
    ${ECM_MODULE_PATH}
    ${ECM_KDE_MODULE_DIR}
    ${CMAKE_SOURCE_DIR}/cmake
    ${CMAKE_SOURCE_DIR}/cmake/Modules
)

find_package(Qt6 REQUIRED COMPONENTS
    Core
    DBus
    Gui
)

include(FeatureSummary)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)

if(NOT KDE_INSTALL_PLUGINDIR)
    set(KDE_INSTALL_PLUGINDIR "${CMAKE_INSTALL_PREFIX}/lib/qt6/plugins")
endif()
if(NOT PLUGIN_INSTALL_DIR)
    set(PLUGIN_INSTALL_DIR "${KDE_INSTALL_PLUGINDIR}")
endif()

find_package(KF6 ${KF_MIN_VERSION} REQUIRED COMPONENTS
    Config
    ConfigWidgets
    CoreAddons
    WindowSystem
)

find_package(KWin REQUIRED)

find_package(KF6KCMUtils REQUIRED)

find_package(KWinDBusInterface CONFIG REQUIRED)

find_package(epoxy REQUIRED)

add_subdirectory(src)

feature_summary(WHAT ALL)