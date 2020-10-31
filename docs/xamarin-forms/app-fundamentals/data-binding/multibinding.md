---
title: "Xamarin.Forms Multi-Bindings"
description: "This article explains how to attach a collection of Binding objects to a single binding target property using the MultiBinding class."
ms.prod: xamarin
ms.assetid: E73AE622-664C-4A90-B5B2-BD47D0E7A1A7
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 10/26/2020
---

# Xamarin.Forms Multi-Bindings

[![Download Sample](~/media/shared/download.png) Download the sample](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/databindingdemos)

Multi-bindings provide the ability to attach a collection of [`Binding`](xref:Xamarin.Forms.Binding) objects to a single binding target property. They are created with the `MultiBinding` class, which evaluates all of its `Binding` objects, and returns a single value through a `IMultiValueConverter` instance provided by your application. In addition, `MultiBinding` reevaluates all of its `Binding` objects when any of the bound data changes.

The `MultiBinding` class defines the following properties:

- `Bindings`, of type `IList<BindingBase>`, which represents the collection of [`Binding`](xref:Xamarin.Forms.Binding) objects within the `MultiBinding` instance.
- `Converter`, of type `IMultiValueConverter`, which represents the converter to use to convert the source values to or from the target value.
- `ConverterParameter`, of type `object`, which represents an optional parameter to pass to the `Converter`.

The `Bindings` property is the content property of the `MultiBinding` class, and therefore does not need to be explicitly set from XAML.

In addition, the `MultiBinding` class inherits the following properties from the `BindingBase` class:

- `FallbackValue`, of type `object`, which represents the value to use when the multi-binding is unable to return a value.
- `Mode`, of type [`BindingMode`](xref:Xamarin.Forms.BindingMode), which indicates the direction of the data flow of the multi-binding.
- `StringFormat`, of type `string`, which specifies how to format the multi-binding result if it's displayed as a string.
- `TargetNullValue`, of type `object`, which represents the value that is used in the target wen the value of the source is `null`.

A `MultiBinding` must use a `IMultiValueConverter` to produce a value for the binding target, based on the value of the bindings in the `Bindings` collection. For example, a [`Color`](xref:Xamarin.Forms.Color) might be computed from red, blue, and green values, which can be values from the same or different binding source objects. When a value moves from the target to the sources, the target property value is translated to a set of values that are fed back into the bindings.

> [!IMPORTANT]
> Individual bindings in the `Bindings` collection can have their own value converters.

The value of the `Mode` property determines the functionality of the `MultiBinding`, and is used as the binding mode for all the bindings in the collection unless an individual binding overrides the property. For example, if the `Mode` property on a `MultiBinding` object is set to [`TwoWay`](xref:Xamarin.Forms.BindingMode.TwoWay), then all the bindings in the collection are considered `TwoWay` unless you explicitly set a different `Mode` value on one of the bindings.

## Define a IMultiValueConverter

The `IMultiValueConverter` interface enables custom logic to be applied to a `MultiBinding`. To associate a converter with a `MultiBinding`, create a class that implements the `IMultiValueConverter` interface, and then implement the `Convert` and `ConvertBack` methods:

```csharp
public class AllTrueMultiConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        if (values == null || !targetType.IsAssignableFrom(typeof(bool)))
        {
            return false;
            // Alternatively, return BindableProperty.UnsetValue to use the binding FallbackValue
        }

        foreach (var value in values)
        {
            if (!(value is bool b))
            {
                return false;
                // Alternatively, return BindableProperty.UnsetValue to use the binding FallbackValue
            }
            else if (!b)
            {
                return false;
            }
        }
        return true;
    }

    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        if (!(value is bool b) || targetTypes.Any(t => !t.IsAssignableFrom(typeof(bool))))
        {
            // Return null to indicate conversion back is not possible
            return null;
        }

        if (b)
        {
            return targetTypes.Select(t => (object)true).ToArray();
        }
        else
        {
            // Can't convert back from false because of ambiguity
            return null;
        }
    }
}
```

