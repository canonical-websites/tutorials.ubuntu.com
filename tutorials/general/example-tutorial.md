---
id: example-tutorial
summary: Learn how to create, write and publish tutorials on learn.edublocks.org, reaching a wide audience of both beginner and advanced Linux users.
categories: general
tags: tutorial, guidelines, guide, write, contribute
difficulty: 2
status: published
feedback_url: https://github.com/AllAboutCode/learn.edublocks.org/issues
published: 2019-04-19
author: Joshua Lowe

---

# How to write a tutorial

## Overview
Duration: 1:00

In this tutorial, you will learn how to write content for learn.edublocks.org.

We will start by looking at general guidelines, the structure of a tutorial and go through the publication and review process.

### What you'll learn

* How to create and structure a tutorial from a single Markdown file
* How to use the additional Markdown features specific to the engine
* How to render it locally to see what your readers will see

### What you'll need

* A computer running Linux
* The `git` command-line client (that you can install by running `sudo apt install git`)
* [Docker](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) to run the website locally
* A [GitHub](https://github.com) account, for the publication and review process

Depending on the topic and your level of experience, writing a tutorial can be a very easy task, but following these guidelines is important to keep the whole set of published tutorials consistent. Let's get started!

## General guidelines
Duration: 1:00

### Mission of tutorials

Tutorials are step by step guides aimed at a very diverse audience. To provide a good learning experience, a consistent and didactic approach is key.

A good tutorial should:

* be focused on one topic or a very small group of related topics. Keep it simple and on point as people who want to learn multiple subjects will take multiple tutorials.
* produce a tangible result. The topic is demonstrated with a *small practical project* and not only a theoretical or "hello world" example. The tutorial reader will come out of it with a working example.
* be short. An estimated 60 minutes for a tutorial is an absolute maximum. Most tutorials should be in the range of 15 - 30 minutes.
* be divided in short steps. Each step is practical and results in user-visible progress.
* be entertaining! Try to have a fun project to work on, even if it's something impractical!


## Metadata and structure
Duration: 5:00

Each tutorial is built using a single Markdown file.

This file starts with a set of metadata in *Front matter* format to be consumed by the tutorials engine. For example, at the top of the source file of this guidelines tutorial, you will find:

```
---
id: happy-birthday
summary: Use basic python skills to create a happy birthday song in python
categories: python3
tags: projects, python, easy, fun
difficulty: 2
status: published
published: 2019-04-19
author: Joshua Lowe
feedback_url: https://github.com/AllAboutCode/learn.edublocks.org/issues

---
```

Let's have a look at each field:

 * **id**: the identifier of the tutorial to be used in the URL. Keep it short but clear.
 * **summary**: a description of the tutorial (10 to 30 words) to be displayed on the frontpage of the site
 * **categories**: any of `micro:bit`, `raspberrypi`, `python`, `circuitpython`
 * **tags**: a comma-separated list of tags used by the site search
 * **difficulty**: the scale spans from 1 to 5. Beginners without previous knowledge of the given topic should be able to follow and understand tutorials from level 1 without any other prerequisite knowledge. As a guide:

     * **1**: Complete beginner
     * **2**: Novice
     * **3**: Experienced user
     * **4**: Advanced
     * **5**: Developer

 * **status**: `draft` or `published`
 * **feedback_url**: where to report bugs and give feedback about this tutorial. Should always be the issues page.
 * **author**: the name and email address between brackets of the author of the tutorial.
 * **published**: a date in YYYY-MM-DD format, to be updated after any major change to the tutorial

After this metadata section, the tutorial starts.

### Title

The title should be set as the first heading of your file.

```markdown
# Tutorial title
```

The tutorial title should be kept short (3 to 8 words as a guide) to not break the design. Try to make concise titles but also specific when possible.

### Steps

Each step is delimited by a second level title, for example:

```markdown
## Step title
```

A step **Duration** in the `MM:SS` format should immediately follow the step title. The total tutorial time will then be computed automatically. A third level heading or empty line will break into the step content.

```markdown
## Step title
Duration: 2:00

Step content starts here.
```

## Dos and Don'ts
Duration: 5:00

In addition to the previous advice on what a tutorial should be and what is mandatory, you should pay special attention to the following points:

### Each step should be concise, but not too short

Be wary of a step's length. In average, 5 to 10 minutes is more than enough for a single step to complete. Don’t make them too short either. Naturally, some steps will be shorter than others (such as the first and last steps).

### If too long, prefer dividing the tutorial

Tutorials are self-sufficient, but they can nonetheless build upon each other (you can link from the requirements section of the first step, for example). One tutorial could require another tutorial to be completed first. And if you are reusing the same code, ensure you provide a repository as a starting point.

If a tutorial is too long, consider breaking it up into several pieces. However, ensure all tutorials present a distinct objective.

### Don’t have too many steps

Steps should be concise and tutorials should be rather short. Consequently, you shouldn’t have too many steps in your tutorial. We don't want to make the reader desperate by glancing at the number of remaining steps before tutorial completion.

### Each step should be rewarding

As a writer, you should try to keep the reader entertained at each step and this is achieved by careful story building. Each step should end on concrete progress towards the end goal. It should be, if possible, tangible and interactive, so that the reader can be familiarised with notions introduced by the step.

### Do not repeat the setup/install phase for each tutorial

Avoid repetitive setups or installation phases, particularly if the tutorial isn’t a beginner one. Beginner tutorials should contain a setup phase while more advanced tutorials should reference other beginner tutorials as prerequisites.

### Code snippets

Inline code blocks are styled with single backticks :

For example:

```markdown
`from micro:bit import *`
```

Which renders as `from micro:bit import *`.


### "Next steps"

With a list of bullet points, offer some guidance on the next steps a reader may want to take. This could be other tutorials being the “next logical ones”, communication channels and places where to get support from.

## Syntax tips
Duration: 05:00

The syntax used is by and large regular [Markdown syntax](https://guides.github.com/features/mastering-markdown/), but there are some specificities:

### Line breaks and empty lines

* Paragraphs are delimited by empty lines
* Line breaks will create a new line

In the context of an admonition or a survey widget, using an empty line will close it and go back to text.

### Images

Images can be hosted locally (relatively linked to the markdown source) or remotely. The tutorial engine will fetch remote images and cache them locally.

In Markdown the syntax for an image is the following:

```markdown
![image title](image-path-or-link)
```

### Admonitions

Admonitions are colored blocks that enclose special information, they can be a positive tip or a negative warning. To create an admonition, write its type ("positive" or "negative") on a line by itself, then begin the next line with a colon and a space.

A **positive** admonition should contain positive information like best practices and time-saving tips.

```markdown
positive
: **Eat your vegetables!**
This is a positive message.
```

Which renders as:

positive
: **Eat your vegetables!**
This is a positive message.

A **negative** admonition should contain negative information such as warnings .

```markdown
negative
: **Eat your vegetables!**
This is a warning.
It can be multi-lines like this.
```

Which renders as:

negative
: **Eat your vegetables!**
This is a warning.
It can be multi-lines like this.


## Local rendering
Duration: 1:00

When writing a tutorial, it's extremely useful to see how it will render on the website.

We are going to need a local instance of learn.edublocks.org, which is something we can achieve without any advanced technical knowledge: it takes 5 minutes and a single command handles all the process.

To do so, start by cloning the tutorials git repository:

```bash
git clone https://github.com/AllAboutCode/learn.edublocks.org
```

Then enter the `learn.edublocks.org` directory and launch the `run` script.

```bash
cd learn.edublocks.org
./run
```

If you have Docker correctly installed, it will build a local learn.edublocks.org instance. The first time (and only the first time) you run this command, it will download all the required dependencies. When the site is ready, a local server starts serving it at `http://localhost:8016`.

By default, the site only displays this how-to tutorial.

The `run serve <path to md file or directory>` command let you choose what to display. For example, to display:

* All available tutorials: `./run serve tutorials`
* A single tutorial: `./run serve my-tutorial.md`

Other options are available by running `run --help`.

Each time you save a file you are working on, it triggers a page refresh on this local server, allowing for rapid iterations on the content.

Once you can have seen your tutorial in its final form, it's time to share it with the world.

## Review process
Duration: 01:00

The tutorials repository is managed by the EduBlocks team. A review process is in place to ensure new tutorials are being looked at.

### Source code structure

The learn.edublocks.org source code contains a `tutorials` directory. Each of its subdirectories matches a tutorial category Your tutorial should be included into one of these.

For example:

```bash
-tutorials/
└── microbit/
    └── awesome-project.md
```

If your tutorial contains locally hosted images, you should create a subdirectory matching your tutorial id and include an `images` directory:

```bash
-tutorials/
└── microbit/
    └── awesome-project
        ├── images/
        └── awesome-project.md
```

Once your tutorial is in the right directory, you can then propose a **pull request** of your changes.

### Making a pull request

If you are not familiar with the `git` command-line, here is how to propose a pull request on the project.

Open a terminal and from the project directory, run the following commands:

Create a new code source branch, parallel to the origin source code:

```bash
git checkout -b <tutorial-id>
```

Add and commit your changes to the branch, with a commit message explaining what you are adding:

```bash
git commit -am "Adding a new tutorial: <title of the tutorial>"
```

Push it to GitHub, as a branch of the project:

```bash
git push -u origin <tutorial-id>
```

You will then be asked to enter your GitHub credentials.

Once your `push` has been processed, open the [GitHub project page](https://github.com/AllAboutCode/learn.edublocks.org). You should see a highlighted box near the top of the page with your newly pushed git branch name (the tutorial id) on it. Click on the "Pull Request" button next to it.

Fill up the form with the required information, such as describing your new tutorial and why it would be useful to readers.

When your pull request is submitted, the team of reviewers will receive an email and add it to the review queue. Once they get to it, they will review the content and open a discussion with you on the Pull Request page. At the end of this process, they will either merge your changes into the project or request additional changes.

## That's all folks!
Duration: 01:00

Congrats, you made it! If you have been watching closely, you are now fully equipped to write a tutorial!

Now you know how to:

* Create a welcoming and informative intro to your content
* Provide an interesting and easy-to-follow learning experience
* Structure your markdown source file
* Render tutorials locally and submit them for publication

