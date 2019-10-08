# 1: About LibMayronObjects


* LibMayronObjects is a framework intended on making object-oriented programming easier for Lua developers. 
* You can create classes and call them to instantiate new instances/objects modeled from those classes.
* You can create interfaces that enforce functions to be implemented by classes.
* You can enforce strict typing rules to class and interface function parameters and return values. 
* Each class can inherit from at most one parent class. All classes either directly, or indirectly, inherit from the Object class.
* There are many useful Object functions that all classes inherit and can use.
* You can create and export packages, and import packages or separate classes and interfaces.
* The framework also comes with some standard collection classes (List, Stack, Map, LinkedList).

## 1.1: NOTE:

There is a Test.lua file to see other working examples of how to use the Library!

# 2: Creating a Class

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");

local MyPackage = LibMayronObjects:CreatePackage("MyPackage");

local MyClass = MyPackage:CreateClass("MyClass");

function MyClass:Print(data, message)
    print(message);
end

function MyClass.Static:Foo()
    print("This is a static (non-instance) function!");
end

local Instance = MyClass();
Instance:Print("Hello, World!");
```

Line 3 creates a new package. This package is only available to the developer until they choose to export it. All entities (classes and interfaces) are created using the package object and can be found inside the package. This will be explained in later.

Line 5 creates a new class with the class name "MyClass". No parent was assigned to it (2nd argument), so it directly inherits from the Object class. It does not implement an interface (3rd argument).

Line 7 declares an instance function for this class. It cannot be called directly from the class. It can only be called from an instance of the class. The first argument (data) is automatically given to the function by LibMayronObjects when called. This is the instance's private data that can only be accessed when inside the function body (unless giving explicit access by the developer, usually through getters and setters).

Line 11 shows how to create a static class function. These functions can and should be called directly from the Class table rather than from a class instance.

Line 15 demonstrates how to create an instance from a class; you simply call the class like you would a function.

You may have noticed that line 14 calls the Print method with one string argument. This is passed as the 2nd argument to MyClass.Print declaration because the first argument is always the instance's private data.

# 3: Getters and Setters

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");

local MyClass = MyPackage:CreateClass("MyClass");

function MyClass:GetTimeRemaining(data)
    return data.timeRemaining;
end

function MyClass:SetTimeRemaining(data, timeRemaining)
    data.timeRemaining = timeRemaining;
end

local Instance = MyClass();
Instance:SetTimeRemaining(60);
local timeRemaining = Instance:GetTimeRemaining();
```

Instance private data is passed in as the first argument to any instance (non-static) function call. This data is only accessible from within the function body unless it is made explicitly accessible by the developer who created the class. This can be achieved using getter and setter functions as shown in the example above. An external value is passed to the setter function (SetTimeRemaining) on line 15 and can be retrieved using a getter function as shown on line 16. This technique can help to prevent outside interference against important class logic.

# 4: Function Definitions

This feature can help enforce strict rules upon your defined classes to ensure that parameter arguments and return values of class functions are the expected variable types. Note that including strict typing is completely optional.

Using the same example from the previous section, we can improve on this by adding some strict typing rules/definitions to the SetTimeRemaining instance function:

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");

local MyClass = MyPackage:CreateClass("MyClass");

MyPackage:DefineReturns("number");
function MyClass:GetTimeRemaining(data)
    return data.timeRemaining;
end

MyPackage:DefineParams("number");
function MyClass:SetTimeRemaining(data, timeRemaining)
    data.timeRemaining = timeRemaining;
end

local Instance = MyClass();
Instance:SetTimeRemaining(60);

local timeRemaining = Instance:GetTimeRemaining();
```

A package contains information relating to classes and function definitions, therefore declaring parameter and return value types must be done using the package object.

Lines 10 and 11 must be included straight before declaring the function, otherwise, the instructions for strict typing will not be set to the correct function definition. However, as mentioned previously, you can also choose to not include the definition calls at all as enabling strict typing of parameter and return values is optional. The first argument (the private instance data) can be ignored when creating strict typing definitions. Line 10 states that the first argument passed to the SetTimeRemaining function must be of type "number", which in this case it is. *Any* developer using the class must comply with this enforced rule. Line 11 states that the return value must be of type "number". The developer who created the class must comply with his own promise by returning a string value.

You can also declare that any value should be returned, as long as it is not nil:

```lua
MyPackage:DefineParams("any");
function MyClass:SetTimeRemaining(data, timeRemaining)
    data.timeRemaining = timeRemaining;
