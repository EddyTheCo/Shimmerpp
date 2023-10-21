---
title: Creating a GUI component 
date: 2023-10-10 08:00:00 +0800
categories: [Tutorial]
tags: [QML,NFT]   
img_path: /assets/img/
image:
  path: posts/utxo/UTXO_model_in_Shimmer.png
---

> This post and its resources are still in development.
{: .prompt-info }

This post explains how to extend the QML language using C++, specifically the [Shimmer++ libraries](https://eddytheco.github.io/Shimmerpp/about/).
After reading the post, developers will be able to create reusable GUI components, making the development faster.

The [code repo](https://github.com/EddyTheCo/NFTQMLShimmerpp/tree/v0.1.1) serves as an example application of this post. 
The repo creates a library that defines custom QML types defined in C++ and QML, and use that library in a [simple NFT game](https://eddytheco.github.io/NFTQMLShimmerpp/).

The linked video shows the benefits of using a declarative language like QML and speculates about the use of this in games, wallet applications, etc. The video also shows how to use  the library in your application.
Make sure you watch the video, so you  always win in our simple NFT game.  


## Why use QML?

QML is a declarative language, designed for creating user interfaces(UI) thinking in speed and easier reading for developers.
It provides an API to enable application developers to extend the QML language with custom types and integrate QML code with JavaScript and C++. 

The QML types can be used in cross-platform applications that run on various software and hardware platforms with little or no change in the underlying codebase while still being a native application with native capabilities and speed.

There is also a large amount of [documentation](https://doc.qt.io/qt-6/qtqml-index.html) on UI creation using QML language and a big [community](https://forum.qt.io/category/12/qml-and-qt-quick) behind Qt that likes and uses QML. 

One can see QML in action in these online applications 

* [open-meteo](https://eddytheco.github.io/qmlonline/?example_url=omclient), [QR decoding](https://eddytheco.github.io/qmlonline/?example_url=qt_qr_dec), [QR encoding](https://eddytheco.github.io/qmlonline/?example_url=qt_qr_gen)

* [QML Online](https://qmlonline.kde.org/)

In this post, we are more interested in the [QML-C++ integration](https://doc.qt.io/qt-6/qtqml-cppintegration-overview.html).
The latter will allow us  not only to use the Shimmer++ libraries in [console applications](https://eddytheco.github.io/Shimmerpp/posts/Creating_Outputs/) but also to create libraries that expose some GUI components.
The component could be for example a user avatar, a user avatar that depends on the NFTs the user owns 
and can be reused in other Dapps like games, social networking, wallets, etc. 


## The example GUI library

As an example, we are interested in developing a library that

1. Can be easily integrated into any Qt/QML GUI application through the use of CMake Packages. 

2. Monitors the last [`NFT Output`](https://wiki.iota.org/tips/tips/TIP-0018/#nft-output) that has a certain address value in the [`Adress Unlock Condition`](https://wiki.iota.org/tips/tips/TIP-0018/#address-unlock-condition).

3. Analyzes the `Immutable Metadata` and `Metadata` [Features](https://wiki.iota.org/tips/tips/TIP-0018/#metadata-feature) of the `Output`. 

4. Displays if present a name, an image, and a unique attack coefficient taken from the `NFT Output`.


### Building a reusable QML module and CMake packages

The part of our code that takes care of creating a [reusable QML module](https://doc.qt.io/qt-6/cmake-build-reusable-qml-module.html) is mainly present in the [`CMakeLists.txt`](https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt) file.
The most important part is the use of the [`qt_add_qml_module`](https://doc.qt.io/qt-6/qt-add-qml-module.html) function.

```cmake
qt6_add_qml_module(nftMonitor
	URI  NFTMonitor
	VERSION 1.0
	SOURCES
	src/nftmonitor.cpp include/nftmonitor.hpp
	OUTPUT_TARGETS out_targets_var
	QML_FILES
	qml/NFTMonitor.qml
	RESOURCE_PREFIX
	"/esterVtech.com/imports"
	RESOURCES
	"images/BestNFT.png"
	OUTPUT_DIRECTORY
	${CMAKE_BINARY_DIR}/NFTMonitor
	)
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}
This function will create [two targets](https://doc.qt.io/qt-6/qt-add-qml-module.html#separate-backing-and-plugin-targets), a backing target called `nftMonitor` and a plugin target called `nftMonitorplugin`.
The targets exposes a [QML Module](https://doc.qt.io/qt-6/qtqml-modules-topic.html) called `NFTMonitor`.
This module provides two custom QML types, one uses C++ and is defined in the `SOURCES` while the other one is [defined in a QML document](https://doc.qt.io/qt-6/qtqml-documents-definetypes.html) in the `QML_FILES` parameter.

The definition of the `RESOURCE_PREFIX` is made to avoid name clashes between different modules when imported.
This resource prefix has to be added to the [QML Import Path](https://doc.qt.io/qt-6/qtqml-syntax-imports.html#qml-import-path) for the QML engine to be able to find the module. 
By using this path, the application that links to our library can rely on [the Qt Resource System](https://doc.qt.io/qt-6/resources.html) for using QML modules and resources in a platform-independent manner.

Setting the `OUTPUT_DIRECTORY` variable will allow us to control where the plugin library, qmldir and typeinfo files are generated.
The latter variable is important for tooling and using the plugin target at runtime.
Even if your application explicitly links to your backing target, the linker could see the module as unused by the application.
It is generally hard to guarantee that a linker preserves the linkage to a library it considers unused.
In that case, the plugin target  is used at runtime to load the module dynamically.
Setup tooling like [qmllint](https://doc.qt.io/qt-6/qtquick-tool-qmllint.html) for our library is out of the scope of this post.

> With this, we have defined two CMake targets that exposes some custom QML types in a library that can be built.
{: .prompt-info }

To use these targets one has to make it available to the CMake configuration of the project using our library.
For this one can use the CMake [add_subdirectory](https://cmake.org/cmake/help/latest/command/add_subdirectory.html) or [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html) function.
This will make available our library at configuration time and build it.

To make available our library once is compiled is better to use the CMake [`install`](https://cmake.org/cmake/help/latest/command/install.html) function.
This will group the needed files and resources in the `installation tree`.

Configure CMake to install the library and put the targets in an export set

```cmake
install(TARGETS nftMonitor nftMonitorplugin ${out_targets_var} 
	EXPORT nftMonitorTargets
	DESTINATION ${CMAKE_INSTALL_LIBDIR}
	)
``` 
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}
The plugin target is only need it if the module is statically built, so that , targets that import such STATIC QML modules also need to explicitly link to corresponding QML plugins.

Install also the public headers and the compiled plugin folder

```cmake
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/nftMonitor
	)
install(DIRECTORY ${CMAKE_BINARY_DIR}/NFTMonitor/
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/QMLPlugins/NFTMonitor
	)
``` 
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}
The compiled plugin folder should be use to load the module dynamically and with tooling.

One also needs to install a CMake file containing code to import the targets from the installation tree into another project, like
```cmake
install(EXPORT nftMonitorTargets
	FILE nftMonitorTargets.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/nftMonitor
	)
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}
To use the powerful [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html) to search for our library one can use the [`CMakePackageConfigHelpers`](https://cmake.org/cmake/help/latest/module/CMakePackageConfigHelpers.html) functions
```cmake
include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/nftMonitorConfig.cmake"
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/nftMonitor
	)
write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/nftMonitorConfigVersion.cmake"
	VERSION 1.0
	COMPATIBILITY SameMajorVersion
	)
install(FILES
	${CMAKE_CURRENT_BINARY_DIR}/nftMonitorConfig.cmake
	${CMAKE_CURRENT_BINARY_DIR}/nftMonitorConfigVersion.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/nftMonitor
	)
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}
> Now we have a library that can be easily reused in other applications by using CMake.
 The project install the backing and plugin target for explicit linking, the plugin folder for dynamically loading the module,
and the headers for using the custom QML type from C++.
{: .prompt-info }

### Using Shimmer++ in the exposed QML type.

To monitor the NFT outputs linked to a certain address we need to use the Shimmer++ libraries.
We will need to use the route `/api/indexer/v1/outputs/nft` of the [REST API](https://wiki.iota.org/tips/tips/TIP-0026/) of the nodes.
The [Event API](https://wiki.iota.org/tips/tips/TIP-0028/) of the nodes will let us know if a new `NFT Output` is created referencing the address we are monitoring by subscribing to `outputs/unlock/{condition}/{address}` topic.
The latter will update  the exposed properties of our QML type in real-time.

To make available the methods to communicate with the REST API of the nodes we use the [Qclient](https://eddytheco.github.io/Qclient-IOTA/) library.
The communication with the Event API is performed by the [QclientMqtt](https://eddytheco.github.io/QclientMqtt-IOTA/) library.

One can easily integrate these libraries into your CMake project like
```cmake
include(FetchContent)
# Get the library that check if a NFT exist on an address using the REST API of the nodes
# This will check if the library is installed first, if not download it from GitHub and compile it
FetchContent_Declare(
	qclient
	GIT_REPOSITORY https://github.com/EddyTheCo/Qclient-IOTA.git
	GIT_TAG v0.2.3
	FIND_PACKAGE_ARGS 0.2 CONFIG
	)
FetchContent_MakeAvailable(qclient)

# Get the library that notifies when a new nft is transferred to an address using the event API of the nodes
# https://wiki.iota.org/shimmer/tips/tips/TIP-0028/
FetchContent_Declare(
	qclientMQTT
	GIT_REPOSITORY https://github.com/EddyTheCo/QclientMqtt-IOTA.git
	GIT_TAG v0.2.3
	FIND_PACKAGE_ARGS 0.2 CONFIG
	)
FetchContent_MakeAvailable(qclientMQTT)
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}
Because we will be working with addresses in human-readable format our QML type needs the [Bech32 methods](https://eddytheco.github.io/Qbech32/) like
```cmake
# Get the library that takes care of encoding and decoding bech32 addresses
FetchContent_Declare(
	qbech32
	GIT_REPOSITORY https://github.com/EddyTheCo/Qbech32.git
	GIT_TAG v0.0.2
	FIND_PACKAGE_ARGS 0.0 CONFIG
	)
FetchContent_MakeAvailable(qbech32)
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/CMakeLists.txt'}

In the `SOURCES` of our QML type, we will use the methods from Shimmer++ to monitor the NFTs in a certain address.
To query the REST API of the node using the `/api/indexer/v1/outputs/nft` route we use
```cpp
// When the node returns the NFT outputs execute this
connect(node_outputs_,&Node_outputs::finished,this,[=]( ){
	// If there are NFT outputs on the address update the values of the QML type
	if(node_outputs_->outs_.size())updateValues(node_outputs_->outs_.front());
	node_outputs_->deleteLater();
	});
...
rest_client->get_outputs<qblocks::Output::NFT_typ>(node_outputs_,"address="+m_address);
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/src/nftmonitor.cpp'}
To subscribe to the `outputs/unlock/{condition}/{address}` we do 

```cpp
resp=event_client->get_subscription("outputs/unlock/address/"+m_address);
connect(resp,&ResponseMqtt::returned,this,[=](auto data){
	//Update the values of the QML type.
	updateValues(Node_output(data));
	});
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/src/nftmonitor.cpp'}
If any of these methods return an `OUTPUT`, the program proceeds to analyze it and update the properties of our QML type.

To calculate the attack coefficient we use the `NFT ID` of the `NFT Output` like
```cpp
auto NFTID=nft_output->get_id();
auto buffer=QDataStream(&NFTID,QIODevice::ReadOnly);
buffer.setByteOrder(QDataStream::LittleEndian);
quint64 atackInd;
buffer>>atackInd;
m_attack=atackInd*1000.0/std::numeric_limits<quint64>::max();
emit attackChanged();
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/src/nftmonitor.cpp'}
With this, a semi-random coefficient between 0 and 1000 is linked to every `NFT Output`, and the property attack of our QML type is updated.
This allows the library to exploit content from other creators while not letting them abuse the game.


With the next code snippet, the function analyzes the metadata feature and updates the name property of the QML type.

```cpp
auto metdataFeat=nft_output->get_feature_(qblocks::Feature::Metadata_typ);
		if(metdataFeat)
		{
			auto metdata=std::static_pointer_cast<const qblocks::Metadata_Feature>(metdataFeat);
			auto data=metdata->data();

			QJsonDocument document = QJsonDocument::fromJson(metdata->data());
			if(!document.isNull())
			{
				QJsonObject rootObj = document.object();
				if(!rootObj["name"].isUndefined()&&rootObj["name"].isString())
				{
					m_name=rootObj["name"].toString();
					emit nameChanged();
				}
			}

		}
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/src/nftmonitor.cpp'}
This allows you to link the address and the NFT to your name(because you like to brag about it).
More complicated applications can be done that allow you to show ownership of the asset, but this is enough for this post.

We do the same to update the image parameter of the QML type from the `Uri` field of the `Immutable Metadata Feature`
```cpp
auto ImmetdataFeat=nft_output->get_immutable_feature_(qblocks::Feature::Metadata_typ);
if(ImmetdataFeat)
{
	auto metdata=std::static_pointer_cast<const qblocks::Metadata_Feature>(ImmetdataFeat);
	auto data=metdata->data();
	QJsonDocument document = QJsonDocument::fromJson(metdata->data());
	if(!document.isNull())
	{
		QJsonObject rootObj = document.object();
		if(!rootObj["uri"].isUndefined()&&rootObj["uri"].isString())
		{
			m_imgSource=rootObj["uri"].toString();
			emit imgSourceChanged();
		}
	}
}
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/src/nftmonitor.cpp'}

We are also interested in getting the issuer of the NFT
```cpp
auto IssuerFea=nft_output->get_immutable_feature_(qblocks::Feature::Issuer_typ);
if(IssuerFea)
{
	m_issuer=std::static_pointer_cast<const qblocks::Issuer_Feature>(IssuerFea)->issuer()->addr().toHexString();
}
//emit the signal in order for QML side to know that the property of the element has changed.
emit issuerChanged();
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/src/nftmonitor.cpp'}
The latter allows the developers to give a higher attack coefficient to  their minted NFTs,
making them rare. 

### Exposing the custom QML type

In our case, we are developing a simple custom type, a type that is going to be instantiated from QML like
```qml
CPPMonitor
{
	id:monitor
        address: "rms1qqm54f5rlprtna9hkrahgmnv36pxyvevhhx6mhgclmj028etgtxpwpw7l88"
	nodeAddr: "https://api.testnet.shimmer.network"
        onIssuerChanged:
        {
            console.log("Issuer:",monitor.issuer);
            console.log("Uri:",monitor.imgSource);
            console.log("Name:",monitor.name);
            console.log("Attack:",monitor.attack);
        }

}
```
QML is designed to be easily [extensible through C++ code](https://doc.qt.io/qt-6/qtqml-cppintegration-overview.html) and allows many use cases.
Depending on your situation one has to choose the right [integration method](https://doc.qt.io/qt-6/qtqml-cppintegration-overview.html#choosing-the-correct-integration-method-between-c-and-qml).

>Due to the tight integration of the QML engine with the [Qt meta-object system](https://doc.qt.io/qt-6/metaobjects.html), any functionality that is appropriately exposed by a [QObject](https://doc.qt.io/qt-6/qobject.html)-derived class or a [Q_GADGET](https://doc.qt.io/qt-6/qobject.html#Q_GADGET) type is accessible from QML code. This enables C++ data and functions to be accessible directly from QML, often with little or no modification.
{: .prompt-info }

Our type is derived from QObject and exposes certain properties to QML
```cpp
class OMONI_EXPORT CPPMonitor : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString  address MEMBER m_address NOTIFY addressChanged)
    Q_PROPERTY(QString  issuer MEMBER m_issuer NOTIFY issuerChanged)
    Q_PROPERTY(QString  imgSource READ getImgSource NOTIFY imgSourceChanged)
    Q_PROPERTY(quint16  attack READ getAttack NOTIFY attackChanged)
    Q_PROPERTY(QString  name READ getName NOTIFY nameChanged)
    Q_PROPERTY(QUrl  nodeAddr MEMBER m_nodeAddr NOTIFY nodeAddrChanged)

    QML_ELEMENT
	...

```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/include/nftmonitor.hpp'}
The `Q_OBJECT` macro must appear in the private section of a class definition that declares its signals and slots or that uses other services provided by Qt's meta-object system.
The `Q_PROPERTY` macro declares the properties of our type to QML.
In the case of 
```cpp
Q_PROPERTY(QString  address MEMBER m_address NOTIFY addressChanged)
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/include/nftmonitor.hpp'}

This binds the exposed QML property `address` to the class member `m_address`.
Emitting the signal `addressChanged()` from C++ notifies the QML engine that the `address` property has changed.
Changing the property from QML triggers the `addressChanged()` signal that can be [connected](https://doc.qt.io/qt-6/signalsandslots.html) to a slot in C++.
 
The `QML_ELEMENT` is used because we want the type to be instantiated from the QML side.

> We have gone through the main parts needed to extend the QML language by the creation of a custom type from C++.
A custom type that uses the Shimmer++ libraries to monitor NFT Outputs linked to an address.
The type also exposes information taken from the `NFT Output`  to QML.
The latter allows for easy use of this information in user interfaces created by the powerful QML language. 
{: .prompt-info }

An example of the use of this type is present  in the `QMLMonitor.qml` file.
In this case, the `QMLMonitor` type is [defined in a QML document](https://doc.qt.io/qt-6/qtqml-documents-definetypes.html) and can be seen as a higher-level type of our `CPPMonitor`.

This document-defined type creates 3 visual items
1. A [Label](https://doc.qt.io/qt-6/qml-qtquick-controls-label.html) that shows the name and attack property of our C++-defined type.
2. An [Image](https://doc.qt.io/qt-6/qml-qtquick-image.html) that shows a URL picture taken from our C++-defined type.
3. A [TextField](https://doc.qt.io/qt-6/qml-qtquick-controls-textfield.html) where one can set the address property of our C++-defined type.

To be able to use our custom `CPPMonitor` in this QML document we need to use an [import statement](https://doc.qt.io/qt-6/qtqml-syntax-imports.html).
```qml
import NFTMonitor
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/qml/QMLMonitor.qml'}
The name `NFTMonitor` comes from the URI parameter declared in the CMake `qt6_add_qml_module` block of our library/module.

The document-defined type exposes two properties, the attack(`attack` property) and the node address(`nodeAddr` property)
```qml
Item
{
    id:root
    property int attack:0;
    property alias nodeAddr:monitor.nodeAddr
...
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/qml/QMLMonitor.qml'}

In the document, the C++-defined type is  instantiated like
```qml
CPPMonitor
{
	id:monitor
	onIssuerChanged:
	{
		root.attack=monitor.attack
	        if(monitor.issuer==="0x000b22cdeed839e4df23def46f7c2e8d04d3b66aab30b2695c7e9cbf21e9ef93cb")
	        {
        		root.attack=1000;
            		image.source= "qrc:/esterVtech.com/imports/NFTMonitor/images/BestNFT.png"
	        }
		else
		{
	            image.source= monitor.imgSource
		}
        }
        address: addrText.text

}
```
{: file='https://github.com/EddyTheCo/NFTQMLShimmerpp/blob/v0.1.1/qml/QMLMonitor.qml'}
Where `address: addrText.text` binds the text property of the `TextField` to the address property of our type.
The `onIssuerChanged` [block](https://doc.qt.io/qt-6/qtqml-syntax-signals.html#receiving-signals-with-signal-handlers) is a JavaScript function that is executed every time the signal `issuerChanged` is emitted from C++.
The latter function sets the URL of the image and gives the maximum attack coefficient to NFTs with a certain issuer.
 

## Conclusions

We have created a CMake package with a library inside. 
A library that is a QML Module with two custom QML types, one defined from C++ sources and the other defined through a QML document.
Both types use the Shimmer++ libraries to communicate with the Stardust nodes and the UTXO model of Shimmer. 
Other examples of custom QML types that interact with the Shimmer ecosystem are
 
- [Account GUI](https://github.com/EddyTheCo/account)

- [Client Connection GUI](https://github.com/EddyTheCo/ConectionSettings)

- [Middle Pay](https://github.com/EddyTheCo/MidlePay)

We are looking forward to using your custom QML types or your contributions to the previous ones.

The purpose of these posts is to create a community around the Shimmer ecosystem. A community that shares knowledge and contributes to the development of applications that trust the Shimmer nodes. Find bugs, and typos, learn and teach us what you know by contributing!

In future posts, I will explain how to use the Shimmer++ libraries and these custom QML types to create a GUI application.
A GUI application that is multi-platform, with the same code we will deploy the application in major desktop platforms, browsers,  mobile and, embedded devices.   

Please let me know in the comments if you find it useful. Let me know your doubts about the Stardust protocol, the Layer 1 of Shimmer, and Shimmer++.


## Watch the video! 
{% include embed/youtube.html id='ibErRWgGI1M' %}
