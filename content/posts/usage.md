# Usage

## How to execute the scripts

Once you have followed all the steps of the installation process you can simply execute one of the scripts by doing the following.

1. `Shift + /` (or as I like to call it **`?`**) to bring up the Action Menu
2. Select the action `ReaScript: Run Reascript (EEL or Lua)`
3. Select the desired script which you have cleverly stored in an easy to find location.

All of the scripts are named to match up the corresponding algorithm. For example, to do harmonic percussive separation, you would need to run the `fluid-hpss.lua` script.

The first time that you execute any of the scripts you will be asked to provide a path to the location of the command line tools. Dont worry, this only happens once per REAPER installation. If you decided to put the executables in `/usr/local/bin` then provide that path to the pop-up box.

## What will happen?

### Slicing

So far, all of the slicing based algorithms will directly segment media items in the arrangement. This is not destructive and is like calling the `Split` action multiple times (`S`). You can undo just like any other action and your original media item is restored to its previously unsegmented state. The numerical results are stored in temporary csv files and are deleted as soon as the process is completed.

### Objects/Layers

Any algorithm that should produce new media files, `nmf`, `hpss`, `transients` or `sines` output to a location set by ReaCoMa. By default, new files are located within the same directory as the source material. You can modify this behaviour by changing the value for `reacoma.output` in `config.lua`.

{{< hint info >}}

There are three options for changing the ReaCoMa output.

#### "source" 
New files will be output to the same directory as the source

#### "media"
New files will be output to the REAPER Media folder.

#### custom path  
New files will be output to the path set. The path needs to be absolute, but ~ expansion is supported.

{{< /hint >}}

The results of the process will be appended to a take of the source material. `fluid-sines.lua` for example will append 2 new takes to the original source item containing the sinusoidal and residual extractions respectively. My rationale for implementing this behaviour is so that you can quickly audition the results or replace source audio with your output process without too much fiddling. The outlier in this case is `fluid-nmf.lua` which will append a multichannel take containing the components asked for by the user.

### Dealing with the output

I would recommend you become familiar with two actions:

1. `explode multichannel audio to...`
2. `explode takes of items to new tracks`

Again, these can be found in the action menu **(`?`)**. This will allow you to explode the output of a FluCoMa process to new tracks for more fine tuned manipulation.