end
```

In this case, the return value and first argument can be a table, number, string, boolean or function, but not nil.

## 4.1: Optional Function Parameters and Return Types 

Additionally, you can declare optional parameter arguments and return values by adding a question mark `"?"` character in front of the definition type. However, *they must be declared at the end of the declaration list* after any non-optional definitions:

```lua
MyPackage:DefineParams("string", "number", "?table");
MyPackage:DefineReturns("any", "function", "?boolean");
```

## 4.2: Object and Widget Names as Function Parameters and Return Types

Because all objects inherit the `GetObjectType()` function from the Object class, you can also validate parameter and return values to check whether or not they are of a certain class type:

```lua
MyPackage:DefineParams("Frame", "Button", "?MySlider");
MyPackage:DefineReturns("IHandler", "MyClass", "?SomeWidget");
```

## 4.3: Default Primative Parameter Values

Since update 3.0, you can now add default parameters. If this function is called without any parameters then message will be assigned the value "foo bar" and value will be assigned the value 14:

```lua
MyPackage:DefineParams("string=foo bar", "number=14");
function MyClass:Print(data, message, value)
    print(message, value);
end
```

Of course, if this method was called with 2 values supplied then the default values will be ignored. Using the optional syntax `"?"` at the front of the value type is not required if a default parameter has been specified.

## 4.4: Default Complex Parameter Values

The above approach in section 4.3 works well for simple values that can be parsed from a string, but more complicated parameter types will struggle to be parsed from strings, such as widgets/frames, tables and functions. For this, use the following approach:

```lua
local myDefaultTable = {msg = "foo bar"};

MyPackage:DefineParams({"table", myDefaultTable});
function MyClass:Print(data, tbl)
    print(tbl.msg);
end
```

This code will print "foo bar" if no parameter is passed to the Print function. If a table is supplied then the parameter will use that instead.

You can also use this syntax for default primative types as an alternative for the syntax expressed in section 4.3:

```lua
-- these are functionally equivalent:
MyPackage:DefineParams("number=14");
MyPackage:DefineParams({"number", 14});
```

# 5: Exporting a Package

Entities (such as classes and interfaces) do not need to be exported. These are created from a package and are automatically added to the package. You should only be exporting packages. By default, packages are not made available outside of the file they are created in. To export a package, you need to supply a namespace. A namespace is similar to a key. It is used to locate a package during the import process.

You can export in 2 ways. Using the libraries Export function directly, or when creating the package by supplying the namespace as the 2nd argument instead:

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");
LibMayronObjects:Export(MyPackage, "RootPackage.AnotherPackage");
```

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage", "RootPackage.AnotherPackage");
```

In this example, both "RootPackage" and "AnotherPackage" are other packages, where "AnotherPackage" is a sub-package of "RootPackage". Both of these packages do *not* have to be explicitly created using the CreatePackage function; instead, these are both created during the exporting process. "MyPackage" is now a sub-package of "AnotherPackage".

If a developer wanted to create a class that inherits "MyParent" from another file where no reference to "MyParent" exists, they would either need to supply the full namespace:

```lua
local MyChild = MyPackage:CreateClass("MyChild", "MyPackage.MyClasses.MyParent");
```

Or, import it first using the full namespace and then including the parent class directly to the 2nd argument:

```lua
local MyParent = LibMayronObjects:Import("MyPackage.MyClasses.MyParent");
local MyChild = MyPackage:CreateClass("MyChild", MyParent);
```

# 6: Importing Packages or Entities

Unlike the Export function, the Import function can import not just another package with a valid namespace (a package that has previously been exported will have been assigned a namespace), but it can also import a single entity (a class or interface).

```lua
-- File 1:
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");
LibMayronObjects:Export(MyPackage, "MyAddOn.Widgets");

local CheckButton = MyPackage:CreateClass("CheckButton");
local Button = MyPackage:CreateClass("Button");
local Slider = MyPackage:CreateClass("Slider");
local TextArea = MyPackage:CreateClass("TextArea");
local FontString = MyPackage:CreateClass("FontString");
local Animator = MyPackage:CreateClass("Animator");

