# Maintainer: Joan Figueras <ffigue at gmail dot com>
# Contributor: Jacek Szafarkiewicz <szafar@linux.pl>
# Contributor: Maxim Baz <$pkgname at maximbaz dot com>

##
## The following variables can be customized at build time. Use env or export to change at your wish
##
##   Example: env USE_SCCACHE=1 COMPONENT=1 makepkg -sc
##
## sccache for faster builds - https://github.com/brave/brave-browser/wiki/sccache-for-faster-builds
## Valid numbers between: 0 and 1
## Default is: 0 => not use sccache
if [ -z ${USE_SCCACHE+x} ]; then
  USE_SCCACHE=0
fi
##
## COMPONENT variable
## 0 -> build normal (with debug symbols)
## 1 -> release (default)
## 2 -> static
## 3 -> debug
## 4 -> release with custom cflags and system libs
## https://github.com/brave/brave-browser/wiki#clone-and-initialize-the-repo
if [ -z ${COMPONENT+x} ]; then
  COMPONENT=1
fi
##

pkgname=brave
_reponame=${pkgname}-browser
pkgver=1.16.68
pkgrel=1
pkgdesc='A web browser that stops ads and trackers by default'
arch=('x86_64')
url='https://www.brave.com/download'
license=('custom')
depends=('gtk3' 'nss' 'alsa-lib' 'libxss' 'ttf-font' 'libva')
makedepends=('git' 'npm' 'python2' 'icu' 'glibc' 'gperf' 'java-runtime-headless' 'clang' 'python2-setuptools')
optdepends=('cups: Printer support'
            'pepper-flash: Adobe Flash support'
            'libpipewire02: WebRTC desktop sharing under Wayland'
            'org.freedesktop.secrets: password storage backend on GNOME / Xfce'
            'kwallet: for storing passwords in KWallet on KDE desktops'
            'sccache: For faster builds')
chromium_base_ver="86"
patchset="6"
patchset_name="chromium-${chromium_base_ver}-patchset-${patchset}"
source=("git+https://github.com/brave/brave-browser.git#tag=v${pkgver}"
        'brave-launcher'
        'brave-browser.desktop'
        "https://github.com/stha09/chromium-patches/releases/download/${patchset_name}/${patchset_name}.tar.xz"
        'brave-custom-build.patch')
arch_revision=a702396adb03e094bfbe836ff05a451465fc986d
for Patches in \
        fix-invalid-end-iterator-usage-in-CookieMonster.patch \
        only-fall-back-to-the-i965-driver-if-we-re-on-iHD.patch \
        remove-dead-reloc-in-nonalloc-LD-flags.patch \
        check-for-enable-accelerated-video-decode-on-Linux.patch \
        xproto-fix-underflow-in-Fp1616ToDouble.patch \
        chromium-skia-harmony.patch
do
  source+=("${Patches}::https://git.archlinux.org/svntogit/packages.git/plain/trunk/${Patches}?h=packages/chromium&id=${arch_revision}")
done

# VAAPI patches from chromium-vaapi in AUR
#source+=("vdpau-support.patch::https://aur.archlinux.org/cgit/aur.git/plain/vdpau-support.patch?h=chromium-vaapi&id=7c05464a8700b1a6144258320b2b33b352385f77")

sha256sums=('SKIP'
            '725e2d0c32da4b3de2c27a02abaf2f5acca7a25dcea563ae458c537ac4ffc4d5'
            'fa6ed4341e5fc092703535b8becaa3743cb33c72f683ef450edd3ef66f70d42d'
            '6f9ab35fa2c9e6e34ec454b829b7b87adaebc10cacecd1ac1daa67035ee44aba'
            '3c0eb4433d1da4bca25f1f2bcd912986bc1eee123291395005587124a2293b9d'
            '69d8b7a439db1af4713245ddf5f44ca647283ba833a8733e848033ebdaf03cdc'
            '7514c6c81a64a5457b66494a366fbb39005563eecc48d1a39033dd06aec4e300'
            '7cace84d7494190e7882d3e637820646ec8d64808f0a2128c515bd44991a3790'
            '03d03a39b2afa40083eb8ccb9616a51619f71da92348effc8ee289cbda10128b'
            '1ec617b362bf97cce4254debd04d8396f17dec0ae1071b52ec8c1c3d86dbd322'
            '771292942c0901092a402cc60ee883877a99fb804cb54d568c8c6c94565a48e1')

