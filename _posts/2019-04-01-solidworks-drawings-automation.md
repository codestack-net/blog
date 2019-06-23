---
layout: post
title: Basic Concepts of Automating Drawings Using SOLIDWORKS API
description: 'Explanation of basic concepts of automating drawings using SOLIDWORKS API: dimensioning, drawing sketches, transforming coordinates'
image:
labels: dimension, drawing, sheet, transform, view
redirect_from:
  - /2019/04/api-automating-drawings-concepts.html
---
Automating drawings using SOLIDWORKS API is one of the most common tasks when it comes to macro development. This is however one of the most challenging sections of development for SOLIDWORKS.

Usually the macros for drawing would

* Automatically add dimensions to views
* Insert and optimise the view locations
* Insert annotations, tables and balloons
* Create sketch segments and points (additional drawing symbols)

In this blog post I will introduce you to the basic concepts of SOLIDWORKS API for drawings.

Drawing document consists of 2D drawing sheets ([ISheet](http://help.solidworks.com/2019/english/api/draftsightapi/interop.dsautomation~interop.dsautomation.isheet.html) interface) which may contain sketch segments, annotations (i.e. dimensions, notes, tables) and drawing views ([IView](http://help.solidworks.com/2019/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IView.html) interface). Drawing sheet can be treated as a sketch ([ISketch](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.isketch.html) interface) and it has its own sketch manager. Drawing views are 2D snapshot of 3D model which also have an underlined sketch which can contain sketch segments and annotation.

Let's now look at the 2 most common scenarios of automating drawing

## Adding Dimensions To Drawing View Entities

When entity needs to be dimensioned there are 2 problems need to be solved:

* Find the proper entity(es) to dimension
* Calculate the correct location of the dimension

There are 2 main approaches could be used when finding the entities for dimensioning in the drawing views:

* Entity can be found in the underlying document and then corresponding entity selected in the drawing view
* Entity can be found directly from the drawing view

First approach enables the possibility of using rich [IPartDoc](http://help.solidworks.com/2019/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IPartDoc.html) and [IAssemblyDoc](http://help.solidworks.com/2019/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IAssemblyDoc.html) SOLIDWORKS API interface to lookup entities, such as [Persist Reference Id](https://www.codestack.net/solidworks-api/document/tracking-objects/persist-references/), named entities, feature, body traversing etc. It is however introducing the limitation as found entity might not necessarily be visible on the view (it can be either hidden or overlapped by another entities). Follow the [Dimension Named Model Entities](https://www.codestack.net/solidworks-api/document/drawing/view-dimension-model-entities/) link for code example of using this approach to add dimension.

Second approach provides less flexibility in the filtering, but allows to only find visible entities. Check the [Dimension Visible Entities](https://www.codestack.net/solidworks-api/document/drawing/view-dimension-drawing-entities/) for the corresponding macro example.

The most complex part here is to calculate the optimal location of the added dimension. It is important to understand that it is required to consider the drawing view transform when calculating the position as dimension. Dimension is inserted into the drawing sheet space while all entities, which are usually used for calculation, reside in the drawing view space. The above examples demonstrate the way of performing this transformation.

## Adding Sketch Segments

As I've previously mentioned there are separate sketches available for the drawing sheet itself and for each drawing view. So depending on the need the sketch segments might need to be inserted using one of these options.

When sketch segment inserted into the sheet space it is not attached to the drawing view. But inserting the sketch segments into the drawing view would embed it. This means that segment will be moved and rotated together with the view.

When transforming the coordinate system from the drawing view model space (i.e. from the underlying 3D document coordinate system) into the sheet space it is required to consider the transformation of the drawing view ([IView::ModelToViewTransform](http://help.solidworks.com/2018/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.iview~modeltoviewtransform.html)) as well as scale of the drawing sheet. Refer the [Draw Sketch Segments In Sheet](https://www.codestack.net/solidworks-api/document/drawing/sheet-context-sketch/) example for detailed explanation and source code.

When transforming the coordinate from the sheet space into the drawing view sketch space it is required to use an inverted matrix of the [ISketch::ModelToSketchTransform](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.isketch~modeltosketchtransform.html), where the sketch retrieved via [IView::GetSketch](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.iview~getsketch.html) SOLIDWORKS API method. This example demonstrates how to use this approach: [Create Sketch Segments In Drawing View](https://www.codestack.net/solidworks-api/document/drawing/drawing-view-sketch/)

Explore the provided code examples for more information and guide for the automation of drawings using SOLIDWORKS API.