The `Convert` method converts source values to a value for the binding target. Xamarin.Forms calls this method when it propagates values from source bindings to the binding target. This method accepts four arguments:

- `values`, of type `object[]`, is an array of values that the source bindings in the `MultiBinding` produces.
- `targetType`, of type `Type`, is the type of the binding target property.
- `parameter`, of type `object`, is the converter parameter to use.
- `culture`, of type `CultureInfo`, is the culture to use in the converter.

The `Convert` method returns an `object` that represents a converted value. This method should return:

- `BindableProperty.UnsetValue` to indicate that the converter did not produce a value, and that the binding will use the `FallbackValue`.
- `Binding.DoNothing` to instruct Xamarin.Forms not to perform any action. For example, to instruct Xamarin.Forms not to transfer a value to the binding target, or not to use the `FallbackValue`.
- `null` to indicate that the converter cannot perform the conversion, and that the binding will use the `TargetNullValue`.

> [!IMPORTANT]
> A `MultiBinding` that receives `BindableProperty.UnsetValue` from a `Convert` method must define its [`FallbackValue`](xref:Xamarin.Forms.BindingBase.FallbackValue) property. Similarly, a `MultiBinding` that receives `null` from a `Convert` method must define its [`TargetNullValue`](xref:Xamarin.Forms.BindingBase.TargetNullValue) propety.

The `ConvertBack` method converts a binding target to the source binding values. This method accepts four arguments:

- `value`, of type `object`, is the value that the binding target produces.
- `targetTypes`, of type `Type[]`, is the array of types to convert to. The array length indicates the number and types of values that are suggested for the method to return.
- `parameter`, of type `object`, is the converter parameter to use.
- `culture`, of type `CultureInfo`, is the culture to use in the converter.

The `ConvertBack` method returns an array of values, of type `object[]`, that have been converted from the target value back to the source values. This method should return:

- `BindableProperty.UnsetValue` at position `i` to indicate that the converter is unable to provide a value for the source binding at index `i`, and that no value is to be set on it.
- `Binding.DoNothing` at position `i` to indicate that no value is to be set on the source binding at index `i`.
- `null` to indicate that the converter cannot perform the conversion or that it does not support conversion in this direction.

## Consume a IMultiValueConverter

A `IMultiValueConverter` is consumed by instantiating it in a resource dictionary, and then referencing it using the `StaticResource` markup extension to set the `MultiBinding.Converter` property:

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:DataBindingDemos"
             x:Class="DataBindingDemos.MultiBindingConverterPage"
             Title="MultiBinding Converter demo">

    <ContentPage.Resources>
        <local:AllTrueMultiConverter x:Key="AllTrueConverter" />
        <local:InverterConverter x:Key="InverterConverter" />
    </ContentPage.Resources>

    <CheckBox>
        <CheckBox.IsChecked>
            <MultiBinding Converter="{StaticResource AllTrueConverter}">
                <Binding Path="Employee.IsOver16" />
                <Binding Path="Employee.HasPassedTest" />
                <Binding Path="Employee.IsSuspended"
                         Converter="{StaticResource InverterConverter}" />
            </MultiBinding>
        </CheckBox.IsChecked>
    </CheckBox>
