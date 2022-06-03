# ArcadeMacro
Tutorial on how to create and use macros on ArcadeZero

# Introduction

This document serves as both a beginner's guide and a reference document on how to use and create macros for ArcadeZero v3.3 and later. If you are not interetested in creating macros, but only using ready-made ones, feel free to skip any code here. But for those who wishes to create their own macros this guide will assume basic familiarity with the scripting language used - Lua.

I highly suggest that you will read through the whole thing anyway, to be able to troubleshoot any macros yourself. If you're new or would like a refresher on the basic of Lua, please refer to the official tutorial: https://www.lua.org/pil/contents.html

###### Contribution: You can help translate this document to other languages!

# Getting started

Tools you will need:
- ArcadeZero v3.3 or later, or any forks of ArcadeZero that supports macros
- For using ready-made macros: .lua macro scripts obtained from other sources.
- For creating macros: a sufficiently good IDE / text editor is recommended (such as VSCode)

This document will be divided into two parts:
- Using macro: where concepts and basic usages are explained
- Creating macro: how to create your own macro
- Reference: a comprehensive of all available classes and methods

# Part 1. Using macros

### 1. What are macros?
Macros are external, customizable scripts made for automating tasks. They are essentially mini-programs that will run within Arcade, and will tell Arcade what to do based on different actions and inputs from the user.

Macros are great for common tasks that are tedious and time consuming to do manually but its steps can easily be described. Examples include: Offsetting notes position, splitting arcs into segments, generating decorative traces, etc.


