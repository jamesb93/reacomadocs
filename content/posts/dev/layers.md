# layers.lua
Let's break down the essential components of `layers.lua`. 


## The container

The `layers.container` is a table that represents the lifetime of a layers type process.

```lua
layers.container = {
    full_path = {},
    take = {},
    item_pos = {},
    item_pos_samples = {},
    take_ofs = {},
    take_ofs_samples = {},
    item_len = {},
    item_len_samples = {},
    cmd = {},
    item = {},
    sr = {},
    playrate = {},
    playtype = {},
    reverse = {},
    outputs = {},
}
```

It contains information such as the full path in your file system of media items and REAPER specific information such as the position, length, sample rate, playback rate and reverse status. In addition to this it contains a value named `cmd` which acts as a container for command line calls which are stored as strings. Most importantly it contains a value named 'outputs' which is programatically appended to with the outputs of each FluCoMa command-line process. For example, if you ran the `fluid-sines.lua` script the `sines` and `residual` outputs would be inserted to this field creating an internal structure like so:

```lua
outputs = {
    sines = "sines_path.wav",
    residual = "residual_path.wav"
}
```

Taking `fluid-sines.lua` as an example, we can see that an instance of the `layers.container` is made at the start of the script.

```lua 
local data = reacoma.layers.container
```

To populate the `layers.container` with information we'll use `layers.get_data()`. This function programatically retrieves and stores information from selected media items into an instance of the layers container. If you observe the source code, these functions accept two arguments. The first argument is a value which pertains to the item that you have selected and the second argument is the name of the instance of your `layers.container`.

For example, to store the data of the third selected item you would call this series of commands.
```lua
local data = reacoma.layers.container
reacoma.layers.get_data(3, data)
```

Conventionally, we wrap this function in a loop that iterates over all of the selected media items so that each item is processed separately. So something like this:

```lua
local data = reacoma.layers.container
for i=1, reaper.CountSelectedMediaItems(0) do
    reacoma.layers.get_data(i, data)
end
```

The data relevant to the first media item is stored inside the `data` and is ordered, so you can access this data by index when needed.

The next stage is to use this data to orchestrate the FluCoMa processes and update the arrangement view of your REAPER session. The first step in doing so, is to form the command line calls that will be executed for each REAPER media item.

In the `fluid-transients.lua` example this too is managed inside of the loop that iterates over selected media items, at each iteration using the index of the selected item to access the relevant data from the `data` container such as the length of the item and the take offset in samples.

```lua
table.insert(
    data.cmd, 
    exe .. 
    " -source " .. reacoma.utils.doublequote(data.full_path[i]) .. 
    " -transients " .. reacoma.utils.doublequote(data.outputs.transients[i]) .. 
    " -residual " .. reacoma.utils.doublequote(data.outputs.residual[i]) ..
    " -order " .. order ..
    " -blocksize " .. blocksize ..
    " -padsize " .. padsize ..
    " -skew " .. skew ..
    " -threshfwd " .. threshfwd .. 
    " -threshback " .. threshback ..
    " -windowsize " .. windowsize .. 
    " -clumplength " .. clumplength ..
    " -numframes " .. data.item_len_samples[i] .. 
    " -startframe " .. data.take_ofs_samples[i]
)
```

We insert this massive string into the `cmd` table, again in an ordered fashion giving it an association between its position in the table and the item number that is selected. In theory, media item number 4 will be stored at `data.cmd[4]`.

The `exe` variable is created almost at the very start of the script and is formed by concatenating the known path to the binary files which were set at installation with the name of the binary itself.

```lua
local exe = reacoma.utils.doublequote(
    reacoma.settings.path .. "/fluid-noveltyslice"
)
```

{{< hint warning >}}
In this example you can see variables (`order`, `blocksize`, `windowsize` etc.) that I haven't accounted for or explained. I'll get back around to explaining how these parameters are gathered in the [parameters]({{< relref "/posts/dev/parameters" >}}) section.
{{< /hint >}}

## Executing the command line calls

This part is actually very simple. We re-loop over the number of selected items and use the strings stored in `data.cmd` as the argument to the `reacoma.utils.cmdline()` function. This will produce wave file outputs which can then be integrated into the arrangement view.

```lua
for i=1, reaper.CountSelectedMediaItems(0) do
    reacoma.utils.cmdline(data.cmd[i])
end
```

## Updating the arrangement view

The last piece of the puzzle is to take all of the files that were output from the command line calls and attach them as takes to the original source media items.

For layers type processes this is wrapped up neatly in the `reacoma.layers.process()`.

```lua
layers.process = function(item_index, data)
    if item_index > 1 then reaper.SetMediaItemSelected(data.item[item_index-1], false) end
    reaper.SetMediaItemSelected(data.item[item_index], true)
    for k, v in orderedPairs(data.outputs) do
        reaper.InsertMedia(data.outputs[k][item_index], 3)
        local item = reaper.GetSelectedMediaItem(0, 0)
        local take = reaper.GetActiveTake(item)
        local src = reaper.GetMediaItemTake_Source(take)
        reaper.SetMediaItemTakeInfo_Value(take, "D_PLAYRATE", data.playrate[item_index])
        reaper.SetMediaItemTakeInfo_Value(take, "I_PITCHMODE", data.playtype[item_index])
        if data.reverse[item_index] then reaper.Main_OnCommand(41051, 0) end
    end
end
```

This function seems complex on the surface but is easily broken down into some simple essential components with some extra decorations. It takes an item_index and a data container as arguments with the convention that the function is placed within a loop that iterates over selected media items. The function could be reduced to some key parts that excludes some of the fancy functions I've implemented, such as taking care of copying the source item's playback rate and time stretching algorithms.

```lua
layers.process = function(item_index, data)
    reaper.SetMediaItemSelected(data.item[item_index], true)
    for k, v in orderedPairs(data.outputs) do
        reaper.InsertMedia(data.outputs[k][item_index], 3)
    end
end
```

This rapid loop simulates selecting a media item `reaper.SetMediaItemSelected(data.item[item_index], true)` and enumerating over the outputs stored in `data.outputs` at the `item_index`. For each item that is enumerated `reaper.InsertMedia(data.outputs[k][item_index], 3) is called which appends the file to the source media item.

The arrangement view is then updated by the function `reacoma.utils.arrange()` which does two things.

1. Create the beginning of an 'undo block' via `reaper.Undo_BeginBlock(undo_msg)`. This is similar to creating a restore point. Every `reaper` call after this is recorded in the block until `reaper.Undo_EndBlock()` is called.
2. Update the arrangement view via `reaper.UpdateArrange()`
3. Close off the undo block with `reaper.Undo_EndBlock()` taking the argument `undo_msg` as the unique identifier for the undo block.

That covers the essential lifetime of a layers process, disregarding the way in which specific algorithm parameters are corralled into each script. Parameter management is quite generic and independent from the algorithm so this will be covered separately.

{{< button relref="/posts/dev/slicing">}}Slicing{{< /button >}}