# Possible replacements are listed in build/linux/unbundle/replace_gn_files.py
# Keys are the names in the above script; values are the dependencies in Arch
declare -gA _system_libs=(
  [ffmpeg]=ffmpeg
  [flac]=flac
  [fontconfig]=fontconfig
  [freetype]=freetype2
  [harfbuzz-ng]=harfbuzz
  [icu]=icu
  [libdrm]=
  [libjpeg]=libjpeg
  [libpng]=libpng
  #[libvpx]=libvpx
  [libwebp]=libwebp
  [libxml]=libxml2
  [libxslt]=libxslt
  [opus]=opus
  [re2]=re2
  [snappy]=snappy
  [zlib]=minizip
)
_unwanted_bundled_libs=(
  $(printf "%s\n" ${!_system_libs[@]} | sed 's/^libjpeg$/&_turbo/')
)

depends+=(${_system_libs[@]})

# Add depends if user wants a release with custom cflags and system libs
if [ "$COMPONENT" = "4" ]; then
    depends+=('libpulse' 'pciutils')
    makedepends+=('lld' 'libva' 'libpipewire02' 'python2-xcb-proto')
else
    makedepends+=('ncurses5-compat-libs')
fi

prepare() {
    cd "${_reponame}"

    # Hack to prioritize python2 in PATH
    mkdir -p "${srcdir}/bin"
    ln -sf /usr/bin/python2 "${srcdir}/bin/python"
    ln -sf /usr/bin/python2-config "${srcdir}/bin/python-config"
    export PATH="${srcdir}/bin:${PATH}"

    msg2 "Prepare the environment..."
    npm install
    npm run init || (npm run update_patches && npm run init)

    msg2 "Apply Chromium patches..."
    cd src/

    # https://crbug.com/893950
    sed -i -e 's/\<xmlMalloc\>/malloc/' -e 's/\<xmlFree\>/free/' \
        third_party/blink/renderer/core/xml/*.cc \
        third_party/blink/renderer/core/xml/parser/xml_document_parser.cc \
        third_party/libxml/chromium/*.cc

    # Upstream fixes
    patch -Np1 -i "${srcdir}"/fix-invalid-end-iterator-usage-in-CookieMonster.patch
    patch -Np1 -i "${srcdir}"/only-fall-back-to-the-i965-driver-if-we-re-on-iHD.patch
    patch -Np1 -i "${srcdir}"/remove-dead-reloc-in-nonalloc-LD-flags.patch
    patch -Np1 -i "${srcdir}"/check-for-enable-accelerated-video-decode-on-Linux.patch
    patch -Np1 -i "${srcdir}"/xproto-fix-underflow-in-Fp1616ToDouble.patch

    # https://crbug.com/skia/6663#c10
    patch -Np0 -i "${srcdir}"/chromium-skia-harmony.patch

    # Force script incompatible with Python 3 to use /usr/bin/python2
    sed -i '1s|python$|&2|' third_party/dom_distiller_js/protoc_plugins/*.py

    # Hacky patching
    sed -e 's/enable_distro_version_check = true/enable_distro_version_check = false/g' -i chrome/installer/linux/BUILD.gn

    # Allow building against system libraries in official builds
    if [ "$COMPONENT" = "4" ]; then
        sed -i 's/OFFICIAL_BUILD/GOOGLE_CHROME_BUILD/' \
            tools/generate_shim_headers/generate_shim_headers.py

        msg2 "Add patches for custom build"
        local _patch
        for _patch in "$srcdir/brave-custom-build.patch" "$srcdir/patches"/*.patch; do
            patch -Np1 -i "$_patch"
        done

        # Remove bundled libraries for which we will use the system copies; this
        # *should* do what the remove_bundled_libraries.py script does, with the
        # added benefit of not having to list all the remaining libraries
        local _lib
        for _lib in ${_unwanted_bundled_libs[@]}; do
            find "third_party/$_lib" -type f \
            \! -path "third_party/$_lib/chromium/*" \
            \! -path "third_party/$_lib/google/*" \
            \! -path "third_party/harfbuzz-ng/utils/hb_scoped.h" \
            \! -regex '.*\.\(gn\|gni\|isolate\)' \
            -delete
        done

        python2 build/linux/unbundle/replace_gn_files.py \
            --system-libraries "${!_system_libs[@]}"
    fi
}

build() {
    cd "${_reponame}"

    export CC=clang
    export CXX=clang++
    export AR=ar
    export NM=nm

    # Hack to prioritize python2 in PATH
    mkdir -p "${srcdir}/bin"
    ln -sf /usr/bin/python2 "${srcdir}/bin/python"
    ln -sf /usr/bin/python2-config "${srcdir}/bin/python-config"
    export PATH="${srcdir}/bin:${PATH}"

    if [ "$USE_SCCACHE" -eq "1" ]; then
        echo "sccache = /usr/bin/sccache" >> .npmrc
    fi

    npm_args=()
    if [ "$COMPONENT" = "4" ]; then
        local _flags=(
            'custom_toolchain="//build/toolchain/linux/unbundle:default"'
            'host_toolchain="//build/toolchain/linux/unbundle:default"'
            'clang_use_chrome_plugins=false'
            'treat_warnings_as_errors=false'
            'fieldtrial_testing_like_official_build=true'
            'proprietary_codecs=true'
            'rtc_use_pipewire=true'
            'link_pulseaudio=true'
            'use_gnome_keyring=false'
            'use_sysroot=false'
            'use_custom_libcxx=false'
            'use_vaapi=true'
        )

        if [[ -n ${_system_libs[icu]+set} ]]; then
            _flags+=('icu_use_data_file=false')
        fi

        if check_option strip y; then
            _flags+=('symbol_level=0')
        fi

        # Facilitate deterministic builds (taken from build/config/compiler/BUILD.gn)
        CFLAGS+='   -Wno-builtin-macro-redefined'
        CXXFLAGS+=' -Wno-builtin-macro-redefined'
        CPPFLAGS+=' -D__DATE__=  -D__TIME__=  -D__TIMESTAMP__='

        # Do not warn about unknown warning options
        CFLAGS+='   -Wno-unknown-warning-option'
        CXXFLAGS+=' -Wno-unknown-warning-option'

        npm_args+=(
            $(echo ${_flags[@]} | tr ' ' '\n' | sed -e 's/=/:/' -e 's/^/--gn=/')
        )
    fi

    ## See explanation on top to select your build
    case ${COMPONENT} in
        0)
            msg2 "Normal build (with debug)"
            npm run build
            ;;
        2)
            msg2 "Static build"
            npm run build -- Static
            ;;
        3)
            msg2 "Debug build"
            npm run build -- Debug
            ;;
        4)
            msg2 "Release custom build"
            npm run build Release -- "${npm_args[@]}"
            ;;
        *)
            msg2 "Release build"
            npm run build Release
            ;;
    esac
}

package() {
    install -d -m0755 "${pkgdir}/usr/lib/${pkgname}/"{,swiftshader}

    # Copy necessary release files
    cd "${_reponame}/src/out/Release"
    cp -a --reflink=auto \
        locales \
        resources \
        brave \
        brave_*.pak \
        chrome_*.pak \
        resources.pak \
        v8_context_snapshot.bin \
        libGLESv2.so \
        libEGL.so \
        "${pkgdir}/usr/lib/brave/"

    cp -a --reflink=auto \
        swiftshader/libGLESv2.so \
        swiftshader/libEGL.so \
        "${pkgdir}/usr/lib/brave/swiftshader/"

    if [ "$COMPONENT" != "4" ] || [[ -z ${_system_libs[icu]+set} ]]; then
        cp -a --reflink=auto \
            icudtl.dat \
            "${pkgdir}/usr/lib/brave/"
    fi

    cd "${srcdir}"
    install -Dm0755 brave-launcher "${pkgdir}/usr/bin/${pkgname}"
    install -Dm0644 -t "${pkgdir}/usr/share/applications/" "${_reponame}.desktop"
    install -Dm0644 "${_reponame}/src/brave/app/theme/brave/product_logo_128.png" "${pkgdir}/usr/share/pixmaps/${pkgname}.png"
    install -Dm0644 -t "${pkgdir}/usr/share/licenses/${pkgname}" "${_reponame}/LICENSE"
}

# vim:set ts=4 sw=4 et:
