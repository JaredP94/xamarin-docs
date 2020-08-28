---
title: "Xamarin.Forms resource dictionaries"
description: "Xamarin.Forms XAML resources are objects that can be shared and reused throughout a Xamarin.Forms application."
ms.prod: xamarin
ms.assetid: DF103686-4A92-40FA-9CF1-A9376293B13C
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 04/01/2020
no-loc: [Xamarin.Forms, Xamarin.Essentials]
ms.custom: video
---

# Xamarin.Forms resource dictionaries

[![Download Sample](~/media/shared/download.png) Download the sample](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/xaml-resourcedictionaries)

A [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) is a repository for resources that are used by a Xamarin.Forms application. Typical resources that are stored in a `ResourceDictionary` include [styles](~/xamarin-forms/user-interface/styles/index.md), [control templates](~/xamarin-forms/app-fundamentals/templates/control-template.md), [data templates](~/xamarin-forms/app-fundamentals/templates/data-templates/index.md), colors, and converters.

In XAML, resources that are stored in a [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) can be referenced and applied to elements by using the `StaticResource` or `DynamicResource` markup extension. In C#, resources can also be defined in a `ResourceDictionary` and then referenced and applied to elements by using a string-based indexer. However, there's little advantage to using a `ResourceDictionary` in C#, as shared objects can be stored as fields or properties, and accessed directly without having to first retrieve them from a dictionary.

## Create resources in XAML

Every [`VisualElement`](xref:Xamarin.Forms.VisualElement) derived object has a [`Resources`](xref:Xamarin.Forms.VisualElement.Resources) property, which is a [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) that can contain resources. Similarly, an [`Application`](xref:Xamarin.Forms.Application) derived object has a [`Resources`](xref:Xamarin.Forms.VisualElement.Resources) property, which is a [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) that can contain resources.

A Xamarin.Forms application contains only class that derives from [`Application`](xref:Xamarin.Forms.Application), but often makes use of many classes that derive from [`VisualElement`](xref:Xamarin.Forms.VisualElement), including pages, layouts, and controls. Any of these objects can have its `Resources` property set to a [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) containing resources. Choosing where to put a particular `ResourceDictionary` impacts where the resources can be used:

- Resources in a `ResourceDictionary` that is attached to a view such as [`Button`](xref:Xamarin.Forms.Button) or [`Label`](xref:Xamarin.Forms.Label) can only be applied to that particular object.
- Resources in a `ResourceDictionary` attached to a layout such as [`StackLayout`](xref:Xamarin.Forms.StackLayout) or [`Grid`](xref:Xamarin.Forms.Grid) can be applied to the layout and all the children of that layout.
- Resources in a `ResourceDictionary` defined at the page level can be applied to the page and to all its children.
- Resources in a `ResourceDictionary` defined at the application level can be applied throughout the application.

With the exception of implicit styles, each resource in resource dictionary must have a unique string key that's defined with the `x:Key` attribute.

The following XAML shows resources defined in an application level `ResourceDictionary` in the **App.xaml** file:

```xaml
<Application xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="ResourceDictionaryDemo.App">
    <Application.Resources>

        <Thickness x:Key="PageMargin">20</Thickness>

        <!-- Colors -->
        <Color x:Key="AppBackgroundColor">AliceBlue</Color>
        <Color x:Key="NavigationBarColor">#1976D2</Color>
        <Color x:Key="NavigationBarTextColor">White</Color>
        <Color x:Key="NormalTextColor">Black</Color>

        <!-- Implicit styles -->
        <Style TargetType="{x:Type NavigationPage}">
            <Setter Property="BarBackgroundColor"
                    Value="{StaticResource NavigationBarColor}" />
            <Setter Property="BarTextColor"
                    Value="{StaticResource NavigationBarTextColor}" />
        </Style>

        <Style TargetType="{x:Type ContentPage}"
               ApplyToDerivedTypes="True">
            <Setter Property="BackgroundColor"
                    Value="{StaticResource AppBackgroundColor}" />
        </Style>

    </Application.Resources>
</Application>
```

In this example, the resource dictionary defines a [`Thickness`](xref:Xamarin.Forms.Thickness) resource, multiple [`Color`](xref:Xamarin.Forms.Color) resources, and two implicit [`Style`](xref:Xamarin.Forms.Style) resources. For more information about the `App` class, see [Xamarin.Forms App Class](~/xamarin-forms/app-fundamentals/application-class.md).

> [!NOTE]
> It's also valid to place all resources between explicit `ResourceDictionary` tags. However, since Xamarin.Forms 3.0 the `ResourceDictionary` tags are not required. Instead, the `ResourceDictionary` object is created automatically, and you can insert the resources directly between the `Resources` property-element tags.

## Consume resources in XAML

Each resource has a key that is specified using the `x:Key` attribute, which becomes its dictionary key in the `ResourceDictionary`. The key is used to reference a resource from the [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) with the [`StaticResource`](xref:Xamarin.Forms.Xaml.StaticResourceExtension) or [`DynamicResource`](xref:Xamarin.Forms.Xaml.DynamicResourceExtension) markup extension.