</ContentPage>    
```

In this example, the `MultiBinding` object uses the `AllTrueMultiConverter` instance to set the [`CheckBox.IsChecked`](xref:Xamarin.Forms.CheckBox.IsChecked) property to `true`, provided that the three [`Binding`](xref:Xamarin.Forms.Binding) objects evaluate to `true`. Otherwise, the `CheckBox.IsChecked` property is set to `false`.

By default, the [`CheckBox.IsChecked`](xref:Xamarin.Forms.CheckBox.IsChecked) property uses a [`TwoWay`](xref:Xamarin.Forms.BindingMode.TwoWay) binding. Therefore, the `ConvertBack` method of the `AllTrueMultiConverter` instance is executed when the [`CheckBox`](xref:Xamarin.Forms.CheckBox) is unchecked by the user, which sets the source binding values to the value of the `CheckBox.IsChecked` property.

The equivalent C# code is shown below:

```csharp
public class MultiBindingConverterCodePage : ContentPage
{
    public MultiBindingConverterCodePage()
    {
        BindingContext = new GroupViewModel();

        CheckBox checkBox = new CheckBox();
        checkBox.SetBinding(CheckBox.IsCheckedProperty, new MultiBinding
        {
            Bindings = new Collection<BindingBase>
            {
                new Binding("Employee1.IsOver16"),
                new Binding("Employee1.HasPassedTest"),
                new Binding("Employee1.IsSuspended", converter: new InverterConverter())
            },
            Converter = new AllTrueMultiConverter()
        });

        Title = "MultiBinding converter demo";
        Content = checkBox;
    }
}
```

## Format strings

A `MultiBinding` can format any multi-binding result that's displayed as a string, with the `StringFormat` property. This property can be set to a standard .NET formatting string, with placeholders, that specifies how to format the multi-binding result:

```xaml
<Label>
    <Label.Text>
        <MultiBinding StringFormat="{}{0} {1} {2}">
            <Binding Path="Employee1.Forename" />
            <Binding Path="Employee1.MiddleName" />
            <Binding Path="Employee1.Surname" />
        </MultiBinding>
    </Label.Text>
</Label>
```

In this example, the `StringFormat` property combines the three bound values into a single string that's displayed by the [`Label`](xref:Xamarin.Forms.Label).

The equivalent C# code is shown below:

```csharp
Label label = new Label();
label.SetBinding(Label.TextProperty, new MultiBinding
{
    Bindings = new Collection<BindingBase>
    {
        new Binding("Employee1.Forename"),
        new Binding("Employee1.MiddleName"),
        new Binding("Employee1.Surname")
    },
    StringFormat = "{0} {1} {2}"
});
```

> [!IMPORTANT]
> The number of parameters in a composite string format can't exceed the number of child `Binding` objects in the `MultiBinding`.

When setting the `Converter` and `StringFormat` properties, the converter is applied to the data value first, and then the `StringFormat` is applied.

For more information about string formatting in Xamarin.Forms, see [Xamarin.Forms String Formatting](string-formatting.md).

## Provide fallback values

Data bindings can be made more robust by defining fallback values to use if the binding process fails. This can be accomplished by optionally defining the [`FallbackValue`](xref:Xamarin.Forms.BindingBase.FallbackValue) and [`TargetNullValue`](xref:Xamarin.Forms.BindingBase.TargetNullValue) properties on a `MultiBinding` object.

A `MultiBinding` will use its [`FallbackValue`](xref:Xamarin.Forms.BindingBase.FallbackValue) when the `Convert` method of an `IMultiValueConverter` instance returns `BindableProperty.UnsetValue`, which indicates that the converter did not produce a value. A `MultiBinding` will use its [`TargetNullValue`](xref:Xamarin.Forms.BindingBase.TargetNullValue) when the `Convert` method of an `IMultiValueConverter` instance returns `null`, which indicates that the converter cannot perform the conversion.

For more information about binding fallbacks, see [Xamarin.Forms Binding Fallbacks](binding-fallbacks.md).

## Nest MultiBinding objects

`MultiBinding` objects can be nested so that multiple `MultiBinding` objects are evaluated to return a value through an `IMultiValueConverter` instance:

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:DataBindingDemos"
             x:Class="DataBindingDemos.NestedMultiBindingPage"
             Title="Nested MultiBinding demo">

    <ContentPage.Resources>
        <local:AllTrueMultiConverter x:Key="AllTrueConverter" />
        <local:AnyTrueMultiConverter x:Key="AnyTrueConverter" />
        <local:InverterConverter x:Key="InverterConverter" />
    </ContentPage.Resources>

    <CheckBox>
        <CheckBox.IsChecked>
            <MultiBinding Converter="{StaticResource AnyTrueConverter}">
                <MultiBinding Converter="{StaticResource AllTrueConverter}">
                    <Binding Path="Employee.IsOver16" />
                    <Binding Path="Employee.HasPassedTest" />
                    <Binding Path="Employee.IsSuspended" Converter="{StaticResource InverterConverter}" />                        
                </MultiBinding>
                <Binding Path="Employee.IsMonarch" />
            </MultiBinding>
        </CheckBox.IsChecked>
    </CheckBox>
</ContentPage>
```

In this example, the `MultiBinding` object uses its `AnyTrueMultiConverter` instance to set the [`CheckBox.IsChecked`](xref:Xamarin.Forms.CheckBox.IsChecked) property to `true`, provided that all of the [`Binding`](xref:Xamarin.Forms.Binding) objects in the inner `MultiBinding` object evaluate to `true`, or provided that the `Binding` object in the outer `MultiBinding` object evaluates to `true`. Otherwise, the `CheckBox.IsChecked` property is set to `false`.

## Use a RelativeSource binding in a MultiBinding

`MultiBinding` objects support relative bindings, which provide the ability to set the binding source relative to the position of the binding target:

```xaml
<ContentPage ...
             xmlns:local="clr-namespace:DataBindingDemos">
    <ContentPage.Resources>
        <local:AllTrueMultiConverter x:Key="AllTrueConverter" />

        <ControlTemplate x:Key="CardViewExpanderControlTemplate">
            <Expander BindingContext="{Binding Source={RelativeSource TemplatedParent}}"
                      IsExpanded="{Binding IsExpanded, Source={RelativeSource TemplatedParent}}"
                      BackgroundColor="{Binding CardColor}">
                <Expander.IsVisible>
                    <MultiBinding Converter="{StaticResource AllTrueConverter}">
                        <Binding Path="IsExpanded" />
                        <Binding Path="IsEnabled" />
                    </MultiBinding>
                </Expander.IsVisible>
                <Expander.Header>
                    <Grid>
                        <!-- XAML that defines Expander header goes here -->
                    </Grid>
                </Expander.Header>
                <Grid>
                    <!-- XAML that defines Expander content goes here -->
                </Grid>
            </Expander>
        </ControlTemplate>
    </ContentPage.Resources>

    <StackLayout>
        <controls:CardViewExpander BorderColor="DarkGray"
                                   CardTitle="John Doe"
                                   CardDescription="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla elit dolor, convallis non interdum."
                                   IconBackgroundColor="SlateGray"
                                   IconImageSource="user.png"
                                   ControlTemplate="{StaticResource CardViewExpanderControlTemplate}"
                                   IsEnabled="True"
                                   IsExpanded="True" />
    </StackLayout>
</ContentPage>
```

In this example, the `TemplatedParent` relative binding mode is used to bind from within a control template to the runtime object instance to which the template is applied. The `Expander`, which is the root element of the [`ControlTemplate`](xref:Xamarin.Forms.ControlTemplate), has its `BindingContext` set to the runtime object instance to which the template is applied. Therefore, the `Expander` and its children resolve their binding expressions, and [`Binding`](xref:Xamarin.Forms.Binding) objects, against the properties of the `CardViewExpander` object. The `MultiBinding` uses the `AllTrueMultiConverter` instance to set the `Expander.IsVisible` property to `true` provided that the two [`Binding`](xref:Xamarin.Forms.Binding) objects evaluate to `true`. Otherwise, the `Expander.IsVisible` property is set to `false`.

For more information about relative bindings, see [Xamarin.Forms Relative Bindings](relative-bindings.md). For more information about control templates, see [Xamarin.Forms Control Templates](~/xamarin-forms/app-fundamentals/templates/control-template.md).

## Related links

- [Data Binding Demos (sample)](/samples/xamarin/xamarin-forms-samples/databindingdemos)
- [Xamarin.Forms String Formatting](string-formatting.md)
- [Xamarin.Forms Binding Fallbacks](binding-fallbacks.md)
- [Xamarin.Forms Relative Bindings](relative-bindings.md)
- [Xamarin.Forms Control Templates](~/xamarin-forms/app-fundamentals/templates/control-template.md)
