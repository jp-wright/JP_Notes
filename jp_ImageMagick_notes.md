# ImageMagick notes:


### To convert a sequence of images to a .gif
There are multiple ways to do this, including smooth ways of doing it in a script or from the command line itself.  

### Following is a detailed example using MatPlotLib's `animation` API:
1. Import the `animation` API:
```python
from matplotlib.animation import FuncAnimation
```
2. Define an `update` function to iterate through the sequence of images for the gif (this means creating each image in the sequence iteratively in your script -- dynamic, powerful).  Here's an example:
```python
def update(i):
    label = 'timestep {0}'.format(i)
    print label
    # Update the line and the axes (with a new xlabel). Return a tuple of
    # "artists" that have to be redrawn for this frame.
    line.set_ydata(x - 5 + i)
    ax.set_xlabel(label)
    return line, ax
```
3. Create an animation object, passing in your update:
```python
anim = FuncAnimation(fig, update, frames=np.arange(0, 10), interval=200)
```
4. Use Matplotlib's `FuncAnimation.save` command to save an animated figure with your attributes of choice, using `imagemagick` as the `writer`.  In this example, we also pass in the test for using `save` as an optional argument on the command line after the script name when the script itself is run.  So, if we wanted to run our script, create a gif, and save it instead of display it, we would enter the command line argument as `python my_script_name.py save`.  (JP Note: I have had difficulty getting this to work, but this is the right code.  Need to investigate it).
```python
if len(sys.argv) > 1 and sys.argv[1] == 'save':
    # print "save me"
    anim.save('/Users/jpw/Desktop/line.gif', dpi=80, fps=30, writer='imagemagick')
else:
    plt.show()  
```  


### Here is how to create a gif using the command line alone, manually:
1. In terminal, navigate to the folder where the sequence of images are.
2. Enter `convert img1.png img2.png img3.png [... imgN.png] gif_name.gif`

    + This takes the images listed, all the way up to image_N, in order and makes an animated gif from them.  Changing the order of the listed images changes the order they appear in the gif.  Will work for most major image filetypes, not simply .png.  
    + If the only files of the image filetype, such as .png, that are in the folder are the images you want to use, you can use the `*` (Unix) command to simply convert all of them (ordered by file name, I believe), to a gif in one command: `convert *.png gif_name.gif`

3. The default gif conversion command above produces a pretty fast-changing gif, so we might want to include some of the following command line arguments to modify our gif to our liking.  Put all optional arguments before the filenames, i.e. immediately after `convert`. For a full list of arguments, not limited to gifs, see [ImageMagick Convert Commands](https://www.imagemagick.org/script/convert.php). __NOTE:__ file types are case sensitive.  'JPG' is different than 'jpg'.

    + `-delay '<value>'` -- insert a pause, in hundredths of a second, between displaying the next image in the gif series.  Enclose value in single quotes.  Can add optional `>` or `<` after `value` to only enforce delay if current delay is greater than or less than the new one specified in this argument.  So, you would use `convert -delay '15' *.png img.gif` to convert all .png files in the folder into a gif called "img.gif" with a 0.2 second delay between each frame.  Here is a short article about how to use this command: [Gif delay](http://blog.floriancargoet.com/slow-down-or-speed-up-a-gif-with-imagemagick/)
    + `-extent '<size>'` -- crop image to set area in size (I think)
    + `-resize '<size>'` -- resize image (e.g. '1224x1632')
    + `-rotate '<degrees>'`
    + `-size '<size>'` -- set image size
    + `-scale '<size>'` -- scale image (e.g. '1224x1632')
    + `-loop '<iterations>'` -- how many times the gif will repeat; default is infinite.
    + `-thumbnail '<area>'` -- area of image to serve as a thumbnail
    + `-transpose` -- rotate 90 degrees
    + `-transverse` -- flip image
    + `-mattecolor '<color>'` -- frame color
    + `-negate` -- replace each pixel with its complementary color.
    + `-quality '<value>'` -- set compression level for JPG/PNG/MIFF. 1 is worst, 100 is best.  Different levels for PNG (think it is 0-7 ?)

4. The largest impact on GIF file size is the size of the original photos.  Reducing their dimensions by scaling them will lower final file size. `convert -delay '12' -resize '378x504' *.JPG img_small.gif`  The best scaling is even ratios of `/2` or `/4`, etc.

    For iPhone portrait image size:
    1. (3024 × 4032) / 2 = 1512x2016 -- x-large
    2. (3024 × 4032) / 4 = 756x1008  -- large
    3. (3024 × 4032) / 6 = 504x672  -- med
    1. (3024 × 4032) / 8 = 378x504  -- small

    Flip these results for landscape photos...
