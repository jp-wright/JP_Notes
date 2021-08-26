# Matplotlib Notes
### Colormaps
Colormaps I think are probably the best to choose from:
RdYlBu_r   -- might be a good choice.
inferno   
spectral
jet       -- pretty but visually inaccurate
rainbow   -- similar to jet






### Matplotlib closing plots
pyplot interface

pyplot is a module that collects a couple of functions that allow matplotlib to be used in a functional manner. I here assume that pyplot has been imported as import matplotlib.pyplot as plt. In this case, there are three different commands that remove stuff:

`plt.cla()` clears an axis, i.e. the currently active axis in the current figure. It leaves the other axes untouched.

`plt.clf()` clears the entire current figure with all its axes, but leaves the window opened, such that it may be reused for other plots.

`plt.close()` closes a window, which will be the current window, if not specified otherwise.

Which functions suits you best depends thus on your use-case.

The `close()` function furthermore allows one to specify which window should be closed. The argument can either be a number or name given to a window when it was created using `figure(number_or_name)` or it can be a figure instance fig obtained, i.e., using `fig = figure()`. If no argument is given to `close()`, the currently active window will be closed. Furthermore, there is the syntax `close('all')`, which closes all figures.





### Matplotlib
**IMPORTANT**: Here is a solution to the IPython 5.x and Matplotlib freezing problem…
MPL	| IPy |	Python | Result
----|-----|--------|-------
2.0.0 | 5.1.0 | 3.6	| Freezes upon closing plot window
1.5.3 | 5.1.0 | 3.6 | Freezes upon closing plot window
2.0.0 | 5.1.0 | 2.7 | Freezes upon closing plot window
2.0.0 | 5.1.0 | 3.5.2 | Freezes upon closing plot window
2.0.0 | 4.3.0 | 3.5.2 | Works as normal **\*\***
2.0.0 | 4.2.0 | 2.7 | Works as normal…

**UPDATE #2**
I edited the source code for Matplotlib (O_o) to change the default backend for Python 3.6 from "MacOSX" to "Qt5Agg".  The file I edited is located at `~/anaconda/pkgs/matplotlib-2.0.0-np111py36_0/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc`.  (Note the directory only changes for the version of Python, so the file will always be found at the same spot, `site-packages/matplotlib/mpl-data/matplotlibrc` for each version of Python).  I changed the first command near the top of the file to say `backend      : Qt5Agg` and commented out the `backend      : macosx` command which was in there.

**UPDATE:**
I have found two short-term band-aids to get around the IPython 5.x freezing plot bug!  
#### Solution \#1  
In any `.py` script you are going to `run` in IPython, include the following `import` and command at the **top** of the script:  

```python
import matplotlib as mpl
mpl.use('Qt5Agg')
```

The placement of this `import` and command is critical, because if you place it after other matplotlib imports, the interactive shell will already have imported matplotlib and thus lock in its backend to be used for the current session.  Once a backend has been chosen in a given IPython session, it cannot be changed.  The session must be exited and a new one started before the backend can be changed.  The current default GUI backend for IPython is `MacOSX`.
> UserWarning:  This call to `matplotlib.use()` has no effect because the backend has already been chosen; `matplotlib.use()` must be called *before* `pylab`, `matplotlib.pyplot`,
or `matplotlib.backends` is imported for the first time.  

#### Solution \#2  
If you want to change the MPL backend in an IPython Interactive Shell, you *must* do it before issuing any other MPL commands.  Simply enter:
1. `import matplotlib`  
2. `%matplotlib qt5`  

This must be the first command issued with MPL.  Otherwise you will see the following error message: `Warning: Cannot change to a different GUI toolkit: qt4. Using [current backend] instead.`  In this case, simply `exit` IPython and reopen it, and change the backend as your first command.  Once you've set the backend to `qt5` you can plot as normal or run scripts with plots in them and the window will not freeze.  Note that you have to do this at the start of every Ipython session using this approach, but once you initialize it, you are set for the rest of the session.  