### 2. How do I install a macro?
macros are defined within a single `.lua` script file (support for other script file referencing is coming soon), and they must be placed within the `Macros` folder of your Arcade installation. For example, if your Arcade is installed at `D:\Programs\Arcade\`, then place your `.lua` script, `mirror.lua` for example, in: `D:\Programs\Arcade\Macros\mirror.lua`.

Within your ArcadeZero, locate the Macro button on the bottom right corner (the button with a puzzle piece icon). The Macro selection menu should show up. You will see any registered macros in here, and if you don't, press the refresh button (top right of the menu).

### 3. How do I use a macro?
On Arcade's startup, and any time you choose to refresh the macro list, the program will look for all valid `.lua` script within the `Macros` folder, as located above. You can click on the name of the macro you want to use and the macro will start. Exactly what happens next depends on the script (that's the whole point!), so if you aren't sure please contact the script's writer(s).

Any invalid script will prompt an error message on the screen. If you see one, that just means that that macro will not be usable. Either fix it yourself, or again, contact the script's writer(s) for support.

# Part 2. Creating macros

If you saw the documentation for Arcade's Scenecontrol, we'll be doing something similar here! We'll walk through a few example, with detailed explanation on what the code does, and some caveats you have to take note of.

This section is only meant to get the reader familiar with the process and logic of writing a macro script. If you want a comprehensive list of all things you can do with a macro, refer to Part 3.

## Example 1. Hello world

This extremely basic example will teach you the basic structure of a macro. There's quite a lot to be unpacked here actually!

```lua
-- helloWorld.lua
addMacro("helloWorld", function()
    notify("Hello World!")
end)
```

Let's see what this does first! Be sure to save this file in the `Macros` folder of your ArcadeZero installation folder, then boot up ArcadeZero. Open your ArcadeZero, and select the macro `helloWorld` as instructed in 2.2 and 2.3. You should see the a toast notification (top middle) displaying the text "Hello World!".

Let's unpack what's happening here.

All macros will be registered into Arcade with the function `addMacro`. Note that you can have as many of these `addMacro()` as you want, and Arcade will register all of them.
-  The first parameter specify the macro name, that will display as buttons within the Macro selection menu. In this case, it's `helloWorld`
-  The second parameter is actually a **function**, which gives instruction on what should happen each time the macro is run. You might not be very familiar with the concept of passing functions as argument into other functions, but this is the only instance that you will need to do this, so don't worry too much
- Within the function body, we give a single instruction: notify the user with the message "Hello World!"

> NOTE! Macro names must be unique. If you try to create another macro with "helloWorld" within the same script, or in another script, an error will be thrown and only the first macro will be accepted.

For the exploratives out there, try these out for yourself:
- Passing numbers to `notify`
- Passing more than 1 arguments to `notify`
- Try `log` instead of `notify`
- Try joining strings with `..` (look up what it does in the lua tutorial!)
- Removing the macro (try to look it up in part 3!)

## Example 2. Creating notes

We'll jump right into the next task, which will probably be the most common task you'll do when working with macros - creating notes.

The last example didn't require opening an actual project. But since you'll be working with a chart file here, prepare a blank test project to play around.

Let's look at a simple script first:
```lua
-- addNote.lua
addMacro("addNote", function()
    local tap = Event.tap(1000, 1, 0)
    -- A tap note at 1000ms, lane 1, timingGroup 0
    local command = tap.save()
    -- Command is an action of saving the tap note
    command.commit()
    -- Actually save the tap note
end)
```

That was quite a lot of steps wasn't it, but trust me they all have a purpose. In fact you will probably never write a macro to create a single note. We'll see what all those steps are for in a bit, but for now, I'll have you fail on purpose.

> FYI the above can be shortened to
> ```lua
> -- addNote.lua
> addMacro("addNote", function()
>     tap = Event.tap(1000, 1, 0)
>     tap.save().commit()
> end)
> ```
> or even
> ```lua
> -- addNote.lua
> addMacro("addNote", function()
>     Event.tap(1000, 1, 0).save().commit()
> end)
> ```
> Which do the same thing but with less code.

Now let's try adding multiple notes, and while we're at it, let's create other note types as well

```lua
-- addNote.lua
addMacro("addNote", function()
    local tap = Event.tap(1000, 1, 0)
    local command = tap.save()
    command.commit()

    local hold = Event.hold(1000, 2000, 1, 0)
    -- A hold note at 1000ms, last until 2000ms, lane 1, timingGroup 0
    hold.save().commit()

    local arc = Event.arc(
        2000, 0, 1, -- Start at 2000ms, at x=0,y=1
        3000, 1, 1, -- End at 3000ms, at x=1, y=1
        false,      -- Is an arc
        0,          -- Color blue
        's',        -- Type s
        0           -- Timing group 0
    )
    arc.save().commit()

    local trace = Event.arc(
        3000, 1, 1, -- Start at 3000ms, at x=1,y=1
        4000, 0, 0, -- End at 4000ms, at x=0, y=0
        true,       -- Is a trace
        0,          -- Color blue
        'sisi',     -- Type sisi
        0           -- Timing group 0
    )
    -- FYI you don't have to add so many line breaks,
    -- but I still recommend keeping your code easy to read.
    trace.save().commit()

    local arctap = Event.arctap(2500, trace)
    -- Arctaps are a bit tricky! They are naturally bound to a trace
    -- So you must create a trace before creating an arctap
    arctap.save().commit()
end)
```

There you go, all 5 main note types. Go ahead and run the macro, and you should see 5 notes appear in your chart. Perfect! (right?)

Okay here's the problem. Your current macro created 5 separate undo items, so it takes 5 times pressing CTRL+Z to undo everything your macro did. Imagine if you want to write a macro that changes hundreds of notes at once, that'd be nightmarish to use.

Let's fix that right now!


```lua
-- addNote.lua
addMacro("addNote", function()
    local batchCommand = Command.create()
    local tap = Event.tap(1000, 1, 0)
    batchCommand.add(tap.save())

    local hold = Event.hold(1000, 2000, 1, 0)
    batchCommand.add(hold.save())

    local arc = Event.arc(
        2000, 0, 1, 
        3000, 1, 1, 
        false,     
        0,        
        's',     
        0       
    )
    batchCommand.add(arc.save())

    local trace = Event.arc(
        3000, 1, 1, 
        4000, 0, 0, 
        true,      
        0,        
        'sisi',  
        0       
    )
    batchCommand.add(trace.save())

    -- You can also shorten like this
    batchCommand.add(
        Event.arctap(3500, trace).save()
    )
    batchCommand.commit()
end)
```

Run it, and you should see that one undo will delete all 5 notes! Similar for redo as well.


Try these out for yourself:
- Use a for loop to add notes
- Try different parameters for notes. Be careful with timingGroups though, as specifying an invalid timing group will cause an error
- Pass a string into `Command.create()` (e.g. `Command.create("test macro")`).

## Example 3. Self destruct button

Our next example is utterly useless. This macro will send your chart to the heaven realm by wiping every single note out of existence. *(not really since you have backups, but just for fun)*

You'll need to do two things here: getting all the notes in the chart, and deleting them. Let's look at the script:

```lua
-- sakuzyo.lua
addMacro("sakuzyoBeam", function()
    local allNotes = Event.query(EventSelectionConstraint.create().any())
    local batchCommand = Command.create("SELF DESTRUCTION")

    for i = 1, #allNotes["tap"], 1 do
        batchCommand.add(allNotes["tap"][i].delete())
    end

    for i = 1, #allNotes["hold"], 1 do
        batchCommand.add(allNotes["hold"][i].delete())
    end

    for i = 1, #allNotes["arc"], 1 do
        batchCommand.add(allNotes["arc"][i].delete())
    end

    for i = 1, #allNotes["arctap"], 1 do
        batchCommand.add(allNotes["arctap"][i].delete())
    end

    for i = 1, #allNotes["timing"], 1 do
        batchCommand.add(allNotes["timing"][i].delete())
    end

    for i = 1, #allNotes["camera"], 1 do
        batchCommand.add(allNotes["camera"][i].delete())
    end
    
    batchCommand.commit()
end)
```

A few things worth noticing here
1. You use `Event.query` to gather the notes within the active chart file.
2. You must pass into `Event.query` a `EventSelectionConstraint`, which will limit what notes will be returned to us. Here we specify as `any()`, which means anything goes.
3. The query returns 6 different arrays of 6 different event types. You can access them with `["{type}"]`
4. Here we're looping over all of them and add a delete command for each note to the batch command, hence deleting everything. Note a special exception for timing: you can't delete timing with starting time of 0ms (Arcade will just ignore your request).

You'll see `EventSelectionConstraint` again later on, so try get familiar with them now.

## Example 4. Create notes, but somewhere else!

We'll go back to example 2. This time instead of creating notes at 1000ms, let's allow specifying where we will create the notes.

```lua
-- addNote.lua
addMacro("addNoteParam", function()
    local request = TrackInput.RequestTiming() -- Ask user to specify a timing
    coroutine.yield()                          -- Wait for response

    timing = request.result["timing"]
    local batchCommand = Command.create()
    local tap = Event.tap(timing, 1, 0)
    batchCommand.add(tap.save())

    local hold = Event.hold(timing, timing + 1000, 1, 0)
    batchCommand.add(hold.save())

    local arc = Event.arc(
        timing + 1000, 0, 1, 
        timing + 2000, 1, 1, 
        false,     
        0,        
        's',     
        0       
    )
    batchCommand.add(arc.save())

    local trace = Event.arc(
        timing + 2000, 1, 1, 
        timing + 3000, 0, 0, 
        true,      
        0,        
        'sisi',  
        0       
    )
    batchCommand.add(trace.save())

    -- You can also shorten like this
    batchCommand.add(
        Event.arctap(timing + 2500, trace).save()
    )
    batchCommand.commit()
end)
```

Let's focus on the snippet that matters the most here
```lua
    local request = TrackInput.requestTiming() -- Ask user to specify a timing
    coroutine.yield()                          -- Wait for response
    local timing = request.result["timing"]
