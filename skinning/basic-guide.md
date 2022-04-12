---
layout: default
title: Basic Guide
parent: Skinning
---
# Basic skinning guide
{: .no_toc }
In this guide we'll create a basic arrow skin with judgements and other ui elements for 4key mode that you can customize however you like.
> Note: this guide doesn't use 100%-custom elements or moddedgame folder, it lists only things provided by default.

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Basic skin structure
* Skins should be named `<filename>.skin.lua`, for example `4key.skin.lua` and should go in their respective skin folder, for example `skins/example-skin`.
* > Note: if you want to add more than one keymode to your skin, you'll have to create another skin file(s) in the same folder.
* Skin's textures can be located anywhere in the skin folder.
* Things are layered on the screen in the order of their appearance in the skin file.
* To make it all work, skin needs 4 things: necessary functions, noteskin, playfield and `return noteskin` in the end of the file.


## Step 0: Importing functions
Just add those right in the beginning of the file and don't question anything, without them it'll never work.
```lua
local NoteSkinVsrg = require("sphere.models.NoteSkinModel.NoteSkinVsrg")
local BasePlayfield = require("sphere.models.NoteSkinModel.BasePlayfield")
```


## Step 1: Noteskin

### Declaring noteskin
First step in making a skin is, of course, filling out required skin metadata.
Copy this code and edit as you need, explanations below
```lua
local noteskin = NoteSkinVsrg:new({
	path = ...,
	name = "example 4k arrow",
	inputMode = "4key",
	range = {-1, 1},
	unit = 480,
	hitposition = 460,
})
```
#### path
Path where the skin will source image files from.
Default: `...` (same directory)
Examples: `../example` or `.../textures`

#### name
Name of the skin in noteskins menu
Examples: `JustVldKsh's 4k arrow` or `circle`

#### inputMode
Input mode this skin file is for, can contain `key`, `pedal` and `scratch`
Examples: `4key`, `7key1scratch`, `5key1pedal1scratch`

#### range
Idk what does, leave it at `{-1, 1}`

#### unit
Leave it at `480`, it affects scale of the elements on the screen and game pretty much expects it to be equal 480.

#### hitposition
Horizontal position of the note receptors. 0 is the center, + is down and - is up

### Adding and ordering inputs in the skin
To be able to actually use the skin, you need to set inputs the skin accepts. Amount of possible inputs should be the same as the set input mode and be separated by commas, but they can be ordered in any way.
Considering that the skin we're making is for 4key mode, next line in the skin will be `noteskin:setInput({"key1", "key2", "key3", "key4"})`.
Examples for other modes:
```lua
noteskin:setInput({"key1", "key2", "key3", "key4", "key5", "key6", "key7"})`
noteskin:setInput({"scratch1", "key1", "key2", "key3", "key4", "key5"})`
noteskin:setInput({"key1", "key2", "key3", "key4", "key5", "key6", "key7", "scratch1"})`
```

### Adding columns
This step is necessary so the game knows where to draw the conveyor with notes and receptors.
```lua
noteskin:setColumns({
	offset = 0,
	align = "center",
	width = {64, 64, 64, 64},
	space = {32, 0, 0, 0, 32}
})
```
* offset - horizontal shift from the default position of the conveyor. - is left and + is right.
* align - horizontal align of the conveyor, can be `left`, `center` and `right`.
* width - width of each note, number of values should be equal to amount of inputs.
* space - space between notes and both sides of the conveyor, number of values should be equal to amount of inputs + 1 (so if there are 4 inputs then there should be 5 values and so on)

### Adding textures
This step can be skipped if you want to waste your time by typing out paths to the textures further in the skin.
> Note: some things (like judgements) can't be used if added here.
As previously said, textures can be located anywhere in the skin folder (feel free to draw your own textures or if you can't then just grab from another one), so for everything to be clean we'll make following folder structure:
```
+-- judges
|   |-- bad.png
|   |-- good.png
|   |-- great.png
|   |-- marv.png
|   |-- miss.png
|   +-- perf.png
|
|-- notes
|   |-- body.png (ln body)
|   |-- down.png
|   |-- left.png
|   |-- right.png
|   |-- tail.png (ln end)
|   +-- up.png
|
|-- receptors
|   |-- down0.png (released)
|   |-- down1.png (pressed)
|   +-- other receptors with same name pattern
|
|-- 4key.skin.lua
|
+-- pixel.png (mandatory 1x1 white image)
```

To use the images in the skin, copy this:
```lua
noteskin:setTextures({
--mandatory 1x1 white image
	{pixel = "pixel.png"},
--notes
	{body = "notes/body.png"},
	{tail = "notes/tail.png"},
	{left = "notes/left.png"},
	{down = "notes/down.png"},
	{up = "notes/up.png"},
	{right = "notes/right.png"},
})

--if you have images with black non-transparent background then you can add next code
--[[
noteskin:setBlendModes({
    image = {"add", "alphamultiply"}
})
]]