**Note**: `qt5` solves our problem, currently, but there are many backends to choose from. It is likely that multiple of these will work as well.  Use `%matplotlib --list` to get the list of supported backends:  
`auto`, `agg`, `gtk`, `gtk3`, `inline`, `ipympl`, `nbagg`, `notebook`, `osx`, `qt`, `qt4`, `qt5`, `tk`, `wx`.  

We know that `osx` -- the standard default for MPL -- is what was causing the issue with plots freezing in IPython, so avoid it for now.  Also, `inline` and `notebook` are for Jupyter Notebooks, and won't be applicable to standard use in an IPython shell.

In 2019 this default backend for Mac was renamed to `MacOSX`.

To simply see which backend IPython is using for MPL, simply enter `%matplotlib`.  Once you do this, however, it will lock in the current backend, so if you want to change it you have to exit and restart a new IPython session.  

#### Official Doc from MPL on changing defaults
The `matplotlibrc` file:  
matplotlib uses `matplotlibrc` configuration files to customize all kinds of properties, which we call rc settings or rc parameters. You can control the defaults of almost every property in matplotlib: figure size and dpi, line width, color and style, axes, axis and grid properties, text and font properties and so on. matplotlib looks for `matplotlibrc` in four locations, in the following order:

1. `matplotlibrc` in the current working directory, usually used for specific customizations that you do not want to apply elsewhere.

2. `$MATPLOTLIBRC/matplotlibrc`.

3. It next looks in a **user-specific** place, depending on your platform:  
On Linux, it looks in `.config/matplotlib/matplotlibrc` (or `$XDG_CONFIG_HOME/matplotlib/matplotlibrc`) if you’ve customized your environment.  
On other platforms, it looks in `.matplotlib/matplotlibrc`.  
See `matplotlib` configuration and cache directory locations.

4. `INSTALL/matplotlib/mpl-data/matplotlibrc`, where `INSTALL` is something like `/usr/lib/python3.5/site-packages` on Linux, and maybe `C:\Python35\Lib\site-packages on Windows`. On my rMBP it is located at `/Users/jpw/anaconda/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc`, where `/Users/jpw/anaconda/lib/python3.6/site-packages` represents the global `INSTALL` of Anaconda.  

Every time you install `matplotlib`, this file *will be overwritten*, so if you want your customizations to be saved, please move this file to your **user-specific** `matplotlib` directory.  

*JP Note*: If using different virtual environments, you will have to modify this file in each virtual environment's directory as well.  This is not difficult, and the directory only changes slightly to include the specific venv location.  So, say I had a venv named "jp_venv1", I would find the `matplotlibrc` file in the same path as the global one above (\#3) except that it would be modified for the specific venv.  In the case of "jp_venv1" the rc file to modify would be at `/Users/jpw/anaconda/envs/jp_venv1/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc`.  Easy!  

**IMPORTANT**  
To display where the currently active `matplotlibrc` file was loaded from, one can do the following:

```python
>>> import matplotlib
>>> matplotlib.matplotlib_fname()
'/home/foo/.config/matplotlib/matplotlibrc'
```
Use this code to find *which* `matplotlibrc` file you need to modify for the current app you are using.  Upon doing this in a fresh IPython (5.1, global install) shell, I was told the location from which the MPL config was loaded for this IPython session was  
 `/Users/jpw/anaconda/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc`.

