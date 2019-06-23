---
layout: post
title: Storing Data In SOLIDWORKS 3rd Party Storage And Store
description: Storing data into SOLIDWORKS 3rd party storage and store using API
image: /res/2019-04-18-solidworks-third-party-storage/solidworks-store-diagram.png
labels: 3rd party storage, c#, data, serialization, vb.net
redirect_from:
  - /2019/04/solidworks-third-party-storage.html
---
When developing SOLIDWORKS add-ins it might be required to store custom data in the model. This might include storing of custom options, attributes or additional model tree (e.g. electrical, inspection etc.)

SOLIDWORKS enables storing the data directly within main model stream. Which makes it easy to serialise and deserialise different data using popular frameworks such as [Binary Serialization](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatters.binary.binaryformatter?view=netframework-4.7.2), [Xml Serialization](https://docs.microsoft.com/en-us/dotnet/api/system.xml.serialization.xmlserializer), [Data Contract Serialisation](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.datacontractserializer?view=netframework-4.7.2), [JSON Serialisation](https://www.newtonsoft.com/json/help/html/SerializingJSON.htm) etc. This approach provides the most performance efficient and simple way of working with data.

Data can be stored into 2 main components: [3rd Party Storage and 3rd Party Storage Store](https://www.codestack.net/solidworks-api/data-storage/third-party/).

3rd Party Storage Store can be considered as a folder which can hold sub storages and streams. This approach is useful when your application is required to store different types of data which should be accessed independently. While 3rd Party Storage is a stream for a single data object to be stored.

Here is a diagram which demonstrates the structure of SOLIDWORKS document storage:

{% include img.html src="/res/2019-04-18-solidworks-third-party-storage/solidworks-store-diagram.png" alt="SOLIDWORKS store diagram" align="center" %}

Red elements represent the containers managed directly by SOLIDWORKS while other elements represent the containers managed by 3rd parties.

3rd Party Storage represented with [IStream](https://docs.microsoft.com/en-us/windows/desktop/api/objidl/nn-objidl-istream) object. It is required to call the [IModelDoc2::IGet3rdPartyStorage](http://help.solidworks.com/2015/english/api/sldworksapi/SOLIDWORKS.Interop.sldworks~SOLIDWORKS.Interop.sldworks.IModelDoc2~IGet3rdPartyStorage.html) SOLIDWORKS API method to get the pointer to the stream. Stream must be released via [IModelDoc2::IRelease3rdPartyStorage](http://help.solidworks.com/2015/english/api/sldworksapi/SOLIDWORKS.Interop.sldworks~SOLIDWORKS.Interop.sldworks.IModelDoc2~IRelease3rdPartyStorage.html) method.

3rd Party Storage store represented with [IStorage](https://docs.microsoft.com/en-us/windows/desktop/api/objidl/nn-objidl-istorage) object. It is required to call the [IModelDocExtension::IGet3rdPartyStorageStore](http://help.solidworks.com/2015/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IModelDocExtension~IGet3rdPartyStorageStore.html) SOLIDWORKS API method to get the pointer to the store. Store must be released via [IModelDocExtension::IRelease3rdPartyStorageStore](http://help.solidworks.com/2015/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IModelDocExtension~IRelease3rdPartyStorageStore.html) method.

Storage and store can be accessed for the read at any time after document is loaded, but it can only be saved within the SaveToStorage and SaveToStorageStore notifications of part, assembly and drawing.

Refer the [3rd Party Storage And Store](https://www.codestack.net/solidworks-api/data-storage/third-party/) for more information.

The following [C# Example](https://www.codestack.net/solidworks-api/data-storage/third-party/tree-structure-serialization/) demonstrates how to serialize custom tree in the model stream. This [VB.NET Example](https://www.codestack.net/solidworks-api/data-storage/third-party/embed-file/) shows how to store the file within the model (attachment). This [C# Example](https://www.codestack.net/solidworks-api/data-storage/third-party/custom-properties-revisions/) uses 3rd Party Storage Store to save revisions (snapshots) of custom properties.

One of the benefits of 3rd Party Storage And Store that it can be also accessed from the [Document Manager API](https://www.codestack.net/solidworks-document-manager-api/).

Explore the following [C# Example](https://www.codestack.net/solidworks-document-manager-api/document/data-storage/third-party/add-watermark/) which demonstrates how to add digital watermark into the model stream and [this C# Example](https://www.codestack.net/solidworks-document-manager-api/document/data-storage/third-party/add-comments/) which allows to manage custom user comments within 3rd Party Storage Store.

## Simplified explanation of Streams and Storages

SOLIDWORKS file is a complex structure which contains different information:

* Information about Features
* Custom properties
* Configurations with custom properties and parameters
* Previews
* Components transformations in assembly
* Annotations
* Attachments (such as design binder and design tables)
* etc.

In order to maximise the accessibility and performance it is vital to organise this data in the most efficient way. It should be possible to read and write the data on demand to different sub-storages. For example when SOLIDWORKS part document contains multiple configurations it should be possible to access the data of specific configuration without loading all of the configurations.

Think of how you organise your projects in your system. You might have a folder called **Projects**, which contains sub folders with each individual project which contains SOLIDWORKS files. You might have another folder called **Macros** to store useful automation macros. Perhaps some **Documents** folder containing different MS Word and Excel files. Of course everyone will have different structures, but the point is all the data is organised.

Let's consider SOLIDWORKS file as a directory. Similar to your projects it might have sub directory called **Configurations**. This directory would have files (similar to Excel) - each file corresponds to the configuration. There may be another file which represents the preview in png format. There will be also data for feature tree, geometry etc. All of those files would have its own format and can be accessed individually.

In fact prior to SOLIDWORKS 2015, documents were stored as a compound OLE Storage and SW file could be just unzipped and some internal data can be inspected. So it was possible to get preview image, attachments etc. Of course some of the elements were encrypted so cannot be read directly.

{% include img.html src="/res/2019-04-18-solidworks-third-party-storage/solidworks-file-ole-storage.png" alt="SOLIDWORKS file OLE storage structure" align="center" %}

When we are talking about 3rd Party Storage and Store those are just another folder in the SOLIDWORKS structure which enables adding, removing or editing files and folders via API.

Storage Store is a folder, while Storage is a file. Those are generic Windows Operating Systems concepts so Windows API can be used to work with those data containers.