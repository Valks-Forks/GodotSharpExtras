# GodotSharpExtras
C# Extras for Godot, provides tools that are not currently implemented by the Core GodotSharp library.  One being the _onready_ keyword.  Instead, this is replaced with using two C# attributes, called NodePath() and ResolveNode() attributes, that allows easily to reference a variable to their proper node in the Scene / SceneTree.

# Installation
To install this library, simply use dotnet command line tool, to add a package reference for GodotSharpExtras, using the following command:

```
dotnet add package GodotSharpExtras
```

# Compilation
In order to compile a custom version of this project, you will need the GodotSharp libraries from an existing Project which can be found under ```.mono``` folder.  In order to make it easier to build, simply copy the .mono folder from one of your projects, directly into the root of this folder, and then run the following command:

```
dotnet build
```

This will compile the DLL file, as well as create a Nuget package, that you can install locally, or you can simply copy the DLL file over to your project, under any folder you want, and add the following to your project's csproj file.

```xml
<ItemGroup>
    <Reference Include="GodotSharpExtras">
        <HintPath>Path\To\GodotSharpExtras.dll</HintPath>
    </Reference>
</ItemGroup>
```

## OnReady()
This function is what wires everything up from the Godot side of things, to the C# Side of things. In order for C# to be able to get the nodes it needs to reference with the proper C# Variables, OnReady() needs to be called.  By the Godot documentation, it isn't recommended to use `this` when calling various functions on the node, but due to the nature in which the system needs to wire things up, it needs a reference to the node that we are running the OnReady() function on to be able to ensure that the variables are properly setup with the values.  This is done through the C#'s Extension Methods.  This allows us to add OnReady() to a node reference, without having to recompile GodotSharp in order to add this method.  See examples for NodePath and ResolveNode on how OnReady() works.

## NodePath
NodePath will use a String Path to the Node that you want to access from the Godot side of things, in C#.  It takes care of actually getting the instance when you call the OnReady() function in C#.

Example:
```cs
using Godot;
using Godot.Sharp.Extras;

public class MyNode : Container {
    [NodePath("Container/Label")]
    Label MyLabel;
    
    public override void _Ready() {
      this.OnReady();
      
      MyLabel.Text = "Hello World!";
    }
}
```

## ResolveNode
ResolveNode allows an exported variable to the editor, to receive a NodePath, that can still be resolved to an actual node used within the C# code, as a reference to the node in the same way that NodePath is used.  The difference is, the NodePath is an Assignment from the Editor, or code, instead of it being a string path to the node itself.

Example:
```cs
using Godot;
using Godot.Sharp.Extras;

public class MyNode : Node2D {
  [Export]
  public string SpritePath;
  
  [ResolveNode(nameof(SpritePath))]
  Sprite PlayerSprite;
  
  public override void _Ready() {
    this.OnReady();
    
    PlayerSprite.Location = new Vector2(120,120);
  }
}
```

## SignalHandler
SignalHandler allows you to connect node signals to methods to automatically be connected when OnReady is executed.  You provide the Signal you wish to connect, along with the property or field of the variable holding the Node reference, to connect said signal.  If the property or field does not exist, will throw an exception.

Example:
```cs
using Godot;
using Godot.Sharp.Extras;

public class MyNode : Node2D {
    [NodePath("VBoxContainer/MyButton")]
    Button _myButton = null;

    public override void _Ready() {
      this.OnReady();
    }

    [SignalHandler("pressed", nameof(_myButton))]
    void OnPressed_MyButton() {
      GD.Print("Hello World!");
    }
}
```

Example 2 (Connect to the node itself):
```cs
using Godot;
using Godot.Sharp.Extras;

public class MyControl : Control
{
    [NodePath("ColorRect")]
    ColorRect _color = null;
    
    public override void _Ready()
    {
        this.OnReady();
    }
    
    [SignalHandler("mouse_entered")]
    void OnMouseEntered() {
        _color.Color = new Color(0,1,0);
    }
    
    [SignalHandler("mouse_exited")]
    void OnMouseExited() {
        _color.Color = new Color(0,0,1);
    }
}
```

## Caveats
When compiling using NodePath, and ResolveNode, the compiler will give a warning about a variable associated with these two attributes, will never be assigned to, or have a value.  This is because the compiler doesn't recognize that we are using the Extension Method OnReady() to actually assign values to these variables, the way to surpress this warning, is simply to assign null to the value of the variable, to surpress this warning.

Example:
```cs
using Godot;
using Godot.Sharp.Extras;

public class MyNode : Container {
  [Export]
  public Label MyLabel;
  
  public override void _Ready() {
    this.OnReady();
    
    MyLabel.Text = "Hello World";
  }
}
```

Will Generate this warning:

```
C:\MyProject\MyNode.cs (6,15): warning CS0649: Field 'MyNode.MyLabel' is never assigned to, and will always have its default value null [C:\MyProject\MyProject.csproj]
```

While doing this will solve the issue:
```cs
using Godot;
using Godot.Sharp.Extras;

public class MyNode : Container {
  [Export]
  public Label MyLabel = null;
  
  public override void _Ready() {
    this.OnReady();
    
    MyLabel.Text = "Hello World";
  }
}
```

Future items will be added as they are worked on.