```

This snippet's purpose is to get input from the macro user, specifically, where the user wants to create our pattern at. `TrackInput.RequestTiming()` will switch Arcade into timing input mode, where the user have to click on the track (similar to how you click on a track to create a tap note, for example), to specify the timing.

Since the user will take some time to do so, and your lua code runs immediately, we use `coroutine.yield()` to suspense the macro's execution. Arcade will resume our macro once the user has inputted the requested information (or in this case, after they clicked on the track where they want to create our pattern).

The `request` variable is where Arcade will pass that information to our macro, and after `coroutine.yield()` we can be sure that that information is valid, and we proceed with retrieving it with `timing = request.result["timing"]`. That timing variable now contain the timing that the user specified.

The rest is simple! We just offset our pattern with whatever `timing` is. Go ahead and give the macro a try!

Try these out for yourself:
- Try `TrackInput.requestPosition(timing)`. This is a bit tricker to use, you have to pass in the timing parameter which will specify where the vertical input plane will be placed. Normally you use a `TrackInput.requestTiming()` then feed the result to `TrackInput.requestPosition(timing)`
- Try requesting and waiting twice

### Example 3.5 Self destruction button, but a bit less destructive

Let's apply the same "request & wait" steps back to example 5. This time we'll create a confirmation box, to make sure that our user didn't misclick and accidentally wipe their chart file for no reason.

This time we're using `DialogInput`
```lua
-- sakuzyo.lua
addMacro("sakuzyoBeam", function()

    local codeField = 
        DialogField.create("code")
            .setLabel("Confirmation")
            .setTooltip("Your confirmation code is 61616")
            .setHint("INPUT CONFIRMATION CODE")
            .textField(FieldConstraint.create().integer())
    local request = 
        DialogInput
            .withTitle("INITIATING SELF DESTRUCTION")
            .requestInput({codeField})

    coroutine.yield()

    local inputtedCode = request.result["code"]
    if inputtedCode != "61616" then
        notify("Destruction sequence shut down")
        return
    end

    local allNotes = Event.query(EventSelectionConstraint.create().any())
    local batchCommand = Command.create("SELF DESTRUCTION")

    for i = 1, #allNotes["tap"], 1 do
        batchCommand.add(allNotes["tap"][i].delete())
    end

    for i = 1, #allNotes["hold"], 1 do
        batchCommand.add(allNotes["hold"][i].delete())
    end

    for i = 1, #allNotes["arc"], 1 do
        batchCommand.add(allNotes["arc"][i].delete())
    end

    for i = 1, #allNotes["arctap"], 1 do
        batchCommand.add(allNotes["arctap"][i].delete())
    end

    for i = 1, #allNotes["timing"], 1 do
        batchCommand.add(allNotes["timing"][i].delete())
    end

    for i = 1, #allNotes["camera"], 1 do
        batchCommand.add(allNotes["camera"][i].delete())
    end
    
    batchCommand.commit()
end)
```

Dialog boxes are considerably more complicated. For once you can create multiple field within the same dialog box, and then read values from all of them. Second you have to make sure our user can only input the right format (like integer only, or maybe we allow both integer and decimal but not alphabetical characters...). These complications are all dealt with in `DialogField`

This snippet
```lua
    local codeField = 
        DialogField.create("code")
            .setLabel("Confirmation code")
            .setTooltip("Your confirmation code is 61616")
            .setHint("INPUT CONFIRMATION CODE")
            .textField(FieldConstraint.create().integer())
