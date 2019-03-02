# RSS Team 13

[cute lil team intro]

## Meet The Team!

### Eric Boehlke

### Yuna Gan

### Nada Hussein

### Mia LaRocca

### Andrew Reilley


## Using the template

We've made this template as straightforward as we could to use.
You can edit this using markdown which is the same language the lab reports are written in.
It is very simple and there are plenty of guides online about how to use it.


We've written a small python script that takes your markdown files and turns the into HTML files for the web.
Each markdown file is contained in its own directory and after it is turned into HTML you will be able to view that page at:


    https://github.mit.edu/pages/rss2019-[TEAM_NUMBER]/website/my/directory


Running the python script requires a small library for converting markdown, so first run:


    sudo apt-get install python3-pip
    sudo pip3 install mistune


Then, if for example you wanted to edit your lab 3 report you would do the following:


    # Edit lab 3
    vim website/labs/3/README.md


    # Run the python script to convert to HTML
    ./make.py


    # Stage, commit and push your changes
    git add website/labs/3/README.md website/labs/3/index.html
    git commit -m "Edited lab 3"
    git push origin master


The template also supports $\LaTeX$ via mathjax! For inline math use single dollar signs  and for multiline math use use double dollar signs like this:

$$
\begin{align}
  e^{i\pi} + 1 = 0
\end{align}
$$
