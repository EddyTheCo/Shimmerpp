---
title: Setting the SDK
date: 2023-07-09 08:00:00 +0800
categories: [Tutorial]
tags: [qt,cmake]   
img_path: /assets/img/
image:
  path: posts/sdk/qt_components_install.png
---


This post explains the basic setup of the SDK to use the Qt/C++ component libraries of Shimmer++.
The Shimmer++ component libraries relay on [Qt6](https://www.qt.io/product/qt6/technical-specifications) and its powerful CMake support. 
To start developing Qt/C++ application for the Shimmer ecosystem one should check the current Qt support for your platform.

- [x] [Qt for Linux/X11](https://doc.qt.io/qt-6/linux.html)
- [ ] [Qt for Windows](https://doc.qt.io/qt-6/windows.html)
- [ ] [Qt for macOS](https://doc.qt.io/qt-6/macos.html) 

We will be using a Linux/X11 platform, specifically Ubuntu 22.04. 
A post written by you explaining how to develop in another platform will be welcome in this blog.


Let's dive in!

## CMake project written in C++

### Install a C++ compiler

If you want to develop in C++ you need a compiler. 
We can use Debian/Ubuntu (apt-get) GCC from the build-essential package and follow Qt [basic requirements](https://doc.qt.io/qt-6/linux.html#debian-ubuntu-apt-get). 	 
```bash
sudo apt-get install build-essential libgl1-mesa-dev
```
### Install CMake

[CMake](https://cmake.org/) is a cross-platform family of tools designed to build, test, and package software.
Exploiting Modern CMake(look it up) the development of C++ applications for different platforms becomes smooth.
The different platforms range from Desktops, web, android to  embedded devices. 

When developing for the Shimmer ecosystem using the C++ libraries, we recommend using CMake >=3.24.
This allows you to exploit the latest modules and options. 
 
To install CMake in your platform follow the [instructions](https://cmake.org/install/).
In the case of Ubuntu, cmake 3.26.4 can be installed using snap
```bash
sudo snap install cmake
```

### Install build system

One can use the preferred build system.
I normally use [Make](https://www.gnu.org/software/make/) although Qt recommends using [Ninja](https://ninja-build.org/).
 
To install Ninja
```bash
sudo apt-get install ninja-build
```

With all this, your system is ready for developing CMake projects written in C++.

## Installing Qt libraries and Qt Creator

Our final purpose is to use the basic-component libraries(BCs) for developing applications that interact with the Layer 1 of Shimmer nodes.

The BCs only dependencies are the C++ Standard Libraries and Qt libraries.
The [Qt Core](https://doc.qt.io/qt-6/qtcore-index.html) library  brings support to methods like [JSON](https://doc.qt.io/qt-6/qjsonobject.html), [Cryptographic Hashes](https://doc.qt.io/qt-6/qcryptographichash.html), [Byte Arrays, Hex Encoding](https://doc.qt.io/qt-6/qbytearray.html), and [Data Streams](https://doc.qt.io/qt-6/qdatastream.html). 
The communication with the Shimmer nodes relies on the  [Qt Network](https://doc.qt.io/qt-6/qtnetwork-index.html), [Qt Websockets](https://doc.qt.io/qt-6/qtwebsockets-index.html), and [Qt Mqtt](https://doc.qt.io/qt-6/qtmqtt-index.html) libraries. 

In that case, the necessary Qt libraries have to be installed on our system.
We recommend installing the Qt IDE and the tools to develop user interfaces provided by Qt. 
There are many ways one can install the libraries and tools, for our case we will use [Qt for Open Source Development](https://www.qt.io/download-open-source). 
And we will use the [online installer](https://www.qt.io/download-qt-installer-oss).
In future post we will explain how to install Qt using [GitHub runners]() for CI/CD and how to install Qt by using a [Yocto/OE](https://www.yoctoproject.org/) Layer.

To use the online installer one needs to have a [Qt account](https://login.qt.io/login). 

Once on the installer, the needed components are 

- [ ] Qt Creator
  + [x]	Qt Creator 11.0.0-beta2 (Preview)

- [x] Qt 6.5.0 
  + [x] Desktop gcc 64-bit 
  + [x] Qt Quick 3D
  + [x] Qt Shader Tools (optional)
  - [x] Additional Libraries
    + [x] Qt Websockets 

![QtComponents](posts/sdk/qt_components_install.png){: width="972" height="589" .w-50 .left}

The installation step creates  a `Qt` folder with the different libraries, the IDE [QtCreator](https://www.qt.io/product/development-tools), tools for developing user interfaces, and a qt-cmake tool.
```markdown
├── 6.5.0
│   ├── gcc_64
│   │   ├── bin
│   │   │   ├── qt-cmake
│   │   │   ├── qml
│   │   │   ├── ...
│   │   ├── include
│   │   │   ├── QtCore
│   │   │   ├── QtGui
│   │   │   ├── QtNetwork
│   │   │   ├── ...
│   │   ├── lib
│   │   │   ├── libQt6Core.so
│   │   │   ├── libQt6Network.so
│   │   │   ├── libQt6WebSockets.so
│   │   │   ├── ...
├── Tools
│   ├── 
│   │   ├── Qt Creator 11.0.0-beta2
│   │   │   ├──bin
│   │   │   │   ├──qtcreator
│   │   │   │   ├──...
│   │   │   ├──...
│   ├── ...
├── ...
```
## Send the simplest block to the Shimmer Testnet

The first test of your development environment will be to send a [Block](https://github.com/iotaledger/tips/blob/main/tips/TIP-0024/tip-0024.md) with a [Tagged Data Payload](https://github.com/iotaledger/tips/blob/main/tips/TIP-0023/tip-0023.md) to the Shimmer Testnet. We will use the [Qclient-IOTA](https://github.com/EddyTheCo/Qclient-IOTA) library examples for doing that.

Let's clone the repo
```bash
git clone https://github.com/EddyTheCo/Qclient-IOTA.git
``` 
Create the build directory and build from there
```bash
mkdir build
cd build
```
Configure the CMake project
```bash
~/Qt/6.5.0/gcc_64/bin/qt-cmake -G Ninja  -DUSE_THREADS=ON ../Qclient-IOTA
```
Here we are selecting Ninja as the build system but this is an optional step.

One of the parameters that compose a [block](https://github.com/iotaledger/tips/blob/main/tips/TIP-0024/tip-0024.md#serialized-layout) is the `Nonce`.
The procedure of finding the nonce is referred to as [Proof of Work](https://github.com/iotaledger/tips/blob/main/tips/TIP-0012/tip-0012.md)(PoW).
The latter is proof that intensive calculation, use of resources has been performed and is used to rate-limit the network.
Both  the clients and  the nodes can produce the PoW.

The finding of the nonce in the C++ library is performed by the [Qpow-IOTA](https://github.com/EddyTheCo/Qpow-IOTA) library.
This library is not optimized as the Rust library, and doing PoW by the client could take time and resources.
Multithreading when performing PoW by the client can be enabled by setting the CMake variable `USE_THREADS=ON`.
The PoW performed by the C++ client is slow. I encourage you to contribute to the following repos

* [Qpow-IOTA](https://github.com/EddyTheCo/Qpow-IOTA),

* [Qcurlp81](https://github.com/EddyTheCo/Qcurlp81),

to make it faster. Remember that Shimmer++ is a community library, a community that encloses also the IOTA Foundation(IF) engineers. 

The recommended configuration is that the nodes perform PoW(watch the linked video to get some secrets), and configuring Multithreading is not needed. 

Build the project
```
cmake --build . -j4
```

A folder `examples` with the different executables is created in the `build` directory.

We will run the `full_block_with_tagged_data_payload` example, the [source code](https://github.com/EddyTheCo/Qclient-IOTA/blob/main/examples/full_block_with_tagged_data_payload.cpp) is 

```cpp
#include"client/qclient.hpp"
#include <QCoreApplication>


#include <QTimer> //included only to kill the app after some time


using namespace qiota::qblocks;  // https://eddytheco.github.io/Qclient-IOTA/namespaceqiota_1_1qblocks.html
using namespace qiota;    //https://eddytheco.github.io/Qclient-IOTA/namespaceqiota.html

int main(int argc, char** argv)
{

    QCoreApplication a(argc, argv);

	
    auto iota_client=new Client(); //Create the client object.
    iota_client->set_node_address(QUrl(argv[1]));  //The node address is the first argument passed by the command line.
    //https://api.testnet.shimmer.network

    if(argc>1)iota_client->set_jwt(argv[2]); //The JSON Web Token is the second command line argument(optional) 
					     //This allows sending blocks to nodes with protected route /api/core/v2/blocks.
					     //https://wiki.iota.org/shimmer/hornet/how_tos/post_installation/#jwt-auth
					     //https://editor.swagger.io/?url=https://raw.githubusercontent.com/iotaledger/tips/main/tips/TIP-0025/core-rest-api.yaml

    const auto data_=dataF("WENN?, SOON"); //Set the data field of the payload.
    const auto tag_=tagF("from Iota-Qt");  //Set the  tag field of the payload.
    const auto payload_=Payload::Tagged_Data(tag_,data_);  //Create the Tagged_Data Payload. 

    auto block_=Block(payload_);  //Create the block with a Tagged_Data Payload inside.

    iota_client->send_block(block_); //The client sends the block to the node using the /api/core/v2/blocks route.

 
    //Print the block-id if the block is accepted by the node.
    //https://github.com/iotaledger/tips/blob/main/tips/TIP-0024/tip-0024.md#block-id
    QObject::connect(iota_client,&Client::last_blockid,&a,[&a](const c_array bid )
    {
        qDebug()<<"blockid:"<<bid.toHexString();
        a.quit();
    });
    //Kill the app after 30 seconds.
    QTimer::singleShot(30000, &a, QCoreApplication::quit);
    return a.exec();
}
```

To run the example 
```bash
./examples/full_block_with_tagged_data_payload https://api.testnet.shimmer.network
```

This could take time because the client is doing the PoW.
The public endpoint `api.testnet.shimmer.network` does not perform PoW.


The recommended setup is to use a node that performs PoW for the client.
If you maintain a node, this is easy to [configure](https://wiki.iota.org/shimmer/hornet/how_tos/using_docker/).
The linked video gives you some secrets for using a node from estervtech to do PoW. 
If the block passed the initial validation of the node, the `Block Id` is returned.



## Conclusion

The dependencies to develop using the Qt/C++ libraries are:

1. C++ compiler (gcc,g++, Xcode, MSVC 2022, MSVC 2019, MinGW 11.2)

2. Build System (Make, Ninja ...) 

3. Cross-platform build environment (CMake )

4. Version control system (Git)

5. Qt libraries (Qt Core, Qt Network ) 

We have sent a block with the most basic payload to the nodes.
Feel free to play with the other [examples](https://github.com/EddyTheCo/Qclient-IOTA/tree/main/examples).
The purpose of these posts is to create a community around the Shimmer ecosystem.
A community that shares knowledge and contributes to the development  of applications that trust the  Shimmer nodes.
Find bugs, typos, learn and teach us what you know by contributing! 
In future posts, I will explain my understanding of the different Outputs used in the [Stardust Protocol](https://blog.shimmer.network/stardust-upgrade-in-a-nutshell/) and how these can be used.
Please let me know in the comments if you find it useful.
Let me know your doubts about the Stardust protocol, the Layer 1 of Shimmer, and Shimerpp.  