-- File 2:
local CheckButton = LibMayronObjects:Import("MyAddOn.Widgets.MyPackage.CheckButton");

local WidgetsPackage =  LibMayronObjects:Import("MyAddOn.Widgets.MyPackage");
local Button = WidgetsPackage:Get("Button");
local Slider = WidgetsPackage:Get("Slider");
local TextArea = WidgetsPackage:Get("TextArea");
local FontString = WidgetsPackage:Get("FontString");
local Animator = WidgetsPackage:Get("Animator");
```

Line 14 shows how you can import just one entity. Lines 16 - 21 show how you can import an entire package and then use the Package's Get function to retrieve individual entities. You can also create new entities to be added to the package that you have imported.

# 7: Creating an Interface

Interfaces can be a great way of specifying rules/patterns for a system. Interfaces cannot inherit or implement other entities. `MyPackage:CreateInterface()` only takes one argument: the interfaces name. Interfaces, like classes, can be both exported and imported.

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");

local IEventHandler = MyPackage:CreateInterface("IEventHandler");

MyPackage:DefineParams("string", "function");
MyPackage:DefineReturns("boolean");
function IEventHandler:Register(eventName, func) end

function IEventHandler:Execute() end

local Frame = MyPackage:CreateClass("Frame", nil, IEventHandler);

function Frame:Register(data, eventName, func)
    -- do stuff
    return true;
end

function Frame:Execute(data)
    -- do stuff
end

local myFrame = Frame();

myFrame:Register("PLAYER_ENTERING_WORLD", function() 
    print("Hello, World!") 
end);
```

Lines 8 and 10 show 2 interface functions being created. Interface functions do not, and should not, have any function body code used for implementation. Instead, classes that are declared as "implementing the interface" must supply the function's implementation.

You do not need to, and should not, redeclare the return and param definitions for functions that have already been defined by the interface. You simply use the same function signature and include the implementation at the class level.

All functions must be implemented for each interface attached to the class. You cannot implement from 2 or more interfaces that share the same function name as this would cause a clash.

You can implement many interfaces, but only inherit from one parent class:

```lua
local MyClass = MyPackage:CreateClass("MyClass", MyParent, IInterface1, IInterface2, IInterface3);
```

## 7.2: Defining Properties for Implementing
You can specify strongly-typed properties that must be implemented inside a constructor

```lua
local MyInterface= TestPackage:CreateInterface("MyInterface");
MyInterface:DefineProperty("MyBoolean", "boolean");

local MyClass= TestPackage:CreateClass("MyClass", nil, MyInterface);

function MyClass:__Construct(data)
    -- must define boolean property:
    self.MyBoolean = true;
end

local myInstance = MyClass();
```

Properties are publically available (not privately stored inside the data table) and must be attached to the instance using the `self` keyword. These properties can also be assigned the `"any"` type so that they are required but the type of value is not important. You can also specify the property type as optional:

```lua
MyInterface:DefineProperty("MyBoolean", "?boolean");
```

This is useful for when a property can either be `nil`, or the expected value type.

# 8: Constructors and Destructors

A class can be assigned a constructor and a destructor. A constructor is a function that is called when an instance is first created from that class. A destructor is called either by the garbage collector if the reference for an instance ceases to exist, or by explicitly calling `Destroy()` on the instance.

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");

local Parent = MyPackage:CreateClass("Parent"); 

function Parent:Talk(data)
    print(data.Dialog);
end

function Parent:__Construct(data)
    data.Dialog = "I am a parent.";
end

local Child = MyPackage:CreateClass("Child", Parent);

function Child:__Construct(data)
    data.Dialog = "I am a child!";
end

function Child:__Destruct(data)
    data.Dialog = nil;
    print("Child object Destroyed!");
end

local child = Child();

child:Talk(); -- prints "I am a child!"

child:Destroy(); -- prints "Child object Destroyed!"
```

The private instance data from the child object is passed to the parent function. This is why line 27 prints "I am a child!" and not "I am a parent.". `__Construct` and `__Destruct` do not need to be manually called. They are implicitly called by LibMayronObjects when required.

# 9: Calling Parent Functions from a Child Instance

An important Object function is `Parent()`. Using parent on an instance object, whose class has a parent class, will cause the parent function to be executed instead. This means that if a child and a parent class both have implementations of the same function (where both functions have the same key/name), the parent function 
will be called *and* the private data sent to the function will be the original, child object.

```lua
local LibMayronObjects = LibStub:GetLibrary("LibMayronObjects");
local MyPackage = LibMayronObjects:CreatePackage("MyPackage");