```
reads: `codeField` is a field of a dialog box, it's created with an identification *key* of "code", the *label* of the field is "Confirmation code", when user hovers over it, the *tooltip* displays as "Your confirmation code is 61616", and when nothing is inputted yet, the *hint* (gray text that disappears once something is typed in) displays as "INPUT YOUR CONFIRMATION CODE". This field is a *text field*, with a *constraint* that the content must be an *integer*.

This snippet
```lua
    local request = 
        DialogInput
            .withTitle("INITIATING SELF DESTRUCTION")
            .requestInput({codeField})
```
similar to example 4, will show our dialog to the user. This dialog has a title of "INITIATING SELF DESTRUCTION", and has one field, which is our `codeField`!

> NOTE! You can create more than one field within a single dialog. Example
> ```lua
>   DialogInput.withTitle("Test").requestInput({field1, field2, field2})
> ```
> Also there are dropdown and checkbox fields as well, check out the reference document!

It might sound daunting but just mess around with it in Arcade and see what everything does. You'll get the hang of it sooner or later!

```lua
    coroutine.yield()

    local inputtedCode = request.result["code"]
    if inputtedCode != "61616" then
        notify("Destruction sequence shut down")
        return
    end
```
Finally our code waits for user input, then it retrieves the inputted code through the key "code" (we chose this key in `DialogField.create("code")`). Then the rest is standard at this point.


### Example 5. amygdata

Our next example is going to be practical, and is actually a lot easier than example 3.5. We'll make an amygdata arc converter, right within Arcade!

```lua
addMacro("amygdata", function()
    local request = EventSelectionInput.requestSingleEvent(
        EventSelectionConstraint.create().solidArc()
    ) -- Ask user to select a single arc note

    coroutine.yield() -- Wait for response

    local arc = request.result["arc"][1]

    batchCommand = Command.create("conversion to amygdata")
    local arcLength = Context.beatLengthAt(arc.timing) / Context.beatlineDensity

    -- Create the arc
    for timing = arc.timing, arc.endTiming, arcLength do
        endTiming = math.min(timing + arcLength, arc.endTiming)

        if (math.abs(endTiming - timing) <= 1) then break end

        startXY = arc.positionAt(timing)
        endXY = arc.positionAt(endTiming)

        batchCommand.add(
            Event.arc(
                timing,
                startXY,
                endTiming, 
                endXY,
                false,
                arc.color,
                arc.type,
                arc.timingGroup
            ).save()
        )
    end

    batchCommand.add(
        arc.delete()
    )

    batchCommand.commit()
end)
```

The most noteworthy part is again, the user input part
```lua
    local request = EventSelectionInput.requestSingleEvent(
        EventSelectionConstraint.create().solidArc()
    ) -- Ask user to select a single arc note
      -- voidArc() will limit to traces only

    coroutine.yield() -- Wait for response

    local arc = request.result["arc"][1]

    ...

    batchCommand.add(
        arc.delete()
    )
