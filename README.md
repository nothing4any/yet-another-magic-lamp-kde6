# Yet Another Magic Lamp

![Screenshot](doc/Screenshot.png)

[Demo](https://www.youtube.com/watch?v=BR4bUwFZDS8)

Yet Another Magic Lamp is a window minimization effect for KWin. Whenever a window
is minimized, it gets sucked down into the dock/panel with a curvy animation.
Compared to the built-in Magic Lamp effect, this one has smoother curves,
better multi-screen panel handling, and more configuration options.

## Tested Platforms

This port targets **KWin 6 (Plasma 6)** and has been tested only on:

| Distro | KWin version | Status |
|--------|-------------|--------|
| Arch Linux / Manjaro | 6.6.5 | Working |
| Arch Linux / Manjaro | 6.7.4 / 6.7.5 | Working |
| Arch Linux / Manjaro | 6.8.2 | Working |

Other distributions or KWin versions may work but have not been verified.
The KWin 5 version is available on the `Plasma/5.23` branch of the original
[upstream repository](https://github.com/zzag/kwin-effects-yet-another-magic-lamp).

## Known Issues

- Some third-party Aurorae window decoration themes (e.g. Orchis) may show a
ghost window outline during the animation. Use the default **Breeze** theme as a
workaround.

## Dependencies

| Package | Arch Linux |
|---------|-----------|
| C++20 compiler (GCC 16+) | `gcc` |
| CMake | `cmake` |
| Extra CMake Modules | `extra-cmake-modules` |
| Qt 6 | `qt6-base` |
| KDE Frameworks 6 (Config, CoreAddons, WindowSystem, ConfigWidgets, KCMUtils) | `kconfig` `kcoreaddons` `kwindowsystem` `kconfigwidgets` `kcmutils` |
| KWin 6 development headers | `kwin` |
| libepoxy | `libepoxy` |

## Build from Source

```sh
git clone https://github.com/zzag/kwin-effects-yet-another-magic-lamp.git
cd kwin-effects-yet-another-magic-lamp

cmake -B build -S . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
cmake --build build -j$(nproc)
sudo cmake --install build
```

The effect plugin will be installed to `/usr/lib/qt6/plugins/kwin/effects/plugins/`
and the configuration module to `/usr/lib/qt6/plugins/kwin/effects/configs/`.

> **Note:** KWin embeds its version string in the Qt plugin IID of every C++
> effect plugin and refuses to load plugins with a mismatched IID. **After every
> KWin update (even a patch release) the effect must be rebuilt and reinstalled**,
> otherwise it silently stops working (KWin only logs the rejection at debug
> level, so nothing appears in the journal). A major KWin update may also
> require source changes — see the fix record below.

## Enable the Effect

Go to **System Settings → Desktop Effects**, search for "Yet Another Magic Lamp",
enable it, and click Apply.

> **Note:** This effect belongs to the "minimize" exclusive group. If the
> built-in "Magic Lamp" effect is enabled, disable it first.

## 旧插件删除
注意：删除前先查看是否存在
~~~
rm -rf /usr/share/kwin/effects/yetanothermagiclamp/
~~~


|修复记录|时间|原因|处理办法
|---|---|---|---|
|1|20260810|kwin6.7.3 升级到了 6.7.4|删除旧插件，下载最新版本构建安装
|2|20261029|KWin 升级到 6.7.5|KWin 将版本号内嵌于插件 IID，升级后旧插件被静默拒绝加载|重新编译安装即可，代码无需修改
|3|20261125|KWin 升级到 6.8.2|`Effect::paintWindow` 返回类型 `void`→`bool`，override 编译报错|修改返回类型并转发链式调用的返回值，重新编译安装（详见下文）

## KWin 6.8.2 适配记录（2026-11-25）

### 现象

KWin 升级到 6.8.2 / KDE Frameworks 6.31 后重新编译，报错：

```
error: 为 'virtual void YetAnotherMagicLampEffect::paintWindow(...)' 指定了冲突的返回类型
note: overridden function is 'virtual bool KWin::Effect::paintWindow(...)'
```

### 原因

KWin 6.8 渲染管线重构后（为处理 GPU reset 场景），所有绘制回调改为返回
`bool` 表示绘制是否成功，`Effect::paintWindow` 声明变为
`[[nodiscard]] virtual bool paintWindow(...)`（见 `/usr/include/kwin/effect/effect.h`）。

这是 6.7→6.8 对本插件唯一的破坏性 API 变更，其余用到的 API
（`apply()`/quad 网格、`EffectWindowVisibleRef`、`minimizedChanged`、
`clientArea`、`AnimationClock`、工厂宏、KCM 的 `KCModule`）均未变化。

### 修改内容

`src/YetAnotherMagicLampEffect.h` / `src/YetAnotherMagicLampEffect.cc`：
override 的返回类型 `void`→`bool`，并 `return` 链式调用的返回值：

```cpp
bool YetAnotherMagicLampEffect::paintWindow(...)
{
    return KWin::effects->paintWindow(renderTarget, viewport, w, mask, deviceRegion, data);
}
```

`src/CMakeLists.txt`：注册效果工厂宏名，消除 AutoMoc 对 `plugin.cc` 的警告
（与上游 KWin 自身构建时 `ecm_setup_qtplugin_macro_names()` 的做法一致）：

```cmake
list(APPEND CMAKE_AUTOMOC_MACRO_NAMES "KWIN_EFFECT_FACTORY_SUPPORTED_ENABLED")
```

### 编译安装命令

```sh
cmake -B build -S . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
cmake --build build -j$(nproc)
sudo cmake --install build
```

### 安装后验证

1. 校验插件内嵌的 IID 与当前 libkwin 一致（两者应打印相同的版本号）：

   ```sh
   strings /usr/lib/qt6/plugins/kwin/effects/plugins/libkwin4_effect_yetanothermagiclamp.so \
       | grep org.kde.kwin.EffectPluginFactory
   strings /usr/lib/libkwin.so.6 | grep org.kde.kwin.EffectPluginFactory
   ```

2. 让 KWin 重新加载效果（无需注销）：系统设置 → 桌面行为 → 桌面特效，
   取消勾选 "Yet Another Magic Lamp" 并应用，再重新勾选并应用。
3. 检查加载状态（返回 `true` 即成功）：

   ```sh
   qdbus org.kde.KWin /Effects org.kde.kwin.Effects.isEffectLoaded libkwin4_effect_yetanothermagiclamp
   ```