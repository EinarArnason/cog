# libcogplatform-wl

add_library(cogplatform-wl MODULE
    cog-im-context-wl-v1.c
    cog-im-context-wl.c
    cog-platform-wl.c
    cog-popup-menu-wl.c
    os-compatibility.c
)
set_target_properties(cogplatform-wl PROPERTIES
    C_STANDARD 99
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/modules
)
target_compile_definitions(cogplatform-wl PRIVATE G_LOG_DOMAIN=\"Cog-FDO\")

find_package(WaylandProtocols REQUIRED)
add_wayland_protocol(cogplatform-wl CLIENT fullscreen-shell-unstable-v1)
add_wayland_protocol(cogplatform-wl CLIENT linux-dmabuf-unstable-v1)
add_wayland_protocol(cogplatform-wl CLIENT presentation-time)
add_wayland_protocol(cogplatform-wl CLIENT text-input-unstable-v1)
add_wayland_protocol(cogplatform-wl CLIENT text-input-unstable-v3)
add_wayland_protocol(cogplatform-wl CLIENT xdg-shell)

pkg_check_modules(Cairo IMPORTED_TARGET cairo)
pkg_check_modules(EGL IMPORTED_TARGET REQUIRED egl)
pkg_check_modules(WAYLAND IMPORTED_TARGET REQUIRED wayland-client)
pkg_check_modules(XkbCommon IMPORTED_TARGET REQUIRED xkbcommon)
find_package(WpeFDO REQUIRED)

target_link_libraries(cogplatform-wl PRIVATE
    cogcore
    PkgConfig::Cairo
    PkgConfig::EGL
    PkgConfig::WAYLAND
    PkgConfig::WebKit
    Wpe::FDO
)

pkg_check_modules(WAYLAND_EGL IMPORTED_TARGET wayland-egl)
if (TARGET PkgConfig::WAYLAND_EGL)
    target_link_libraries(cogplatform-wl PRIVATE PkgConfig::WAYLAND_EGL)
endif ()

install(TARGETS cogplatform-wl
    DESTINATION ${COG_MODULEDIR}
    COMPONENT "runtime"
)

if (COG_WESTON_DIRECT_DISPLAY)

    # Direct video display extension.
    find_path(WPEBACKEND_FDO_HAS_VIDEO_PLANE_DISPLAY_DMABUF_EXT
        NAMES wpe/extensions/video-plane-display-dmabuf.h
        PATHS ${WpeFDO_INCLUDE_DIRS}
        NO_DEFAULT_PATH
        REQUIRED
    )

    pkg_search_module(LIBWESTON REQUIRED libweston-11-protocols libweston-10-protocols libweston-9-protocols libweston-8-protocols)
    pkg_get_variable(LIBWESTON_PKG_DATA_DIR ${LIBWESTON_MODULE_NAME} "pkgdatadir")
    add_wayland_protocol(cogplatform-wl CLIENT "${LIBWESTON_PKG_DATA_DIR}/weston-direct-display.xml")
    target_compile_definitions(cogplatform-wl PRIVATE COG_ENABLE_WESTON_DIRECT_DISPLAY=1)
    if (COG_WESTON_CONTENT_PROTECTION)
        add_wayland_protocol(cogplatform-wl CLIENT "${LIBWESTON_PKG_DATA_DIR}/weston-content-protection.xml")
        target_compile_definitions(cogplatform-wl PRIVATE COG_ENABLE_WESTON_CONTENT_PROTECTION=1)
    else ()
        target_compile_definitions(cogplatform-wl PRIVATE COG_ENABLE_WESTON_CONTENT_PROTECTION=0)
    endif ()
else ()
    target_compile_definitions(cogplatform-wl PRIVATE COG_ENABLE_WESTON_DIRECT_DISPLAY=0)
    target_compile_definitions(cogplatform-wl PRIVATE COG_ENABLE_WESTON_CONTENT_PROTECTION=0)
endif ()

# Optional pointer support
pkg_check_modules(WaylandCursor IMPORTED_TARGET wayland-cursor)
if (TARGET PkgConfig::WaylandCursor)
    target_compile_definitions(cogplatform-wl PRIVATE COG_USE_WAYLAND_CURSOR)
    target_link_libraries(cogplatform-wl PRIVATE PkgConfig::WaylandCursor)
endif ()
