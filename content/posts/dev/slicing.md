# slicing.lua

## The slicing container
The slicing container is very similar to the layers container containing empty containers that are ready to fill with relevant information for processing. What is unique to the slicing container is a blank internal table to hold slicing indices named `slice_points_string`. This is where we hold information on where to segment the source media item.

```lua
slicing.container = {
    full_path = {},
    item_pos = {},
    item_pos_samples = {},
    take_ofs = {},
    take_ofs_samples = {},
    item_len_samples = {},
    cmd = {},
    slice_points_string = {},
    tmp = {},
    item = {},
    reverse = {},
    sr = {},
    playrate = {}
}
```

## Populating the container
Populating our data container is the same as what we have seen already. We create an instance of it and then use a function to grab information for each selected media item.

```lua
local data = reacoma.slicing.container

for i=1, reaper.CountSelectedMediaItems(0) do
    reacoma.layers.get_data(i, data)
end
```

## Executing the command line calls
Again, the process for executing the command line calls mirrors what we have already seen. We have to form some strings that are stored and executed on the command line.

```lua
local cmd = exe .. 
" -source " .. reacoma.utils.doublequote(data.full_path[i]) .. 
" -indices " .. reacoma.utils.doublequote(data.tmp[i]) .. 
" -maxfftsize " .. reacoma.utils.getmaxfftsize(fftsettings) ..
" -maxkernelsize " .. kernelsize ..
" -maxfiltersize " .. filtersize ..
" -feature " .. feature .. 
" -kernelsize " .. kernelsize .. 
" -threshold " .. threshold .. 
" -filtersize " .. filtersize .. 
" -fftsettings " .. fftsettings .. 
" -minslicelength " .. minslicelength ..
" -numframes " .. data.item_len_samples[i] .. 
" -startframe " .. data.take_ofs_samples[i]
table.insert(data.cmd, cmd)
```

We then loop and execute a call for each media item selected and grab the result which is stored in a CSV file, appending it to the `slice_points_string` table inside our `data` container.

```lua
for i=1, num_selected_items do
    reacoma.utils.cmdline(data.cmd[i])
    table.insert(data.slice_points_string, reacoma.utils.readfile(data.tmp[i]))
    reacoma.slicing.process(i, data)
end
```

There is a bit of wizardry to coerce the CSV outputs and to account for multiple channels being output to the CSV format which is why its wrapped up in the `reacoma.utils.readfile()` function.

Once each call has been completed we segment the media items according to the slice_points_string data that was returned to us using the `reacoma.slicing.process()` function. I would encourage you to dig into this function which can be found in `slicing.lua`. It's quite involved and takes cares off some edge cases that can arise from the slicing processes, such as:

- No slices being made
- Ensuring that boundary slices exist
- Calculating slices on time stretched items
- Inverting the slicing values in time for reversed items

## Updating the arrangement view
Once we've segmented the items we need to update the arrangement view. If you don't do this the changes won't propogate visually but actually are effective nonetheless.

```lua
reacoma.utils.arrange("reacoma-noveltyslice")
reacoma.utils.cleanup(data.tmp)
```

`reacoma.utils.cleanup()` is called to remove all of the temporary CSV files that are made as interim storage for slice information. This is pointed towards `data.tmp` which contains the full path for each file that was made in the whole process.

Let's move on to how the parameter system works in the `lib`.

{{< button relref="/posts/dev/parameters">}}Parameters{{< /button >}}