The `StaticResource` markup extension is similar to the `DynamicResource` markup extension in that both use a dictionary key to reference a value from a resource dictionary. However, while the `StaticResource` markup extension performs a single dictionary lookup, the `DynamicResource` markup extension maintains a link to the dictionary key. Therefore, if the dictionary entry associated with the key is replaced, the change is applied to the visual element. This enables runtime resource changes to be made in an application. For more information about markup extensions, see [XAML Markup Extensions](~/xamarin-forms/xaml/markup-extensions/index.md).

The following XAML example shows how to consume resources, and also defines additional resources in a [`StackLayout`](xref:Xamarin.Forms.StackLayout):

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="ResourceDictionaryDemo.HomePage"
             Title="Home Page">
    <StackLayout Margin="{StaticResource PageMargin}">
        <StackLayout.Resources>
            <!-- Implicit style -->
            <Style TargetType="Button">
                <Setter Property="FontSize" Value="Medium" />
                <Setter Property="BackgroundColor" Value="#1976D2" />
                <Setter Property="TextColor" Value="White" />
                <Setter Property="CornerRadius" Value="5" />
            </Style>
        </StackLayout.Resources>

        <Label Text="This app demonstrates consuming resources that have been defined in resource dictionaries." />
        <Button Text="Navigate"
                Clicked="OnNavigateButtonClicked" />
    </StackLayout>
</ContentPage>
```

In this example, the [`ContentPage`](xref:Xamarin.Forms.ContentPage) object consumes the implicit style defined in the application level resource dictionary. The [`StackLayout`](xref:Xamarin.Forms.StackLayout) object consumes the `PageMargin` resource defined in the application level resource dictionary, while the [`Button`](xref:Xamarin.Forms.Button) object consumes the implicit style defined in the [`StackLayout`](xref:Xamarin.Forms.StackLayout) resource dictionary. This results in the appearance shown in the following screenshots:

[![Consuming ResourceDictionary Resources](resource-dictionaries-images/consuming.png "Consuming ResourceDictionary resources")](resource-dictionaries-images/consuming-large.png#lightbox "Consuming ResourceDictionary resources")

> [!IMPORTANT]
> Resources that are specific to a single page shouldn't be included in an application level resource dictionary, as such resources will then be parsed at application startup instead of when required by a page. For more information, see [Reduce the Application Resource Dictionary Size](~/xamarin-forms/deploy-test/performance.md).

## Resource lookup behavior

The following lookup process occurs when a resource is referenced with the [`StaticResource`](xref:Xamarin.Forms.Xaml.StaticResourceExtension) or [`DynamicResource`](xref:Xamarin.Forms.Xaml.DynamicResourceExtension) markup extension:

- The requested key is checked for in the resource dictionary, if it exists, for the element that sets the property. If the requested key is found, its value is returned and the lookup process terminates.
- If a match isn't found, the lookup process searches the visual tree upwards, checking the resource dictionary of each parent element. If the requested key is found, its value is returned and the lookup process terminates. Otherwise the process continues upwards until the root element is reached.
- If a match isn't found at the root element, the application level resource dictionary is examined.
- If a match still isn't found, a `XamlParseException` is thrown.

Therefore, when the XAML parser encounters a `StaticResource` or `DynamicResource` markup extension, it searches for a matching key by traveling up through the visual tree, using the first match it finds. If this search ends at the page and the key still hasn't been found, the XAML parser searches the `ResourceDictionary` attached to the `App` object. If the key is still not found, an exception is thrown.

## Override resources

When resources share keys, resources defined lower in the visual tree will take precedence over those defined higher up. For example, setting an `AppBackgroundColor` resource to `AliceBlue` at the application level will be overridden by a page level `AppBackgroundColor` resource set to `Teal`. Similarly, a page level `AppBackgroundColor` resource will be overridden by a control level `AppBackgroundColor` resource.

## Stand-alone resource dictionaries

A class derived from [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) can also be in a separate stand-alone file. The resultant file can then be shared among applications.

To create such a file, add a new **Content View** or **Content Page** item to the project (but not a **Content View** or **Content Page** with only a C# file). Delete the code-behind file, and in the XAML file change the name of the base class from [`ContentView`](xref:Xamarin.Forms.ContentView) or [`ContentPage`](xref:Xamarin.Forms.ContentPage) to [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary). In addition, remove the `x:Class` attribute from the root tag of the file.

The following XAML example shows a [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) named **MyResourceDictionary.xaml**:

```xaml
<ResourceDictionary xmlns="http://xamarin.com/schemas/2014/forms"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
    <DataTemplate x:Key="PersonDataTemplate">
        <ViewCell>
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="0.5*" />
                    <ColumnDefinition Width="0.2*" />
                    <ColumnDefinition Width="0.3*" />
                </Grid.ColumnDefinitions>
                <Label Text="{Binding Name}"
                       TextColor="{StaticResource NormalTextColor}"
                       FontAttributes="Bold" />
                <Label Grid.Column="1"
                       Text="{Binding Age}"
                       TextColor="{StaticResource NormalTextColor}" />
                <Label Grid.Column="2"
                       Text="{Binding Location}"
                       TextColor="{StaticResource NormalTextColor}"
                       HorizontalTextAlignment="End" />
            </Grid>
        </ViewCell>
    </DataTemplate>
