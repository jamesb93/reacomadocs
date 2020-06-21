# parameters.lua

You might want to create parameters for your custom process to facilitate rapid prototyping and to make it more flexible. A part of the ReaComa `lib` helps facilitate the management of parameters and gives you some functionality for free such as storage, recall and defaults. For each 'default' ReaCoMa script I've mirrored the defaults that can be found in the Max or Supercollider documentation while also making it so that once the user changes a given parameter it will use that as the new default for each execution of the script. You don't have to make your scripts work this way though and its entirely up to you how you deal with parameters depending on how you structure your code. Let's break down the `fluid-noveltyslice.lua` example on how parameters work within the `lib`.

For each process that you want to have defaults for you'll need to create an *'archetype'*. These are created in the `params.lua` file inside the `params.archetype` table. The archetype for `fluid-noveltyslice.lua` looks like this.

```lua
noveltyslice = {
    name = "fluidParamNoveltySlice",
    feature = "0",
    fftsettings = "1024 -1 -1",
    filtersize = "1",
    kernelsize = "3",
    threshold = "0.5",
    minslicelength = "2",
}
```

The only essential component is that you give your archetype a unique `name` value. Everything else is up to you including the name of parameters and the data type for the paramete you hold. I've implemented them as strings because it makes sense for forming command line calls but you're welcome to use any native lua type you want, it depends how you use these parameters later. The `name` value gives your archetype an identity for storage in the REAPER extended states API.

In your code you first want to create an instance of your parameter archetype.

```lua
local processor = reacoma.params.archetype.noveltyslice
```

And then you check whether or not this archetype has been used before for your REAPER install.

```lua
reacoma.params.check_params(processor)
```

You can then check against the names and values of the `processor` to observe and use the defaults in whatever way you see fit. In the default ReaCoMa scripts I heavy handedly redefine the names of the parameters as a comma separated string which is used to parse the parameters and to get stored state of that archetype's values if it exists. I do it this way because I want the order of parameters to be deterministic, as lua tables are hash tables and not ordered so reliably iterating over the tables parameters and values in the same order each call is not possibly. I also didn't want to add the extra overhead of implementing an ordered strucutre which makes the code more terse and less readable.

In the simplest form you can extract a default like this:

```lua
parameter = processor["parameter_name"]
```

and store it in an extended state like so:

```lua
reaper.SetExtState(processor.name, parameter_name, value)
```

I perform a somewhat complex dance between extracting the values, formatting them as comma seperated strings and storing/recalling from the extended state if it exists so that I can then pass this information to `reaper.GetUserInputs()`. This function uses comma separated strings exclusively, otherwise everything would implemented as native lua tables.

To prompt the user with some parameters and default values these are the essential parts you need.

```lua
local param_names = "feature,threshold,kernelsize,filtersize,fftsettings,minslicelength"
local param_values = reacoma.params.parse_params(param_names, processor)
local confirm, user_inputs = reaper.GetUserInputs("Noveltyslice Parameters", 6, param_names, param_values)
```

If the user confirms this dialogue box (in the case that they cancel or click away to cancel the action) we then go on to extract whatever they stored in the GUI and store this back in the extended states that populated the GUI in the first place.

```lua
reacoma.params.store_params(processor, param_names, user_inputs)
```

You can also extract what the user selected by splitting the comma separated string that is returned by `reaper.GetUserInputs()` to then pass on values to the command line calls that are formed later on. For example:

```lua
local params = reacoma.utils.commasplit(user_inputs)
local feature = params[1]
local threshold = params[2]
local kernelsize = params[3]
local filtersize = params[4]
local fftsettings = params[5]
local minslicelength = params[6]
```

This last bit is why there is so much faffing and so many conversions between tables and comma separated strings. We need to know the order of the parameters at all times so that we can appropriately structure the command line calls and to ensure that we don't assign the `threshold` to the `-feature` parameter for example.