local SuperParent = MyPackage:CreateClass("SuperParent");
local Parent = MyPackage:CreateClass("Parent", SuperParent);
local Child = MyPackage:CreateClass("Child", Parent);
local SuperChild = MyPackage:CreateClass("SuperChild", Child);

function SuperParent:Print(data)
    print(data.origin == "SuperChild");
    return "This is SuperParent!";
end

function Parent:Print(data)
    return "This is Parent!";
end

function Child:Print(data)
    return "This is Child!";
end

function SuperChild:Print(data)
    return "This is SuperChild!";
end

function SuperChild:__Construct(data)
    data.origin = "SuperChild";
end

local instance = SuperChild();

assert(instance:Print() == "This is SuperChild!");
assert(instance:Parent():Print() == "This is Child!");
assert(instance:Parent():Parent():Print() == "This is Parent!");
assert(instance:Parent():Parent():Parent():Print() == "This is SuperParent!");
```

Line 10 will print "SuperChild", because line 35 passes the private instance data belonging to the SuperChild instance object, through the chain of `Parent()` calls, to the SuperParent class.

All of the assert functions in this example will not trigger an error. This shows that each function implementation is being called correctly.

## 9:2: Calling a Parent Constructor inside a Child Constructor

You can also use the special `Super()`, functiona call as a shortcut for calling the parent constructor. This is useful if every child class of a parent shares similar constructor logic:

```lua
function Parent:__Construct(data, settings)
    data.frame = CreateFrame("Frame");
	data.frame:SetSize(100, 50);
	data.frame:SetPoint(unpack(settings.position));
end

function Child:__Construct(data, settings)
    self:Super(settings); -- this will create the frame	
	data.frame:SetScript("OnEnter", child_OnEnter);
end
```

# 10: Attributes

You can assign attributes to class functions (methods). These are pre-defined classes that run code inside an `OnExecute` function before the class function is called. These `OnExecute` functions are called with the instance, the instance private data, the name of the function to be called, and all parameters the function is going to be called with. Therefore, you can apply pre-function execution logic and evne return `false` to cancel the function call. A useful example of this is to cancel the execution of a function if the player is in combat. An attribute has already been built to add this functionality and can be found in the library inside the `Attributes` folder:

```lua
-- Import the attributes package:
local Attributes = Lib:Import("Framework.System.Attributes");

-- Create an attribute (must implement the IAttribute interface and implement OnExecute)
local InCombatAttribute = Attributes:CreateClass("InCombatAttribute", nil, "IAttribute");

function InCombatAttribute:__Construct(data, executeLater, silent)
    data.executeLater = executeLater;
    data.silent = silent;
end

function InCombatAttribute:OnExecute(data, instance, instanceData, funcName, ...)
    if (InCombatLockdown()) then

        -- This will execute the function when the player comes out of combat!
        if (data.executeLater) then
            functionCalls:Push(Lib:PopTable(instance, funcName, ...));
            return;
        end

        if (data.silent) then
            return false; -- do not call the function
        end

        Lib:Error("Failed to execute %s.%s: Cannot execute while in combat.", instance:GetObjectType(), funcName);
    end

    return true; -- continue executing the function
end
```

Once an attribute has been defined, we can attach it to any function and the OnExecute function will be called before the function it is applied to is called. In this case, if the player is in combat, we return false and the function is not called but will be cached and called after the player exits combat (triggered by the `"PLAYER_REGEN_ENABLED"` event):

```lua
MyPackage:SetAttribute("Framework.System.Attributes.InCombatAttribute");
function MyClass:SensitiveOutOfCombatCode()
    -- calling this code when the player is in combat will cause taint! 
end
```

Of course, similar to `CreateClass`, you can import the attribute and then use the object as the first parameter of `SetAttribute`:

```lua
local inCombatAttribute = Lib:Import("Framework.System.Attributes.InCombatAttribute");

MyPackage:SetAttribute(inCombatAttribute);
function MyClass:SensitiveOutOfCombatCode()
    -- calling this code when the player is in combat will cause taint! 
