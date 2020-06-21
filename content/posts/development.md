---
title: ReaCoMa
type: docs
---

# Developing using the ReaCoMa library

## ReaCoMa-lib

The first thing to understand the small lua library that underpins the orchestration of ReaCoMa processes. This can be found in the subdirectory 'lib' of the main repository. In this directory are several lua files with names describing their function. It's worth having a dig into these files to become more familiar with some of the functions and tables that are defined there.

## reacoma.lua
The first major component that you should look at is `reacoma.lua`. This is the entry point to the library itself and should be imported at the top of any script you are developing. The conventional way of doing this is to use `loadfile()` which will parse the lua file but not execute it. In the below example I use the strong pattern matching in lua to get the location of the script that I'm currently developing and construct the full path of the `reacoma.lua` file from there.

```lua
local info = debug.getinfo(1,'S');
local script_path = info.source:match[[^@?(.*[\/])[^\/]-$]]
loadfile(script_path .. "lib/reacoma.lua")()
```

Calling `loadfile()` to load `reacoma.lua` is similar to a Pythonic `import reacoma` for example. Once you've done this you have access to all of the library functions, variables and data structures and the conventional way to then utilise these is to orchestrate whatever functionality you need around these imported items.

Now we'll go into more detail about the two types of processes: layers and slicing which will give you the understanding to make your own archetypes.

{{< button relref="/posts/dev/layers">}}Layers{{< /button >}}