</ResourceDictionary>
```

In this example, the [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) contains a single resource, which is an object of type [`DataTemplate`](xref:Xamarin.Forms.DataTemplate). **MyResourceDictionary.xaml** can be consumed by merging it into another resource dictionary.

## Merged resource dictionaries

Merged resource dictionaries combine one or more [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) objects into another `ResourceDictionary`.

### Merge local resource dictionaries

A local [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) file can be merged into another `ResourceDictionary` by creating a `ResourceDictionary` object whose [`Source`](xref:Xamarin.Forms.ResourceDictionary.Source) property is set to the filename of the XAML file with the resources:

```xaml
<ContentPage ...>
    <ContentPage.Resources>
        <!-- Add more resources here -->
        <ResourceDictionary Source="MyResourceDictionary.xaml" />
        <!-- Add more resources here -->
    </ContentPage.Resources>
    ...
</ContentPage>
```

This syntax does not instantiate the `MyResourceDictionary` class. Instead, it references the XAML file. For that reason, when setting the [`Source`](xref:Xamarin.Forms.ResourceDictionary.Source) property, a code-behind file isn't required, and the `x:Class` attribute can be removed from the root tag of the **MyResourceDictionary.xaml** file.

> [!IMPORTANT]
> The [`Source`](xref:Xamarin.Forms.ResourceDictionary.Source) property can only be set from XAML.

### Merge resource dictionaries from other assemblies

A [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) can also be merged into another `ResourceDictionary` by adding it into the [`MergedDictionaries`](xref:Xamarin.Forms.ResourceDictionary.MergedDictionaries) property of the `ResourceDictionary`. This technique allows resource dictionaries to be merged, regardless of the assembly in which they reside. Merging resource dictionaries from external assemblies requires the [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) to have a build action set to **EmbeddedResource**, to have a code-behind file, and to define the `x:Class` attribute in the root tag of the file.

> [!WARNING]
> The [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) class also defines a [`MergedWith`](xref:Xamarin.Forms.ResourceDictionary.MergedWith) property. However, this property has been deprecated and should no longer be used.

The following code example shows two resource dictionaries being added to the [`MergedDictionaries`](xref:Xamarin.Forms.ResourceDictionary.MergedDictionaries) collection of a page level [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary):

```xaml
<ContentPage ...
             xmlns:local="clr-namespace:ResourceDictionaryDemo"
             xmlns:theme="clr-namespace:MyThemes;assembly=MyThemes">
    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Add more resources here -->
            <ResourceDictionary.MergedDictionaries>
                <!-- Add more resource dictionaries here -->
                <local:MyResourceDictionary />
                <theme:LightTheme />
                <!-- Add more resource dictionaries here -->
            </ResourceDictionary.MergedDictionaries>
            <!-- Add more resources here -->
        </ResourceDictionary>
    </ContentPage.Resources>
    ...
</ContentPage>
```

In this example, a resource dictionary from the same assembly, and a resource dictionary from an external assembly, are merged into the page level resource dictionary. In addition, you can also add other `ResourceDictionary` objects within the [`MergedDictionaries`](xref:Xamarin.Forms.ResourceDictionary.MergedDictionaries) property-element tags, and other resources outside of those tags.

> [!IMPORTANT]
> There can be only one `MergedDictionaries` property-element tag in a [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary), but you can put as many `ResourceDictionary` objects in there as required.

When merged [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary) resources share identical `x:Key` attribute values, Xamarin.Forms uses the following resource precedence:

1. The resources local to the resource dictionary.
1. The resources contained in the resource dictionaries that were merged via the `MergedDictionaries` collection, in the reverse order they are listed in the `MergedDictionaries` property.

> [!NOTE]
> Searching resource dictionaries can be a computationally intensive task if an application contains multiple, large resource dictionaries. Therefore, to avoid unnecessary searching, you should ensure that each page in an application only uses resource dictionaries that are appropriate to the page.

## Related links

- [Resource Dictionaries (sample)](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/xaml-resourcedictionaries)
- [XAML Markup Extensions](~/xamarin-forms/xaml/markup-extensions/index.md)
- [Xamarin.Forms Styles](~/xamarin-forms/user-interface/styles/index.md)
- [ResourceDictionary](xref:Xamarin.Forms.ResourceDictionary)

## Related video

> [!Video https://channel9.msdn.com/Shows/XamarinShow/XamarinForms-101-Application-Resources/player]

[!include[](~/essentials/includes/xamarin-show-essentials.md)]
