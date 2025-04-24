# Menus & Widgets

This document will cover the most important things to know about how the widget system is set up, and how you can best utilize the tools and functions within.

---
## Overview

There exists a class called `UMainWidget`. This widget is on screen permanently and is the base from which all other widgets are added and shown. As such, 
`AArsenicPlayerController` has a reference to this widget so we can access its functions at any time from anywhere. The various functions that live on this 
widget help with adding and removing widgets to/from the screen, setting input modes, and handling the mouse cursor. Most of that is handled automatically 
when you call `PushWidgetToTargetStack`. The main widget has 4 stacks that you can push menus to: 

- `HUD_Stack` (which is pretty much used only once to add the 
HUD), 
- `InteractableHUD_Stack` (for drag and drop inventories and such), 
- `Menu_Stack` (for menus like a pause menu or settings menu), and the 
- `Popup_Stack` (for context widgets). 

>For more details, go to `UI->Widgets->MainWidget.h`

The `Common UI` plugin has also been set up to detect which input device is being used, and `UCommonActionWidgets` set input icons automatically based on those 
input types. Conveniently, I've created a `UBottomNavBarBase` class that can be added to any `UArsenicMenuBase` (important, it MUST be `UArsenicMenuBase`) that 
contains a button that will handle closing the current widget. It works like you'd expect a back button to.

![WB_NavBarBase Image](img/Menus%20And%20Widgets/bottom%20nav%20bar.png)

---
## Input

When a menu is opened, we must call `SwitchToMenuInput()` on the `ArsenicPlayerController`. This removes the **Gameplay** Input Mapping Context which moves the character, 
and replaces it with the **Menu** Input Mapping Context which helps with navigating menus. When `MainWidget` no longer has any widgets pushed, it broadcasts `OnStacksCleared`
which the `PlayerController` responds to by calling `SwitchToGameplayInput()`. UI inputs are in `_GAME->UI->Input->InputActions`, and gameplay inputs are in 
`_GAME->Blueprints->Player->Input->InputActions`

Widgets can't (and won't) behave as expected when using traditional Input Actions directly in the widget because of a bug in the engine version we're using. Instead, menus 
have to respond to input by responding to delegates broadcast by the `ArsenicPlayerController`. For example, to tab left or tab right, the `ArsenicPlayerController` broadcasts 
that one of those inputs has been pressed. Then, the `Settings` menu responds by changing the widget index. Callbacks are set up in blueprint so we can access various widget 
components more easily, but the broadcasts themselves are handled in C++ by the `ArsenicPlayerController`. 

![WB_SettingsMenu Binding to delegates](img/Menus%20And%20Widgets/BindingToDelegates.png)

`Common UI` supports global back actions, and our menu system makes great use of them. If you're creating a menu that you'll want to close by simply pressing `B` on controller or `ESC`,
inherit from `UArsenicMenuBase`. This automatically responds to back actions, but requires that `IsModal` and `IsBackHandler` are set to `true`. You can find these bools in the widget's 
details panel. When you inherit from `UArsenicMenuBase`, you can also add a `UBottomNavBarBase` widget to add a clickable back button and navigation icons that change based on the current 
input device. 

![IsBackHandler and IsModal bools](img/Menus%20And%20Widgets/IsBackHandler.png)

---
## Focus

`Focus` is used to allow controllers to navigate menus, and is also one of the strongest points of frustration in UMG. We will be dealing with this for the duration of the project, so make 
sure you follow these steps to minimize frustration and maximize functionality.
In the details panel of your widget blueprint:

- Set `IsFocusable` to `true`, and ensure the `DesiredFocusWidget` is set to `self` (or a button depending on your menu)
- `Supports Activation Focus` should be `true`, and depending on your menu, `Auto Restore Focus` should be `true` as well but is not required.
- Ensure the `Input Config` is set to `Game and Menu` and that `Game Mouse Capture Mode` is set to `No Capture`.
- Lastly, in the widget blueprint's event graph, override the `GetDesiredFocusTarget` function and set the return value to either `Self` or a button in your widget hierarchy. Then `OnActivated`, call 
`SetFocus` - the target is `GetDesiredFocusTarget`. You may have to add a small delay to account for widget transition animation time. This is displayed in `WB_PauseMenu` and `WB_SettingsMenu`, with 
the settings menu having tabbed navigation and sub-widget input logic.

![Focus details in WB_SettingsMenu](img/Menus%20And%20Widgets/FocusDetails.png)

![WB_SettingsMenus' OnActivated Event](img/Menus%20And%20Widgets/SetFocusEvent.png)
