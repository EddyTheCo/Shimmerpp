---
title:  Qt/QML application for Shimmer
date: 2024-01-04 08:00:00 +0800
categories: [Tutorial, Showcase]
tags: [QML,NFT]   
img_path: /Shimmerpp/assets/img/
image:
  path: posts/gui/platformsGuiApp.png
---

This post explains how to build an application that uses the [Shimmer++ libraries](https://eddytheco.github.io/Shimmerpp/about/) for different platforms.
Specifically, the post will pinpoint the important parts needed to build the application for major Desktop platforms([Windows](https://doc.qt.io/qt-6/windows.html), [Linux](https://doc.qt.io/qt-6/linux.html), [macOS](https://doc.qt.io/qt-6/macos.html)),
 [browsers](https://doc.qt.io/qt-6/wasm.html), 
[Android](https://doc.qt.io/qt-6/android.html), and
[embedded](https://doc.qt.io/qt-6/embedded-linux.html) devices.

The [code repo](https://github.com/EddyTheCo/NftMinter/tree/v0.5.0) contains the application source code and will be the main reference of this post.
A [Yocto](https://www.yoctoproject.org/) Layer [repo](https://github.com/EddyTheCo/meta-evt) shows how to use the Shimmer++ libraries in embedded devices.
Developers are encouraged to check the repos for a more complete understanding of this post.

The linked video showcases the application running on different platforms and explains the logic behind NFTs working as wallets.  

## The Qt/QML/Shimmer++ application

In previous posts and videos, we have seen how to [build a console application](https://eddytheco.github.io/Shimmerpp/posts/Creating_Outputs/) and how to [create reusable GUI components](https://eddytheco.github.io/Shimmerpp/posts/Extending_QML/) using the Shimmer++ libraries. 
With this knowledge, we are ready to create a full application using the Qt/C++ Shimmer libraries and some custom QML modules that will make the development process smooth.

This time we will build a Dapp that mints NFTs on Shimmer networks with Stardust protocol.

The application has to 

1. Communicate directly with the Shimmer nodes(Layer 1).

2. Keep track of the NFTs  and base tokens owned by certain account 

3. Create new NFTs and allow to change ownership of the NFTs, set the issuer, and burn it.

4. Allow to transfer ownership of all the assets on the account.

5. Be open-source and executed locally by the user. 

For simplicity, our application will be meant to be used and thrown.
We do not want users inserting their private keys into the application.
For that, the application creates random keys that constitute an account.
The user funds the account, uses the application, and then clears the account using point 4.


The points 1 and 5 are important for the application to be considered a [Dapp](https://vitalik.eth.limo/general/2023/12/28/cypherpunk.html).
Every user will have an application that will continue working as long the Layer 1 is up and running.
Even if the main developers disappear the application could be modified to introduce the new changes in the protocol.

 
To fulfill  point 1 we will use the [Connection Settings](https://github.com/EddyTheCo/ConectionSettings) library.
This component uses the Shimmer++ libraries to communicate with the nodes using the [Core REST](https://wiki.iota.org/tips/tips/TIP-0025/), [UTXO Indexer](https://wiki.iota.org/tips/tips/TIP-0026/), and [EVENT](https://wiki.iota.org/tips/tips/TIP-0028/) API. 
The library also exposes a [QML module](https://eddytheco.github.io/qmlonline/?example_url=iotaconnection) that allows to change the settings of the connection with the nodes 

The application has to create/recover an account(a set of key pairs).
In that case, we will use another [QML module](https://eddytheco.github.io/qmlonline/?example_url=iotaaccount) and C++ library present on the [IotaAccount repo](https://github.com/EddyTheCo/account).
This library takes care of the creation of deterministic addresses from a seed. 
The seed can be given as a binary seed or by Mnemonic sentences following [Bip-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki).

And finally, to fulfill all the requirements of the asset management for a certain account we will rely on the Shimmer++ [Wallet library](https://github.com/EddyTheCo/qWallet-IOTA).
 
### Important parts of the code

As always we start with the [`CMakeLists.txt`](https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt) because we love `CMake`.
There we have
```cmake
set(USE_QML ON)
```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt'}

This variable is used by some libraries like the Connection-Settings one, to create or not the corresponding QML module. 

The dependencies of the application are declared like 
```cmake
find_package(Qt6 REQUIRED COMPONENTS Core Gui Quick Qml OPTIONAL_COMPONENTS Multimedia)

FetchContent_Declare(
    IotaWallet
    GIT_REPOSITORY https://github.com/EddyTheCo/qWallet-IOTA.git
    GIT_TAG v0.1.0
    FIND_PACKAGE_ARGS 0.1 CONFIG
)
FetchContent_MakeAvailable(IotaWallet)

FetchContent_Declare(
    qrCode
    GIT_REPOSITORY https://github.com/EddyTheCo/qrCode.git
    GIT_TAG  v1.0.2
    FIND_PACKAGE_ARGS 1.0 CONFIG
)
FetchContent_MakeAvailable(qrCode)

FetchContent_Declare(
    DTPickersQML
    GIT_REPOSITORY https://github.com/EddyTheCo/DateTimePickers.git
    GIT_TAG v0.1.2
    FIND_PACKAGE_ARGS 0.1 CONFIG
)
FetchContent_MakeAvailable(DTPickersQML)
```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt'}

The main dependency is the IotaWallet library which also exposes the Account and Connection-Settings methods. 

When creating a `QGuiApplication` that uses QML, we need an application target and associate a QML module to it like 

```cmake
qt_add_executable(nftminter main.cpp)


qt6_add_qml_module(nftminter
    URI  Esterv.Iota.NFTMinter
    VERSION 1.0
    SOURCES
    src/CreateNFT.cpp include/CreateNFT.hpp
    QML_FILES
    "qml/SendDialog.qml"
    "qml/BoxNFT.qml"
    "qml/BoxMenu.qml"
    "qml/ConfDrawer.qml"
    "qml/window.qml"
    RESOURCE_PREFIX
    "/esterVtech.com/imports"
    RESOURCES
    "fonts/Anton/Anton-Regular.ttf"
    "fonts/Permanent_Marker/PermanentMarker-Regular.ttf"
    IMPORT_PATH ${CMAKE_BINARY_DIR}
)
```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt'}

The QML module has several types like `SendDialog`, and `BoxNFT`  defined on the QML files `qml/SendDialog.qml` and `qml/BoxNFT.qml`.
The C++ SOURCES also defines some custom QML types.  


If the application will be build also for Android devices one should add
```cmake
if(ANDROID)
    set_property(TARGET nftminter PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    set_property(TARGET nftminter APPEND PROPERTY QT_ANDROID_MIN_SDK_VERSION 30)
    set_property(TARGET nftminter APPEND PROPERTY QT_ANDROID_TARGET_SDK_VERSION 34)
    set_property(TARGET nftminter APPEND PROPERTY QT_ANDROID_SDK_BUILD_TOOLS_REVISION 34.0.0)

    FetchContent_Declare(
        android_openssl
        DOWNLOAD_EXTRACT_TIMESTAMP true
        URL      https://github.com/KDAB/android_openssl/archive/refs/heads/master.zip
    )
FetchContent_GetProperties(android_openssl)
if(NOT android_openssl_POPULATED)
    FetchContent_Populate(android_openssl)
    include(${android_openssl_SOURCE_DIR}/android_openssl.cmake)
    add_android_openssl_libraries(nftminter)
endif(NOT android_openssl_POPULATED)

endif(ANDROID)
```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt'}

The latter sets some [cmake variables used by Qt](https://doc.qt.io/qt-6/android-build-environment-variables.html) and adds [openssl support for android](https://doc.qt.io/qt-6/android-openssl-support.html) in case is needed.  


To ship the Qt libraries with your application one normally uses the command line tools, named windeployqt, macdeployqt, and androiddeployqt.
But we can also use [Qt's CMake deployment API](https://www.qt.io/blog/cmake-deployment-api) and make the process more smooth.
To use this API the project has these CMake commands

```cmake
set_target_properties(nftminter PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)

...


install(TARGETS nftminter
    BUNDLE  DESTINATION .
    DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
if(QTDEPLOY)
    qt_generate_deploy_qml_app_script(
        TARGET nftminter
        OUTPUT_SCRIPT deploy_script
    )
install(SCRIPT ${deploy_script})
endif(QTDEPLOY)
```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt'}

At the moment Qt's CMake deployment API is available for the 3 major Desktop platforms,
the project limits its use by the `QTDEPLOY` variable.


The last part 

```cmake
if(EMSCRIPTEN)
target_compile_definitions(nftminter PRIVATE USE_EMSCRIPTEN)
    add_custom_command(
        TARGET nftminter
        POST_BUILD
        COMMAND ${CMAKE_COMMAND}
        ARGS -E copy "${CMAKE_CURRENT_BINARY_DIR}/nftminter.js" "${CMAKE_CURRENT_BINARY_DIR}/nftminter.wasm" "${CMAKE_CURRENT_BINARY_DIR}/qtloader.js" "${CMAKE_CURRENT_SOURCE_DIR}/wasm"
    )
endif()
```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/CMakeLists.txt'}

It is related to setting up our application for browsers.
It simply copies the products of building the application with emscripten to the source directory.


There is no need to revisit the QML Module because we have explained that in the previous [post](https://eddytheco.github.io/Shimmerpp/posts/Extending_QML/).

The only new part is the `main.cpp`

```cpp
int main(int argc, char *argv[])
{

    QGuiApplication app(argc, argv);
#if defined(FORCE_STYLE)
    QQuickStyle::setStyle(FORCE_STYLE);
#endif

	QQmlApplicationEngine engine;
    engine.addImageProvider(QLatin1String("qrcode"), new QRImageProvider(1));
    engine.addImageProvider(QLatin1String("wasm"), new WasmImageProvider());
    engine.addImportPath("qrc:/esterVtech.com/imports");

    const QUrl url(u"qrc:/esterVtech.com/imports/Esterv/Iota/NFTMinter/qml/window.qml"_qs);
	engine.load(url);

	return app.exec();
}

```
{: file='https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/main.cpp'}

Here `GuiApplication` and a QML engine are created. 
The main event loop starts by loading the qml file `qml/window.qml` to the engine.
  

## Conclusions

We have created a GUI application that mints NFTs on the testnet of Shimmer.
The [repo](https://github.com/EddyTheCo/NftMinter/tree/v0.5.0) shows how to build the app for the different platforms.
One should take a  look at the [GitHub action](https://github.com/EddyTheCo/NftMinter/blob/v0.5.0/.github/workflows/build-test-install.yml) that creates the releases for major desktop platforms and Android devices. 
And also the recipes to build CMake applications in the Yocto Layer repo.

In the next post we will use NFTs as tickets!
We will showcase an IoT application that allows you to book a locker and use it by owning an NFT.

The purpose of these posts is to create a community around the Shimmer ecosystem. A community that shares knowledge and contributes to the development of applications that trust the Shimmer nodes. Find bugs, and typos, learn and teach us what you know by contributing!


Please let me know in the comments if you find it useful. Let me know your doubts about the Stardust protocol, the Layer 1 of Shimmer, and Shimmer++.


## Watch the video! 
{% include embed/youtube.html id='UK_493BTI1M' %}
