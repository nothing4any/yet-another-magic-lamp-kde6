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

include(FeatureSummary)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)

find_package(Qt6 REQUIRED COMPONENTS
    Core
    DBus
    Gui
)

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

# Determine plugin install directory. KDEInstallDirs may leave it empty
# when the install prefix matches Qt's prefix.
if("${KDE_INSTALL_PLUGINDIR}" STREQUAL "")
    get_target_property(QT_PLUGINS_DIR Qt6::Core INTERFACE_QT_PLUGINS_DIR)
    set(PLUGIN_INSTALL_DIR "${QT_PLUGINS_DIR}")
else()
    set(PLUGIN_INSTALL_DIR "${KDE_INSTALL_PLUGINDIR}")
endif()

add_subdirectory(src)

feature_summary(WHAT ALL)