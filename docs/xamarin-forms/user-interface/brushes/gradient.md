---
title: "Xamarin.Forms Brushes: Gradients"
description: "The Xamarin.Forms GradientBrush class is an abstract class that describes a gradient, composed of gradient stops."
ms.prod: xamarin
ms.assetid: 24763E56-74EC-4082-897B-E4EAACCADFEE
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 07/27/2020
no-loc: [Xamarin.Forms, Xamarin.Essentials]
---

# Xamarin.Forms Brushes: Gradients

![Preview API](~/media/shared/preview.png "This API is currently pre-release")

[![Download Sample](~/media/shared/download.png) Download the sample](/samples/xamarin/xamarin-forms-samples/userinterface-brushdemos/)

The `GradientBrush` class derives from the `Brush` class, and is an abstract class that describes a gradient, which is composed of gradient stops. A gradient brush paints an area with multiple colors that blend into each other along an axis. Classes that derive from `GradientBrush` describe different ways of interpreting gradient stops, and Xamarin.Forms provides the following gradient brushes:

- `LinearGradientBrush`, which paints an area with a linear gradient. For more information, see [Xamarin.Forms Brushes: Linear gradients](lineargradient.md).
- `RadialGradientBrush`, which paints an area with a radial gradient. For more information, see [Xamarin.Forms Brushes: Radial gradients](radialgradient.md).

The `GradientBrush` class defines the `GradientStops` property, of type `GradientStopsCollection`, which represents the brush's gradient stops, each of which specifies a color and an offset along the brush's gradient axis. A `GradientStopsCollection` is an `ObservableCollection` of `GradientStop` objects. The `GradientStops` property is backed by a [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) object, which means that it can be the target of data bindings, and styled.

> [!NOTE]
> The `GradientStops` property is the `ContentProperty` of the `GradientBrush` class, and so does not need to be explicitly set from XAML.

## Gradient stops

Gradient stops are the building blocks of a gradient brush, and specify the colors in the gradient and their location along the gradient axis. Gradient stops are specified using `GradientStop` objects.

The `GradientStop` class defines the following properties:

- `Color`, of type [`Color`](xref:Xamarin.Forms.Color), which represents the color of the gradient stop. The default value of this property is `Color.Default`.
- `Offset`, of type `float`, which represents the location of the gradient stop within the gradient vector. The default value of this property is 0, and valid values are in the range 0.0-1.0. The closer this value is to 0, the closer the color is to the start of the gradient. Similarly, the closer this value is to 1, the closer the color is to the end of the gradient.

These properties are backed by [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) objects, which means that they can be targets of data bindings, and styled.

> [!IMPORTANT]
> The coordinate system used by gradients is relative to a bounding box for the output area. 0 indicates 0 percent of the bounding box, and 1 indicates 100 percent of the bounding box. Therefore, (0.5,0.5) describes a point in the middle of the bounding box, and (1,1) describes a point at the bottom right of the bounding box.

The following XAML example creates a diagonal `LinearGradientBrush` with four colors:

```xaml
<LinearGradientBrush StartPoint="0,0"
                     EndPoint="1,1">
    <GradientStop Color="Yellow"
                  Offset="0.0" />
    <GradientStop Color="Red"
                  Offset="0.25" />
    <GradientStop Color="Blue"
                  Offset="0.75" />             
    <GradientStop Color="LimeGreen"
                  Offset="1.0" />
</LinearGradientBrush>                                                       
```

The color of each point between gradient stops is interpolated as a combination of the color specified by the two bounding gradient stops. The following diagram shows the gradient stops from the previous example:

![Frame painted with a diagonal LinearGradientBrush](gradient-images/gradient-stops.png)

In this diagram, the circles mark the position of gradient stops, and the dashed line shows the gradient axis. The first gradient stop specifies the color yellow at an offset of 0.0. The second gradient stop specifies the color red at an offset of 0.25. The points between these two gradient stops gradually change from yellow to red as you move from left to right along the gradient axis. The third gradient stop specifies the color blue at an offset of 0.75. The points between the second and third gradient stops gradually change from red to blue. The fourth gradient stop specifies the color lime green at at offset of 1.0. The points between the third and fourth gradient stops gradually change from blue to lime green.

## Related links

- [BrushesDemos (sample)](/samples/xamarin/xamarin-forms-samples/userinterface-brushdemos/)
- [Xamarin.Forms Brushes: Linear gradients](lineargradient.md)
- [Xamarin.Forms Brushes: Radial gradients](radialgradient.md)