# JP Atom Notes
Atom is highly customizable using the various "packages" offered both by Atom's developers themselves and by the open-source developer community.  To access the plugins currently installed and to download more, go to Atom --> Preferences --> Packages. Atom comes with "Core Packages" installed by default which include many of the "duh" options we will want.  Here are some basic tips for new users to get the most from Atom.  When given the choice, usually safest to choose a package from Atom developers as opposed to third-party.  

Atom was also developed by some of the people who helped develop Github, so it is made to work with Git and Github directly.  Depending on your project Atom will scan your directory (more about this in class, can cause memory issues) to monitor it for changes to your project / repo.  This means it also integrates easily with Github if we choose to utilize it.  

<BR>

### Syntax Highlighting
Atom allows highlighting for just about any major programming language you can name.  For this class specifically you will want to make sure `language-python` is enabled.  Search for "Python" in the search bar in the Packages tab in settings.  Other common ones you'll likely run across in some daily usage are:

+ `language-json`
+ `language-git`
+ `language-html`
+ `language-sql`
+ `language-yaml`
+ `language-xml`
+ `language-css`
+ `language-hyperlink`
+ `language-gfm`

Other major languages such as Ruby, C, and Javascript are all supported as well.  We won't use them in this class, but if you start to in other parts of your data science career, please search for them and enable them.

<BR>

### Keyboard Shortcuts
Atom has some nifty shortcuts which save a lot of time.

##### Multiple Cursors
Foremost is the ability to use _multiple cursors_ at a time!  To do this, simply hold down Command (`Cmd`)  and click where you want additional cursors to be.  This allows multi-line, multi-point editing.  Depending on the file, this can be an unrivaled convenience.

Similarly if you double click a word and have the `highlight selected` package installed, you can hold down `Cmd + d` to cycle to the next appearance of the highlighted word, creating a new cursor at it as well.

##### Jump to Next Occurrence of Word
`Cmd + d`

##### Go to Specific Line
Hit `Ctrl + g` and type the line number you want to jump to.  Makes debugging easier too, as you can see the error message and jump directly to the line in question.

##### Multi-line Shifting
When you highlight a block of text (multiple lines), you can shift it all together.  This is especially common when using `Tab`.  Frequently you'll want to indent or un-indent an entire block of code.  To un-indent, hold `Shift` when you hit `Tab`.

You can also shift any line or group of lines at once in unison.  To do this just use `Ctrl + Cmd` and hit either the `Up Arrow` or `Down Arrow` key.  

##### Instant New Line
There's also a nifty hotkey to create a new line.  Frequently we want have an idea and want to make a new line to start typing, but we are in the middle of a line already, so we would normally go to the end of this line and then hit `Return` to make a new line.  But since we do this so much, Atom has a nice shortcut which is to simply hold down `Cmd` when you hit `Return` and a new line will be made -- with your cursor at the beginning -- regardless of where you are currently.

##### Multi-line Comments
When coding in Python using a hashtag / pound sign (`#`) turns a line of code into a _comment_.  A comment is just what it sounds like, a note or instruction for a human reader and is ignored entirely by the Python script itself.  In Atom, we can comment multiple lines at once by highlighting them and hitting `Cmd + /`.  This will also un-comment any number of lines if they are already commented out.  Very handy!

##### Code Folding
Atom also supports code folding, as do most modern editors.  Most people are familiar with these by using the little triangles on the left border of the editor that collapse, or fold, a block of code (such as a function) into a single line.  The actual code itself is still valid; this is simply to de-clutter your screen in an effort to let you temporarily visually "hide" parts of code you are done with or not currently working on.  As your files grow larger clicking on every function, class, or extended container listing becomes unruly.  

So we can use some nifty keyboard shortcuts to quicken the process and get the full benefits of code folding.  For a single fold on the line where your cursor is, use `Cmd + Option (Alt)` + `[` (left bracket, `[`) to fold the line and the same command with a right bracket (`]`) to unfold the line.  To fold or unfold _all_ collapsable blocks of code, simply add `Shift` to the above command.  Last, if you want to fold/unfold at a specific indentation level, hold `Cmd` and hit `k`, then hit `0-9` (while still holding `Cmd`) to select the indentation level at which you want to collapse or expand.  

##### In-file and In-project Searching
Finally, we should mention that as you work with larger scripts or even libraries and code bases that have many interconnected pieces, you will want to _search_ for a keyword or function, etc.  Atom has a robust search feature which we activate with `Cmd + F`.  We can use it to not only search for words but to also replace them (think if you have 17 occurrences of a variable named "distance_added" but it really is "distance_removed" and you need to rename it; using search we can highlight all the occurrences and _replace all_ with the new name in one go).  We can limit searches to an area we've highlighted (good for changing text in a single function or class), match by case sensitivity, and even use _RegEx_ (a string matching module).  

