# Open Immediate Mode GUI Standard Proposal

Written by Gek in 2021

simple demo available in my TinyGL repository

https://www.github.com/gek169/TinyGLPlusPlus.git

There are a vast array of so-called "portable" and so-called "flexible" UI libraries out there.

Many are flawed because they do not support arbitrary renderers, while
others are flawed because they have far too many widgets and methods to implement.

Fundamentally, to implement any kind of graphical user interface of any complexity,
you need the following functionality:
1) The ability to render text.
2) The ability to render a box (Styled with an image, color scheme gradient, whatever)
3) The ability to render a box that fits text within it
4) The ability to handle "hints" about how to render boxes and text.
5) The ability to detect if a cursor (x,y coordinate) intersects with a box
6) The ability to have a virtual cursor which can move between buttons using keys.
7) The ability to detect a "cursor button" being pushed.
8) The ability to detect text input (for text boxes) (Implementation specific)

Using only this functionality, it is variably difficult to implement all common GUI elements:
1) All forms of button. (trivial)
2) All forms of checkbox.
3) Textboxes- if the user has clicked while the cursor is above a textbox, redirect text input 
	to the textbox until a click is registered outside the textbox.
	If the textbox is not editable, do not allow the user to type in the box.
4) Sliders- Varies depending on cursor input mode.
	a) for direct cursor input such as a mouse:
		A button which follows the mouse cursor between two coordinates,
		after being clicked and before the click is released.
	b) for keyboard input/directional keys:
		A button which moves a small fixed amount when directional keys are pressed in the corresponding directions.
5) Images- A box or text box with an image decoration.
6) Lists- A text box containing a textual list.

Not only does this library require zero initialization, it keeps very little state- only recent user inputs.

Note that all coordinates used by the library are NORMALIZED!

You must implement these methods:
```c
int omg_box(
//Box top left corner (Normalized, 0,0 is top left, 1,1 is bottom right)
float x, float y, 
//Width and height (Also normalized!)
float xdim, float ydim, 
//sucks- indicates whether or not this gui element can "suck" up the cursor in button-controlled modes.
//Static text boxes should not suck up the cursor.
			int sucks, 
//Button jump- How far should the keymode cursor jump in X and Y to reach neighboring buttons?
//Note that if the cursor jumps outside the bounds of the screen,
//it will wrap around to the other side.
			float buttonjumpx, float buttonjumpy, 
//Implementation-specific hints for drawing this box.
			int hints);

//This function is explained below in the example.
int omg_textbox(float x, float y, const char* text, int textsize, int sucks, float buttonjumpx, float buttonjumpy, int hints, int hintstext);
```

You must poll user inputs like this

```c
if(using_cursorkeys)
			omg_update_keycursor(upkeystae, downkeystate, leftkeystate, rightkeystate, Abuttonkeystatae);
		else{
			omg_update_mcursor(mouse_normalized_x, mouse_normalized_y, touchstate);
		}
```
Where states are 1 if the button is currently being held, and 0 if it is not.

Polling of user inputs should happen before any boxes or textboxes are rendered.

in the file where you are going to implement OpenIMGUI's methods, you should 

```c
#define OPENIMGUI_IMPL
```

before including the header.


when rendering gui elements (textboxes, boxes) the return value tells you whether or not the cursor intersects the box (or is highlighting it)

You should NOT use this to activate click events, but rather hover events. 

You should test omg_cb AND the return value of rendering your gui elements.

Here is an example of using omg_textbox:

```c
if(
	omg_textbox(
//textbox coordinates (Width and height are automatically determined from text.)
				tbcoords.d[0], 
				tbcoords.d[1], 
//text and text size hint. The handling of hints is implementation specific
				"Click me and I toggle color!", 16, 
//sucks, buttonjumpx, buttonjumpy
				1, 0.4, 0.4, 
//hints for the box and the text respectively. Hint handling is implementation specific.
				0xFFFFFF, haveclicked?0xFF0000:0x00) 
	&& omg_cb == 1
	)
		{puts("Detected click! EVENT FIRED!\n");haveclicked = !haveclicked; }
```

omg_cb is 1 on the frame that the user clicks/presses the button.

omg_cb is 2 on the frame that the user releases the mousebutton or gamepad menu select button.

otherwise it is zero.

```
Authored by Gek/ DMHSW on March 3rd, 2021

Licensed to you under the CC0 license.
```
