---
layout: post
title: SOLIDWORKS Custom Properties Automation
description: Detailed overview of automating custom properties using SOLIDWORKS API. Reading and writing custom properties to file, configuration and cut-list items. Handling the custom properties modification using events. Accessing the custom properties from Document Manager API, introduction to invisible custom properties
image: \res\2019-09-18-custom-properties-automation\solidworks-custom-properties.png
labels: custom properties,cut-list,data,storage
---
{% include img.html src="\res\2019-09-18-custom-properties-automation\solidworks-custom-properties.svg" width=350 alt="SOLIDWORKS Custom Properties" align="center" %}

Custom properties automation is one of the most demanding parts of SOLIDWORKS API. Despite there are only 4 basic APIs available via [ICustomPropertyManager ](http://help.solidworks.com/2018/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.ICustomPropertyManager.html) interface for reading, writing, modifying and deleting properties, there is endless number of possible application could be written based on those APIs. This includes but not limited to large applications such as PDM systems, ERP integrations, change management, quality control and small automation macros such as [properties sorting](https://www.codestack.net/solidworks-api/data-storage/custom-properties/sort/), [copying](https://www.codestack.net/solidworks-api/data-storage/custom-properties/copy-file-specific-to-configuration/) etc.

In this blog article I will go through various applications and code example for managing custom properties using SOLIDWORKS API.

* [Accessing Custom Property Manager](#accessing-custom-property-manager)
    * [General](#general)
    * [Configuration Specific](#configuration-specific)
    * [Cut-List](#cut-list)
* [Reading Properties](#reading-properties)
    * [Single Property](#single-property)
    * [Batch](#batch)
* [Writing Custom Properties](#writing-custom-properties)
* [File Summary](#file-summary)
* [Managing Properties Via Document Manager](#managing-properties-via-document-manager)
    * [Invisible Custom Properties](#invisible-custom-properties)
* [Handling Modification Events](#handling-modification-events)

## Accessing Custom Property Manager

All types of custom properties: file specific (general), configuration specific or cut-lists can be managed in unified way via [ICustomPropertyManager](http://help.solidworks.com/2018/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.ICustomPropertyManager.html) interface. The methods of this interface behave in the same way for all of the above group of properties and the only difference in a way of retrieving the instance of this interface.

### General

{% include img.html src="res\2019-09-18-custom-properties-automation\general-custom-properties.png" alt="General custom properties" align="center" %}

In order to connect to file specific (general) custom properties, it is required to call the [IModelDocExtension::CustomPropertyManager](http://help.solidworks.com/2016/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.imodeldocextension~custompropertymanager.html) property and pass an empty string as a property parameter.

~~~ vb
Dim swModel As SldWorks.ModelDoc2
...
Dim swCustPrpMgr As SldWorks.CustomPropertyManager
Set swCustPrpMgr = swModel.Extension.CustomPropertyManager("")
~~~

### Configuration Specific

In order to connect to configuration specific custom property manager it is required to call the [IModelDocExtension::CustomPropertyManager](http://help.solidworks.com/2016/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.imodeldocextension~custompropertymanager.html) property and pass the configuration name as a property parameter.

~~~ vb
Dim swModel As SldWorks.ModelDoc2
...
Dim swCustPrpMgr As SldWorks.CustomPropertyManager
Set swCustPrpMgr = swModel.Extension.CustomPropertyManager(confName)
~~~

Alternatively the pointer can be directly retrieved from the configuration via [IConfiguration::CustomPropertyManager](http://help.solidworks.com/2018/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IConfiguration~CustomPropertyManager.html) property.

~~~ vb
Dim swModel As SldWorks.ModelDoc2
Dim swConf As SldWorks.Configuration
...
Set swConf = swModel.GetConfigurationByName(confName)
Dim swCustPrpMgr As SldWorks.CustomPropertyManager
Set swCustPrpMgr = swConf.CustomPropertyManager
~~~

Configuration specific properties are only available for parts and assemblies and not applicable for drawings.

### Cut-List

{% include img.html src="res\2019-09-18-custom-properties-automation\cut-list-properties.png" alt="Cut-list custom properties" align="center" %}

In order to connect to cut-list properties manager for weldment and sheet metal parts, it is required to call the [IFeature::CustomPropertyManager](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.ifeature~custompropertymanager.html) property of the cut-list element feature in the Feature Manager tree.

~~~ vb
Dim swCutListFeat As SldWorks.Feature
...
Dim swCustPrpMgr As SldWorks.CustomPropertyManager
Set swCustPrpMgr = swCutListFeat.CustomPropertyManager
~~~

Although cut-lists can contain the configuration specific properties (e.g. length could be resolved to different values in different configurations) SOLIDWORKS API doesn't recognize this functionality and only provides values of the active configuration of the part document. If it is required to retrieved the configuration specific values of cut-list property, manually activate the configuration via [IModelDoc2::ShowConfiguration2](http://help.solidworks.com/2019/english/api/sldworksapi/SOLIDWORKS.Interop.sldworks~SOLIDWORKS.Interop.sldworks.IModelDoc2~ShowConfiguration2.html) method.

It is however possible to read the configuration specific cut-list properties from the referenced configuration of the component in the assembly by retrieving [assembly context](https://www.codestack.net/solidworks-api/document/assembly/context/) pointer to the cut-list feature. This method doesn't require to activate the configuration of the component's model to resolve the configuration specific cut-list properties. Refer [Read Component Cut-List Properties](https://www.codestack.net/solidworks-api/data-storage/custom-properties/read-component-cutlist/) macro for code example.

It is also possible to read configuration specific cut-list properties using [Document Manager API](#managing-properties-via-document-manager). 

## Reading Properties

There are 2 general ways of reading the values of custom properties. 

### Single Property

Value and additional information of a specific property can be retrieved by property name via [ICustomPropertyManager::Get6](http://help.solidworks.com/2018/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.ICustomPropertyManager~Get6.html) SOLIDWORKS API method. Note, you might need to use older version of this method, such as ::Get5, ::Get4 depending on the target version of SOLIDWORKS.

~~~ vb
Dim swCustPrpMgr As SldWorks.CustomPropertyManager
...
Dim prpVal As String
Dim prpResVal As String
Dim wasResolved As Boolean
Dim isLinked As Boolean

Dim res As Long
res = swCustPrpMgr.Get6(prpName, cached, prpVal, prpResVal, wasResolved, isLinked)
~~~

This method can be used in 2 modes:

* To retrieve the cached value
* To retrieved up-to-date value

The *cached* flag of ::GetX method only applies to configurations, as both general and cut-list properties are not configuration specific. When cached option is used, properties are read from the cached storage. This means that the returned values might not be up-to-date for the properties defined with configuration specific equations, such as weight, dimension value, volume etc. Alternatively when cached option is set to *False* values will be resolved if needed, that means that SOLIDWORKS might activate the configuration to resolved the value which may mark model as dirty. This option would imply performance penalty as model will be regenerated to update properties.

{% include img.html src="res\2019-09-18-custom-properties-automation\unevaluated-property-value.png" alt="Unresolved configuration specific custom property" align="center" %}

This method allows to extract comprehensive information about the property. 

 * Property Value (text expression) is a static value or the formula of the property
 * Evaluated Value - resolved value of the property
 * Was the property resolved. SOLIDWORKS API Help documentation states that this flag would be equal to *False* if cached value is retrieved from the configuration which was never activated (e.g. value \*0.00 value of Weight property in the snapshot above), but in all my tests this property always equals to *True*
 * Was this property linked or not (e.g. if the property derived from the parent part)

{% include img.html src="res\2019-09-18-custom-properties-automation\derived-part-property-page.png" alt="Derived part custom properties option" align="center" %}

The method returns the value indicating if property was read from cache, was resolved or didn't exist.

Refer the [Read All Properties](https://www.codestack.net/solidworks-api/data-storage/custom-properties/read-all-properties/) for the example of reading properties one-by one from all property sources: file, configuration and cut list.

### Batch

Names and values of all properties can be extracted via [ICustomPropertyManager::GetAll3](http://help.solidworks.com/2018/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.ICustomPropertyManager~GetAll3.html) SOLIDWORKS API method.

~~~ vb
Dim swCustPrpsMgr As SldWorks.CustomPropertyManager
...
Dim vPrpNames As Variant
Dim vPrpTypes As Variant
Dim vPrpVals As Variant
Dim vResVals As Variant
Dim vPrpsLink As Variant

Dim prpsCount As Integer
prpsCount = swCustPrpsMgr.GetAll3(vPrpNames, vPrpTypes, vPrpVals, vResVals, vPrpsLink)
~~~

Unlike single property extraction method, batch can only returns cached values from the configuration specific properties.

Refer the [Read Component Cut-List Properties](https://www.codestack.net/solidworks-api/data-storage/custom-properties/read-component-cutlist/)

## Writing Custom Properties

New custom property can be added by calling the [ICustomPropertyManager::Add3](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.icustompropertymanager~add3.html). This method allows to add new property and/or update the value of existing one.

~~~ vb
Dim swCustPrpMgr As SldWorks.CustomPropertyManager
Dim prpName As String
Dim prpVal As String
Dim prpType As swCustomInfoType_e
...
Dim res As Long
res = swCustPrpMgr.Add3(prpName, prpType, prpVal, swCustomPropertyAddOption_e.swCustomPropertyReplaceValue)

If res <> swCustomInfoAddResult_e.swCustomInfoAddResult_AddedOrChanged Then
    Err.Raise vbError, "", "Failed to set custom property. Error code: " & res
End If
~~~

Refer [Write All Properties](https://www.codestack.net/solidworks-api/data-storage/custom-properties/write-all-properties/) example which adds new custom property to all properties sources.

## File Summary

Summary information (such as author, comments, subject) can be extracted by calling the [IModelDoc2::SummaryInfo](http://help.solidworks.com/2018/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IModelDoc2~SummaryInfo.html) method and passing one of the required summary field as the parameter.

Refer the [Write Summary Information](https://www.codestack.net/solidworks-api/data-storage/custom-properties/write-summary-information/) and [Read Summary Information](https://www.codestack.net/solidworks-api/data-storage/custom-properties/read-summary-information/) for code examples.

## Managing Properties Via Document Manager

[SOLIDWORKS Document Manager](https://www.codestack.net/solidworks-document-manager-api/) is a lightweight library to access metadata from SOLIDWORKS files without the need of opening them in SOLIDWORKS or even having SOLIDWORKS installed. It is ideal for managing SOLIDWORKS custom properties (both reading and writing) to gain the maximum performance.

As Document Manager reads the data directly from file stream it cannot be used to re-evaluate the expressions for the configuration, i.e. it will only return the cached values of custom properties.

This library allows to access custom properties from all sources, i.e. general file properties, configuration specific properties and cut-list properties (furthermore configuration specific cut-list properties are also accessible).

Follow the [Read All Properties](https://www.codestack.net/solidworks-document-manager-api/document/data-storage/custom-properties/read-all-properties/) and [Write All Properties](https://www.codestack.net/solidworks-document-manager-api/document/data-storage/custom-properties/write-all-properties/) for the code examples of managing custom properties using SOLIDWORKS Document Manager API.

### Invisible Custom Properties

SOLIDWORKS models contains several invisible custom properties, such as $PRP:"SW-File Name", $PRP:"SW-Title". Those are present in the model but cannot be modified or listed from the user interface. It is however possible to access those properties via Document Manager API. Refer the [Read All Invisible Custom Properties](https://www.codestack.net/solidworks-document-manager-api/document/data-storage/custom-properties/read-all-invisible-properties/) for a VBA macro example which lists all invisible properties and outputs those to the immediate window of VBA editor. But it is also possible to add new custom properties to this list, refer the [Add Invisible Custom Property](https://www.codestack.net/solidworks-document-manager-api/document/data-storage/custom-properties/add-invisible-custom-property/). Once added the property can only be viewed or modified via Document Manager API and won't be available to the users. You can use this functionality to store some read only data, such as watermark, stamps etc.

### Handling Modification Events

SOLIDWORKS API provides 3 notifications to handle the modifications of the custom properties: AddCustomPropertyNotify, DeleteCustomPropertyNotify, ChangeCustomPropertyNotify. But unfortunately since SOLIDWORKS 2018 those events are only got raised for the properties modified by API (e.g. from macro) and ignored if property is modified by the user via *Summary Info* dialog. This introduces the major limitation for the automation software.

Refer the [Handle Modification Events](https://www.codestack.net/solidworks-api/data-storage/custom-properties/handle-events/) for a workaround for this issue. In this example the *CustomPropertiesEventsHandler* class raises a single event once the property is modified which can be handled in the following way:

~~~ cs
var handler = new CustomPropertiesEventsHandler(app, model);
handler.PropertyChanged += OnPropertyChanged;
...

private static void OnPropertyChanged(PropertyChangeAction_e type, string name, string conf, string value)
{
    Console.WriteLine($"Property {name}; Action: {type}; Configuration: {conf}; Value: {value}");
}
~~~