A lesser-known feature that is very powerful is the ability to search in an entire directory.  To do this, open your Project View Tree (`Cmd + |`, that's a _pipe_ character, on the backslash key above `Return` usually), right-click on the directory you want to search and choose "Search Directory".  A new tab will open that has an indexed result of all the matches of what you searched for throughout that directory / project.  Clicking on any of the links in this results window will take you to the file in question.  Very nice.

##### Tabs
Cycle through open tabs:  
`Cmd + Shift + right/left bracket`  

Move Tabs:  
`Ctrl + Shift + left/right arrow`  
I like to keep my project tabs organized in a certain manner much of the time.  I like to use the keyboard to actually move the tabs.  This is not a particular timesaver, but is rather just a nice touch that I alone will probably enjoy.

##### Highlight All Text Inside Brackets
`Ctrl + Cmd + m`  
This will highlight all content inside a set of brackets, regardless of spacing.

##### Hotkey Menu
To see what all hotkeys are type `Cmd + Shift + P`.


<BR>

### Extra Packages
Here are a few third-party packages (there are thousands out there) that are useful and reliable.

+ `Highlight Selected` - When you double click or highlight a word, this package automatically highlights all other appearances of that word in your file on your screen.  This is really handy when trying to find where the other instances of an argument to a function is modified, for example.

+ `Minimap` - Gives a condensed window overlay view of your entire script.  You can click anywhere on the minimap to jump directly to that spot.  If you have a smaller screen you might not care for the extra screen real estate being used by this.  Up to you.  You can also `minimap highlight selected` to highlight all the occurrences of a word in your minimap.  I don't use the Minimap generally, but it can be nice for very large files, especially if you're plugging your laptop into a larger monitor.

+ `File Icons` - Adds a softly-colored icon to every file based on its file type.  Convenient way to identify what types of files you are working with.  Highly recommended.

+ `Markdown preview plus` - Lets you preview a Markdown file in either Atom Markdown or the ubiquitous Github Flavored Markdown.  Simply open a Markdown file, like your assignments or lectures, and hit __Ctrl__ + __Shift__ + __M__ to open a new tab or pane with the rendered file (i.e. it will look like the nice README files on Github).  I personally dislike the use of split panes on a laptop screen due to limited screen real estate, so I recommend turning off split pane previews in the settings of this package (this means a new full tab will be used). To change these settings go to Atom -> Preferences -> Packages -> <search for Markdown preview plus> -> Settings.  Uncheck "Open Preview in Split Panes" and do check "Use Github Style."

+ `Markdown writer` - Helps with the rote tasks of writing Markdown.  Hitting `Return` when in the middle of a bullet point list will create another bullet point, for example.  Convenient.

+ `Markdown PDF` - make PDF from Markdown on the fly

+ `Autoclose HTML` - For those writing HTML this is a nice way to ensure you don't have dangling tags.  Every time you start a new HTML tag with the opened angle bracket, this package creates the closing angled bracket for that tag.  Convenient.

+ ...various others including Github integration packages.  

<BR>

### Atom Settings
##### Open Atom from the Terminal  
Click 'Atom' in the menubar up top and choose __Install Shell Commands__.
Now you can use commands in the terminal to open files or folders with Atom.

+ `atom .` -- opens current terminal directory in Atom
+ `atom <filename>` -- opens file in Atom (good for hidden files)
+ `atom` -- opens a new Atom window



##### Soft Wrap
Inside Settings there is an Editor Setting called __Soft Wrap__.  This means that the text displayed in your editor will wrap to a new line based on how large the current Atom window is.  If you make it larger, then more text will fit on one line.  A line that is wrapped is _not_ a new line as far as the editor is concerned, it is just a display convenience.  For those coding on laptops, which is most of us, the ability to visually wrap text based on how large I choose the window to be is a screen real estate-maximizing luxury.  It's also excellent for notes.

Some people may want to enforce a Hard Wrap at a set line length, which is fine.  But for working on a laptop, I find the Soft Wrap (_not_ Soft Wrap at Line Length, which I disable) option to be most flexible and effective.  Do note that having long, multi-line statements or comments in code can make it very messy and frustrating to others reading your work on a different monitor setup (e.g. requiring them to scroll comically far to the right).  To address this, I always implement a manual New Line return near the edge of the Atom window.  But having Soft Wrap allows me the chance to have a keyword or statement extend slightly beyond this when it makes sense instead of forcing it onto a new line prematurely.  Ultimately this is a user preference, but one I am advocating for.

##### Tab v. Enter to "Confirm" Auto-completion Suggestions
Depending on personal preference, one may wish to have `TAB` be the "confirm" key instead of `ENTER`.  The default is either `ENTER` or both.  I use `TAB` only because whenever I hit the `ENTER` key, I want it to immediately cause a new line return regardless of if my cursor is currently in the middle of a word which is prompting an auto-complete suggestion.  By using `ENTER` as the confirmation key, the auto-complete suggestion will expand into the area at which your cursor was placed when you hit `ENTER`.  This bugs me greatly, so I use only `TAB`, much like in MS Excel.  The setting to change this option is nestled away in the Atom package __autocomplete-plus__.  
