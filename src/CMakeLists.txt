add_subdirectory(kcm)

set(effect_SRCS
    Model.cc
    YetAnotherMagicLampEffect.cc
    plugin.cc
)

kconfig_add_kcfg_files(effect_SRCS
    YetAnotherMagicLampConfig.kcfgc
)

# Tell CMake's AUTOMOC about the KWin effect factory macro, otherwise
# plugin.cc triggers a "does not contain a Q_OBJECT" warning.
list(APPEND CMAKE_AUTOMOC_MACRO_NAMES "KWIN_EFFECT_FACTORY_SUPPORTED_ENABLED")

add_library(kwin4_effect_yetanothermagiclamp SHARED ${effect_SRCS})

target_link_libraries(kwin4_effect_yetanothermagiclamp
    Qt6::Core
    Qt6::Gui
    KF6::ConfigCore
    KF6::ConfigGui
    KF6::CoreAddons
    KF6::WindowSystem
    KWin::kwin
    epoxy::epoxy
)

install(
    TARGETS
        kwin4_effect_yetanothermagiclamp

    DESTINATION
        ${PLUGIN_INSTALL_DIR}/kwin/effects/plugins/
)