--automatic image-variable assign
noteskin:setImagesAuto()

--manual image-variable assign
--[[
noteskin:setImages({
	pixel = {"pixel"},
	body = {"body"},
	tail = {"tail"},
	left = {"left"},
	down = {"down"},
	up = {"up"},
	right = {"right"},
})
]]
```

### Short notes
```lua
noteskin:setShortNote({
	image = {"left", "down", "up", "right"},
	h = 64,
})
```
* image - image(s) used as notes
* h - height of notes

### Long notes
```lua
noteskin:setLongNote({
	head = {"left", "down", "up", "right"},
	body = "body",
	tail = "tail",
	h = 64,
})
```
* head - start of LN
* body - body
* tail - end
* h - texture height

### Non-mandatory elements

#### Measure line
Can be added but not necessary, removing it is the same as using NML (NoMeasureLine) mod
```lua
noteskin:addMeasureLine({
	h = 4,
	color = {0.5, 0.5, 0.5, 1},
	image = "pixel"
})
```
* color - in 0-1 range, order is red green blue alpha


## Step 2: Playfield
Time to draw the playfield itself starting with creating it first:
```lua
local playfield = BasePlayfield:new({
	noteskin = noteskin
})
```
From now on things will get layered by the order of appearance in the file.

### Drawing notes and receptors
Here we'll draw notes and receptors on the screen itself. If you don't need receptors (for example if you use bottom of the screen instead) then don't add them.
You also can edit camera perspective in this step if you know how to.

```lua
playfield:enableCamera()
--notes
playfield:addNotes()

--receptors
playfield:addKeyImages({
	h = 64,
	padding = 480-noteskin.hitposition,
	pressed = {"receptors/left1.png", "receptors/down1.png", "receptors/up1.png", "receptors/right1.png"},
	released = {"receptors/left0.png", "receptors/down0.png", "receptors/up0.png", "receptors/right0.png"},
})
playfield:disableCamera()
```

* playfield:addNotes adds notes and addKeyImages adds receptors
* h - is obviously receptor height
* padding is tied to hitposition in this example so it moves together with it, 0 is at bottom
* pressed - images when keys are pressed/held
* released - images when keys are released

### Base elements
Base elements can be added (but not mandatory) with `playfield:addBaseElements()`
If you need to add only specific elements then you can use `playfield addBaseElements("progress", "hp", "score")` instead. There are 6 base elements: `progress`, `hp`, `score`, `accuracy`, `combo` and `hit error`.

If you need to customize one or multiple base elements then specify other non-modified elements and use [BasePlayfield.lua](https://github.com/semyon422/soundsphere/blob/master/sphere/models/NoteSkinModel/BasePlayfield.lua) as reference.

### Judgements
It's also possible to add judgements to the skin, which we'll do here.
In this example we'll use osu!mania OD 10 timing windows.
> Note: timing windows are used in seconds, so if the window is 16ms then it'll be 0.016 seconds, mirrored on both positive and negative side.
```lua
playfield:addDeltaTimeJudgement({
	x = 0, y = 540, ox = 0.5, oy = 0.5,
	rate = 2,
	transform = playfield:newLaneCenterTransform(1080),
	judgements = {
		"judges/miss.png",
        -0.121,
        "judges/bad.png",
        -0.097,
		"judges/good.png",
		-0.067,
		"judges/great.png",
		-0.034,
		"judges/perf.png",
		-0.016,
		"judges/marv.png",
		0.016,
		"judges/perf.png",
		0.034,
		"judges/great.png",
		0.067,
		"judges/good.png",
		0.097,
        "judges/bad.png",
        0.121,
        "judges/miss.png"
	}
})
```
* x, y, ox, oy - judgement location, change only x and y
* rate - image scale
* transform - i guess sort of a fancy align, idk actually
* judgements - array of alternating judgement images and timings, center is 0 and image between specified timings is used.
> Note: if you use custom timings.lua then different judgement might appear as worst, for example bad instead of miss, tweak the skin as needed. Also default soundsphere miss window is 160ms.

## Step 3: "Exporting" and troubleshooting
To "export" the skin so the game can actually read the information from it, add `return noteskin` in the end and save the file. Now you can try the skin ingame and if it didn't crashed and works as intended then congrats, you did everything correctly.

But in case it did crashed, you can try few things:
* Make sure you did actually save the file
* Look if you forgot a comma somewhere
* If game tells if table x should have y values, then add/remove values in the corresponding table
* if you still don't have pixel.png in your skin, it can be found in other soundsphere skins or in the resources folder in the game's root.
* If you can't figure out the problem on your own, feel free to ask for help in #help or #skins channels of [soundsphere Discord](https://discord.gg/ubKMtTk).


## Closing thoughts
After following this guide you should end up with a relatively basic soundsphere skin that you can modify as you wish.
If you think you can improve this guide in any way, feel free to [contribute on Github](https://github.com/JustVldKsh/soundsphere-wiki)
_Note: had to leave out adding BGA to the skin from the guide because of page build errors._