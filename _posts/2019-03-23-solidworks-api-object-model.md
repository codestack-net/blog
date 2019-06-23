---
layout: post
title: Understanding API Object Model
description: Explanation of SOLIDWORKS API Object Model and Classes Hierarchy. Explanation of relationship between classes and methods.
image: /res/2019-03-23-understanding-api-object-model/solidworks-api-interactive-diagram.png
labels: api object model, class hierarchy, class relationship, solidworks api, 
redirect_from:
  - /2019/03/understanding-api-object-model.html
---
SOLIDWORKS API object model contains hundreds of interfaces and thousands of methods and properties. When starting developing applications or macros for SOLIDWORKS it might be confusions to understand all the relationships to find the correct route of retrieving the pointer to the required object.

Usually the pointer to the root object (ISldWorks) is provided (either via calling the Application.SldWorks property in the macro or from the app parameter in the ConnectToSw function).

To access the pointers to another options it is important to understand the accessibility map of the methods and interfaces.

SOLIDWORKS API Help Documentation provides a special Accessors section for all interfaces which helps to understand how this specific interface can be accessed.

To access the pointers to another options it is important to understand the accessibility map of the methods and interfaces.

SOLIDWORKS API Help Documentation provides a special Accessors section for all interfaces which helps to understand how this specific interface can be accessed.

{% include img.html src="/res/2019-03-23-understanding-api-object-model/solidworks-api-methods-accessors.png" alt="SOLIDWORKS API methods accessors" align="center" %}

For example, the snapshot above is an Accessors section of the IAnnotation Interface which means that the pointer to IAnnotation Interface could be retrieved either via IAnnotation::GetNext2 Method or IAnnotationView::Annotations property or other properties or methods in this list.

At the same time some of the other interface can be directly cast without the need of calling the method. For example IModelDoc2 is a parent of IPartDoc, IAssemblyDoc and IDrawingDoc which means that specific declaration of IPartDoc variable has an access of all methods and properties of the IModelDoc2.

For more information on this subject please follow the [Accessors](https://www.codestack.net/solidworks-api/getting-started/api-object-model/accessors/) article.

To simplify the process of finding the accessors I have published an interactive diagram

{% include img.html src="/res/2019-03-23-understanding-api-object-model/solidworks-api-interactive-diagram.png" alt="SOLIDWORKS API Object model interactive diagram" align="center" %}

You can zoom, pan and click on the diagram blocks to explore the relationship.

This diagram represents the most commonly used interfaces and method. I will be adding more relations and methods to this diagram in future.

Follow the [Class Diagram Relationship](https://www.codestack.net/solidworks-api/getting-started/api-object-model/class-diagram/) article for more information.