end
```

# 11: Generic Types

Generic types can be used to specify what type of values an instance of a Generic class can use.
Use the "Of" class method when instantiating an instance to specify what types to use:

```lua
local KeyValuePair = TestPackage:CreateClass("KeyValuePair<K, V>");

TestPackage:DefineParams("K", "V");
function KeyValuePair:Add(data, key, value)
    -- do stuff...
end

local myInstance = KeyValuePair:Of("string", "number")(); 
myInstance:Add("testKey", 123);
```

# 12: Object Functions

As previously mentioned and demonstrated, classes are either assigned a parent class manually or are implicitly given the Object class as their default parent class. This means that all classes inherit from Object directly or indirectly (Object is the parent of the parent's, parent's class, etc...). Therefore, all Object functions can be used by all classes. 

Below is a list of all Object functions (without including the private instance data as the first argument):

## Object:GetObjectType()
```
Returns the class name (string).
```

## Object:IsObjectType(objectName)
```css
@param objectName (string): The class or interface name.

Returns a boolean value indicating if the instance type is directly, or indirectly, the same type as the objectName supplied. If the class of the instance inherits from a parent (or a parent's parent, etc...), or implements an interface whose name is equal to the objectName argument, then the instance is said to be of that type and true will be returned.
```

## Object:Parent()
```
Returns the parent class ready to be used with a parent function. Usually, you cannot call a non-static function on a class object, however the original private instance data from the instance object that called the Parent() function (or chain of Parent() functions) is passed along to the final function call.
```

## Object:Equals(other)
```css
@param other (table): Another LibMayronObjects class (or value).

Returns a boolean value indicating if two instances are equal. Two classes can be equal if all private data key and value pairs match regardless of whether the instance table references are different.
```

## Object:GetParentClass()
```
Returns the Parent Class of the instance.
```

## Object:GetClass()
```
Returns the Class of the instance.
```

## Object:Clone()
```
Returns a new instance whose private data key and value pairs match the original, cloned instance.
```
### Example:
```lua
local Instance2 = Instance:Clone();
print(Instance:Equals(Instance2)); -- prints true
```

## Object:Destroy()
```
Removes all private instance data keys and calls the __Destruct function.
```

# 13: Other Package Functions not Previously Demonstrated

## Package:ProtectEntity(class)
```
If you create a class or interface, you can mark it as protected. Protected entities cannot have functions re-assigned or modified.
```

## Package:AddSubPackage(package)
```
You can add a previously created package to as a sub-package to another package without needing to export either of the packages.
```

## Package:ForEach(func)
```css
@param func (function): Apply a function to all entities found inside the package

Can be used to print all entity names using GetObjectType().
```

## Package:Size()
```
Returns the size of the package, i.e. the total number of items found inside the package.
```

## Package:GetName()
```
Returns the name of the package.
```

# 14: Other LibMayronObjects Functions not Previously Demonstrated

## LibMayronObjects:SetSilentErrors(silent)
```css
@param silent (boolean): LibMayronObjects errors will be added to the error log if set to true, rather than being alerted in-game.
```

## LibMayronObjects:GetErrorLog()
```
Returns the error log (table), consisting of all accomulated errors triggered while LibMayronObjects was set to silent mode (LibMayronObjects:SetSilentErrors()).
```

## LibMayronObjects:FlushErrorLog()
```
Empties the error log.
```

## LibMayronObjects:GetNumErrors()
```
Returns the number of errors collected.
```

## LibMayronObjects:IterateArgs(...)
```
This is a more efficient way of iterating over a variable argument list or for any situation where a table is needed to hold values only for iterating purposes such as:
```
```lua
for id, value in ipairs({...}) do
    -- do stuff
end
```
```
For example, if this for-loop is called every 0.2 seconds then you are creating a new anonymous table each time which results in a memory leak. Use IterateArgs to avoid this:
```
```lua
for id, value in Lib:IterateArgs(...) do
    -- do stuff
end
```

## LibMayronObjects:PopTable()
```
Returns an empty table from an internal table stack (a reusable table)
```

## LibMayronObjects:PushTable(tbl)
```
Add a table to the internal table stack for later a reuse. This function will remove any metatable set to the table and empty it in the process.
```

## LibMayronObjects:UnpackTable(tbl)
```
This will unpack the table (like using the upack function) and place it in the table stack for later reuse.
```