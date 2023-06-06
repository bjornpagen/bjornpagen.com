This is the first post in a series of many.

In a long journey to create the best blog the world has ever seen,
I have attempted to use a lot of different software. Astro, Hugo,
raw static sites... However, I've come to the conclusion that it's
best to use a static site generator, but not one that's already done
(ie: Hugo). I needed to write my own.

## Why Start from Scratch?

Many years ago, I made a blog using Hugo. I remember it being
extremely difficult to deal with the configuration and to
modify the output to my liking. Also, if I wanted to do cool
graph stuff with my blog or notebook (like a knowledge tree),
there is no trivial way to get this working with something
that I haven't written myself.

## Workflow

When I login to my computer, I see a terminal. I like to edit
files directly from CLI: I've noticed it's the only way to
decrease the resistance enough for me to actually write anything.

To make it as easy as possible to write, I use helix editor in
my terminal emulator, with all of my notes as individual markdown
files inside a git repo. There was no way I was going to maintain
a blog that I have to write in HTML, and no way that I was going
to be satisfied with the CSS output of blog generators like Hugo.

I didn't want to do *any* extra configuration with my raw `.md`
files to make them special. So, I needed to write my own markdown
generator that injected the special details that it needs.