More config options can be found [here](http://matplotlib.org/users/customizing.html).

IPython 4.3.0 works with Python 3.5.2 but not Python 3.6.  So if you want Python 3.x and also have MPL work with the IPython interactive shell without modifying any config files or changing the backend, either via *solution \#1* or *solution \#2*, you will have to use the following setup:  
+  MPL 2.0.0  
+  IPython 4.3.0
+  Python 3.5.2  

The main issues with this setup is that you don't get to use IPython 5.x and all of its convenient improvements.  There are numerous posts about IPython 5.x having some bugs with MPL.  Perhaps in a future IPython update (a 5-point if we are lucky) this will be resolved.  Until then, either




Use the following to determine the current backend being used by Matplotlib:
```python
import matplotlib
matplotlib.get_backend()
```
Note that "MacOSX" is commonly the default for MPL.

Use the following to change the backend used by MPL in a script.
```python
import matplotlib as mpl
mpl.use('Qt5Agg')
```

If you want to change the MPL backend in an IPython Interactive Shell, you *must* do it before issuing any other MPL commands.
`%matplotlib [backend of choice]` must be the first command issued with MPL.  Otherwise you will see the following error message: `Warning: Cannot change to a different GUI toolkit: qt4. Using [current backend] instead.`  In this case, simply `exit` IPython and reopen it, and change the backend as your first command.



### Datetime Axes
When doing time series plotting, to use a `datetime` series as an (x) axis you must pass the `.date` attribute only (not the full `datetime` object, to my knowledge).  In my `jp_gas_cubby.py` file, I experienced this first hand.  I had to use the `.date` attribute for the x-axis in `plt.plot()` command for the x-axis to be displayed as dates.  Using `df.plot()` seems to automatically format the axis but this excludes using the advanced niceties of `matplotlib`.  In general, something to look further into, but noting here for future reference.  

Note I believe since `df.plot()` uses `matplotlib` under the hood, that all `time` components of `datetime` objects are excluded when plotting a time series with a `datetime` axis.








### Connected Line/Scatter Plots With Different Face Colors
This drove me crazy one night trying to figure out.
There is no way to change the fill color for markers in a line plot in one command.
Instead, we must plot the line part with `.plot()` and the markers with `.scatter()`.

```python
dfp = self.df[(self.df['season'] == season) & (self.df['team'] == team)].set_index('week')
dfp['colors'] = ['#52BE80' if gm == 1 else '#CD6155' for gm in dfp['won_game']]
wins = dfp[dfp['won_game'] == 1]
losses = dfp[dfp['won_game'] == 0]
fig = plt.figure(figsize=self.figsize_dct[figsize])

plt.plot(dfp.index, dfp[metric].values, lw=2.5, zorder=2, label=team)
for result in [wins, losses]:
    plt.scatter(result.index, result[metric].values, lw=1, marker='o', s=66.5, c=result['colors'], edgecolor='k', zorder=3, label=team)
```
Example:  
![Example Connected Scatter plot](/Users/jpw/Desktop/connected_scatterplot.png)







### Custom Colormaps
```python
def custom_cmap(self, data_range):
    """
    Function to create a custom colormap from any specified dataset.
    Reason: Had a weird issue which I think should be easily overcome, obviating the need for the use of custom colormap.
    Problem: When using a default colormap in the scatterplot arguments, such as c=data_range, cmap='jet', everything works fine when plotting an x and y that correspond to that exact data_range, as usual.  But when choosing to plot only a single point (team) from that data range, and expecting the point's color to correspond to where it falls within the data_range, I couldn't get it to work.  Always gave me the bottom of the color range (i.e. the lowest/first value -- dark blue in the case of 'jet').
    Solution: Create a custom colormap, use cm.jet as the actual colormap and the exact same data_range (totdvoa, in this case) as the variables.  When plotting a single point from that range, MPL honors its position within the colormap and plots correctly.  I think there must be a way to make the default cmap work, I am just missing something about how to slice into it correctly, or some such.  Either way, this works now, so away we go.
    Return: Have to actually return both my_cmap, for the colorbar function, and mymap for the cmap plotting function.
    """

    # Scale the data range of interest by normalizing it and assigning it color values from a provided colormap, cm.jet in this case.  Can actually create own colormap, but that isn't needed, here.
    my_cmap = cm.ScalarMappable(colors.Normalize(np.min(data_range), np.max(data_range)), cm.jet)

    # Must set_array for the given data range, so the colormap has a list of values to plot from.
    my_cmap.set_array(data_range)

    # Take the list of values and assign them to corresponding RGBAlpha values, as determined by the original provided colormap, such as cm.jet.
    mymap = my_cmap.to_rgba(data_range)

    return mymap, my_cmap
    ```