```

We're this time asking the user to select a single arc note from the chart, as a form of asking for input values. We then retrieve the arc's timing, position, etc., then create our on arcs based on those information. Finally we delete the inputted arc, and our conversion is complete!

> Please note that you still have to specify `request.result["arc"][1]` to gather the arc's data. `request.result` alone won't work.

The last thing to point out is the `Context` class, which provides very useful information about the current chart and settings, such as BPM, currently active arc color, currently active arc type, etc. Be sure to check out the reference document for everything you can do with this class!

Try this out for yourself:
- Convert this macro to process multiple arcs at once (hint: `EventSelectionInput.requestEvents(..)`)
- Try to make it work for traces as well (hint: `.arc()` combines `.solidArc()` and `.voidArc()`)
- This is not quite amygdata arc yet! If the arc's starting y coordinate and ending y coordinate is the same, then no "arc beams" will be created. Try expanding on this!

### A few extra tips
- Remember to always use `local` for assigning local variable
- Error message can be hard to read within Arcade. You can always open the `Error Log` (middle left hand side)
- If you plan to share your macros, consider prefixing your macro with something unique (my own macros follow the format `zero.{category}.{name}`) to avoid collision
- You can create macro within macro. Not sure if this is useful

# Part 3. Reference

## 1. Global functions and classes
### 1.1. Functions
Function|Description|Output
-|-|-
addMacro(string macro, function macroDef)|Register a macro|Nil
removeMacro(string macro)|Unregister a macro|Nil
log(object content)|Output content to the log file|Nil
notify(object content)|Output content to the toast notification|Nil
xy(number x, number y)|Return an XY coordinate with specified x y value|XY
toNumber(string s)|Convert string to a number, or 0 if fails|number
toBool(string s)|Convert string to a bool, or false if fails|bool
### 1.2. Context
Static property|Description|Type
-|-|-
offset|The current audio offset of this chart (ms)|number
beatlineDensity|The current beatline density setting of Arcade|number
baseBpm|The current base bpm setting of Arcade|number
songLength|The length of current song (ms)|number
allArcColors|String list of available arc colors (which is just "Blue", "Red", "Green"))|Table (of strings)
currentArcColor|Current default arc color, as an index of the list above|number
allArcTypes|String list of available arc types ("b", "s", ...)|Table (of strings)
currentArcType|Current default arc type|string
currentIsVoidMode|true means creating traces by default, false means creating arcs by default|bool
currentTimingGroup|Currently active timing group|bool
timingGroupCount|Number of timing groups in the current chart|bool


Static method|Description|Output
-|-|-
beatLengthAt(number timing, number timingGroup = 0)|Length of a beat at specified timing and within a timing group (ms)|number
bpmAt(number timing, number timingGroup = 0)|Bpm value at specified timing and within a timing group (ms)|number
divisorAt(number timing, number timinggroup = 0)|Divisor value at specified timing and within a timingg group (ms)|number
### 1.3. XY
Property|Description|Type
-|-|-
x|The horizontal x coordinate (arc unit)|number
y|The vertical y coordinate (arc unit)|number

Method|Description|Output
-|-|-
mirrorX(float axis = 0.5f)|Returns a coordinate flipped horizontally along the specified x coordinate|XY
mirrorY(float axis = 0.5f)|Returns a coordinate flipped vertically along the specified y coordinate|XY
toString()|Returns a string representation. Used for logging|string

Also supports operator overloading
```lua
    xy1 = xy(1,2)
    xy2 = xy(3,4)
    log(xy1 + xy2)
    log(xy1 - xy2)
    log(xy1 * 3)
    log(3 * xy1)
    log(xy1 / 2)
