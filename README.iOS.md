
## Build Skia for macOS and iOS

```sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="${PWD}/depot_tools:${PATH}"

git clone -b aseprite-m71 https://github.com/aseprite/skia.git skia-native
cd skia-native
export SKIA_NATIVE="${PWD}"
python tools/git-sync-deps
gn gen out/Release --args="is_official_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false extra_cflags_cc=[\"-frtti\"]"
ninja -C out/Release skia
cd ..

cd skia-ios
export SKIA_IOS="${PWD}"
git clone -b aseprite-m71 https://github.com/aseprite/skia.git skia-ios
gn gen out/Release --args='is_official_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false extra_cflags_cc=["-frtti"] target_os="ios" target_cpu="x64"'
ninja -C out/Release skia
cd ..
```

## Build Aseprite for macOS and iOS

```sh
git clone -b ios --recursive https://github.com/turbolent/aseprite.git
cd aseprite
export ASEPRITE="${PWD}"
mkdir build-native
cd build-native
cmake \
  -DCMAKE_OSX_ARCHITECTURES=x86_64 \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=10.7 \
  -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk \
  -DSKIA_DIR=$SKIA_NATIVE \
  -DSKIA_LIBRARY=$SKIA_NATIVE/out/Release/libskia.a \
  -DWITH_HarfBuzz=OFF \
  -G Ninja \
  ..
ninja aseprite
cd ..

mkdir build-ios
cd build-ios
cmake \
  -DCMAKE_TOOLCHAIN_FILE=../ios.toolchain.cmake \
  -DIOS_PLATFORM=SIMULATOR64 \
  -DCMAKE_MAKE_PROGRAM=ninja \
  -DIOS_DEPLOYMENT_TARGET=11.0 \
  -GNinja \
  -DCMAKE_USE_OPENSSL=OFF \
  -DENABLE_OPENSSL=OFF \
  -DENABLE_ACL=OFF \
  -DENABLE_TEST=off \
  -DWITH_HarfBuzz=OFF \
  -DSKIA_DIR=$SKIA_IOS \
  -DSKIA_LIBRARY=$SKIA_IOS/out/Release/libskia.a \
  -DHAVE_GLIBC_STRERROR_R=off \
  -DHAVE_POSIX_STRERROR_R=on \
  -DHAVE_POLL_FINE=on \
  -DENABLE_DEVMODE=on \
  -DENABLE_SCRIPTING=off \
  -DCMAKE_USE_OPENSSL=off \
  -DBUILD_CURL_EXE=off \
  -DBUILD_CURL_TESTS=off \
  -DCURL_STATICLIB=on \
  -DENABLE_UPDATER=off \
  -DHAVE_GLIBC_STRERROR_R__TRYRUN_OUTPUT="" \
  -DHAVE_POSIX_STRERROR_R__TRYRUN_OUTPUT="" \
  -DLAF_MODP_B64_GEN=$ASEPRITE/build-native/bin/modp_b64_gen \
  -DGEN_EXE=$ASEPRITE/build-native/bin/gen \
  ..
ninja aseprite
```