```
## 2. Input methods
### 2.1 TrackInput
Static method|Description|Output
-|-|-
requestTiming(bool showVertical = false, string notification = null)|Request the user to input a timing value (similar to creating a tap note). Optionally enable vertical grid, and provide a different toast notification|TrackRequest
requestPosition(int timing, string notification = null)|Request the user to input a position value (similar to creating an arc note). The grid will be positioned according to the provided timing value. Optionally provide a different toast notification|TrackRequest
requestLane(string notification = null)|Request the user to select a track lane. Optionally provide a different toast notification|TrackRequest
### 2.2 EventSelectionInput
Static method|Description|Output
-|-|-
requestSingleEvent(EventSelectionConstraint constraint)|Request the user to select a single event that satisfies the constraint|EventSelectionRequest
requestEvents(EventSelectionConstraint constraint)|Request the user to select any number of events that satisfies the constraint. The user must press enter to confirm the selection|EventSelectionRequest
### 2.3 DialogInput
Static method|Description|Output
-|-|-
withTitle(string title)|Create a dialog input with specified title|DialogInput

Method|Description|Output
-|-|-
requestInput({DialogField field1, DialogField field2,..})|Build the dialog with the specified list of fields|DialogRequest
## 3. Request Types
### 3.1 TrackRequest
Property|Description|Type
-|-|-
result["timing"]|Returned timing value|number
result["x"]|Returned x coordinate value|number
result["y"]|Returned y coordinate value|number
result["lane"]|Returned lane value|number
### 3.2 EventSelectionRequest
Property|Description|Type
-|-|-
result["tap"]|Returned list of taps|Table (of LuaTap)
result["hold"]|Returned list of holds|Table (of LuaHold)
result["arc"]|Returned list of arcs and traces|Table (of LuaArc)
result["arctap"]|Returned list of arctaps|Table (of LuaArcTap)
result["timing"]|Returned list of timing events|Table (of LuaTiming)
result["camera"]|Returned list of camera events|Table (of LuaCamera)
### 3.3 DialogRequest
Property|Description|Type
-|-|-
result|Map from a field's key to user inputted field value on said field. E.g result["key1"] return input for field DialogField.create("key1")|Table
## 4. DialogField
Property|Description|Type
-|-|-
key|The field's key|string
label|The field's label, appears on the left column of a dialog|string
hint|The field's hint, appears on empty text fields only|string
tooltip|The field's tooltip, appears on hover|string
defaultValue|The field's default value|dynamic
dropdownOptions|The different options for a dropdown menu|Table (of dynamic)
fieldConstraint|The field's constraint for a text field|FieldConstraint

Static method|Description|Output
-|-|-
create(string key)|Create a field with specified key|DialogField

Method|Description|Output
-|-|-
setLabel(string label)|Set the field's label|DialogField
setTooltip(string tooltip)|Set the field's tooltip|DialogField
setHint(string hint)|Set the field's hint|DialogField
defaultTo(string hint)|Set the field's hint|DialogField
textField(FieldConstraint constraint)|Convert field to a text field with specified constraint|DialogField
dropdownMenu(dynamic option1, dynamic option2,...)|Convert field to a dropdown menu with specified constraint|DialogField
checkbox()|Convert field to a checkbox|DialogField
## 5. Events
### 5.0 LuaChartEvent (abstract)
Property|Description|Type
-|-|-
timing|The event's timing (ms)|number
timingGroup|The event's timing group|number
attached|Whether this lua representation has a real note in the chart attached to it|number

Method|Description|Type
-|-|-
copy()|Create a copy of the event (not attached)|LuaChartEvent
save()|Create a command that saves the event to the chart|LuaChartCommand
delete()|Create a command that deletes the event from the chart|LuaChartCommand

Classes from 5.1 to 5.6 inherits all properties and methods mentioned in 5.0.

### 5.1 LuaTap (extends LuaChartEvent)
Property|Description|Type
-|-|-
lane|The tap note's lane|number
### 5.2 LuaHold
Property|Description|Type
-|-|-
endTiming|The hold note's end timing (ms)|number
lane|The hold note's lane|number
### 5.3 LuaArc
Property|Description|Type
-|-|-
startXY|The arc note's starting XY coordinate|XY
endXY|The arc note's ending XY coordinate|XY
endTiming|The arc note's end timing (ms)|number
type|The arc note's arc type ("b", "s", ...)|string
color|The arc note's color|number
isVoid|true means a trace, false means an arc|bool
startX(readonly)|The arc note's starting X coordinate|number
startY(readonly)|The arc note's starting Y coordinate|number
endX(readonly)|The arc note's ending X coordinate|number
endY(readonly)|The arc note's ending Y coordinate|number

Method|Description|Output
-|-|-
positionAt(number timing, bool clamp = true)|Arc's XY coordinate position at timing. Clamping will limit the resulting point to either ends of the arc|XY
xAt(number timing, bool clamp = true)|Arc's x coordinate position at timing. Clamping will limit the resulting point to either ends of the arc|number
yAt(number timing, bool clamp = true)|Arc's y coordinate position at timing. Clamping will limit the resulting point to either ends of the arc|number
### 5.4 LuaArcTap
Property|Description|Type
-|-|-
timing|The arctap note's timing (ms)|number
arc|The parent arc note|LuaArc
### 5.5 LuaTiming
Property|Description|Type
-|-|-
bpm|The timing event's bpm value|number
divisor|The timing event's divisor value|number
### 5.6 LuaCamera
Property|Description|Type
-|-|-
mx|The camera event's x coordinate displacement|number
my|The camera event's y coordinate displacement|number
mz|The camera event's z coordinate displacement|number
rx|The camera event's x axis rotation|number
ry|The camera event's y axis rotation|number
rz|The camera event's z axis rotation|number
## 6. Command classes
### 6.1 Command
Static method|Description|Type
-|-|-
create(string name)|Create an empty chart edit command with specified name|LuaChartCommand
### 6.2 LuaChartComand
Method|Description|Type
-|-|-
add(LuaChartCommand target)|Merge all commands from target into this instance|Nil
commit()|Execute any included commands|Nil
## 7. Constraint classes
### 7.1 FieldConstraint
Static method|Description|Type
-|-|-
create()|Create an default FieldConstraint (that accepts anything)|FieldConstraint

Method|Description|Type
-|-|-
any()|Set constraint to accept all characters|FieldConstraint
float()|Set constraint to accept decimal numbers only|FieldConstraint
integer()|Set constraint to accept integers only|FieldConstraint
gEqual(number value)|Set constraint to accept number greater than or equal to value|FieldConstraint
lEqual(number value)|Set constraint to accept number less than or equal to value|FieldConstraint
greater(number value)|Set constraint to accept number greater than value|FieldConstraint
less(number value)|Set constraint to accept number less than value|FieldConstraint
custom(function dynamic->bool, string message = "Invalid")|Specify a custom constraint. Invalidates all other constraints. Optionally specify a feedback message|FieldConstraint
union(FieldConstraint otherConstraint)|Combines two constraints (similar to an operator OR)|FieldConstraint
getConstraintDescription()|Get the auto-generated description|string
### 7.2 EventSelectionConstraint
Static method|Description|Type
-|-|-
create()|Create an default EventSelectionConstraint (that accepts anything)|EventSelectionConstraint

Method|Description|Type
-|-|-
any()|Set constraint to accept all events|EventSelectionConstraint
tap()|Set constraint to accept tap events only|EventSelectionConstraint
hold()|Set constraint to accept hold events only|EventSelectionConstraint
arc()|Set constraint to accept arc and trace events only|EventSelectionConstraint
solidArc()|Set constraint to accept arc events only|EventSelectionConstraint
voidArc()|Set constraint to accept traces events only|EventSelectionConstraint
arctap()|Set constraint to accept arctap events only|EventSelectionConstraint
timing()|Set constraint to accept timing events only|EventSelectionConstraint
camera()|Set constraint to accept camera events only|EventSelectionConstraint
floor()|Set constraint to accept tap and hold events only|EventSelectionConstraint
sky()|Set constraint to accept arc, trace and arctap events only|EventSelectionConstraint
short()|Set constraint to accept tap and arctap events only|EventSelectionConstraint
long()|Set constraint to accept hold, arc and trace events only|EventSelectionConstraint
judgeable()|Set constraint to tap, hold, arc and arctap events only|EventSelectionConstraint
fromTiming(number timing)|Set constraint to accept events with timing greater than or equal to value|EventSelectionConstraint
toTiming(number timing)|Set constraint to accept events with timing less than or equal to value|EventSelectionConstraint
ofTimingGroup(number group)|Set constraint to accept events within the specified timing group|EventSelectionConstraint
custom(function dynamic->bool, string message = "Invalid")|Specify a custom constraint. Invalidates all other constraints. Optionally specify a feedback message|EventSelectionConstraint
union(FieldConstraint otherConstraint)|Combines two constraints (similar to an operator OR)|EventSelectionConstraint
getConstraintDescription()|Get the auto-generated description|string
