---
id: fortune-teller
summary: Create a simple micro:bit powered Fortune Teller so that when you shake the micro:bit, it will generate a random answer to a question that you have asked.
categories: micro:bit
tags: projects,easy,micro:bit
difficulty: 3
status: published
published: 2019-04-15
author: Joshua Lowe
feedback_url: https://github.com/AllAboutCode/learn.edublocks.org/issues

---

# Fortune Teller

## Overview

In this tutorial we are going to create a simple micro:bit powered Fortune Teller so that when you
shake the micro:bit, it will generate a random answer to a question that you have asked.

You will need 
- An internet connection
- A BBC micro:bit
- A USB cable

## Get Started
Duration: 1:00

You’ll need to load up EduBlocks. You can do this by opening a web browser of your choice and typing [https://app.edublocks.org](https://app.edublocks.org) into the search box. Once you've loaded up EduBlocks, you'll be presented with the mode selector. 

![screenshot](https://i.ibb.co/tQ0JcTz/Screenshot-2019-04-14-edublocks.png)

Now, we want to select the micro:bit mode. To do this simply click on the blue select button underneath the micro:bit icon. This will load up the micro:bit mode.

Once you've selected the micro:bit mode, you should see it pop up:

![screenshot](https://i.ibb.co/93PHxFY/Screenshot-2019-04-14-edublocks-2.png)

## Import the libraries
Duration: 1:00

Now its time to build our code. We can drag our code blocks from the EduBlocks toolbar which is on the left hand side of the screen. The pink blocks can be found in the basic menu. This will form the start of the code. Both blocks are necessary for this program to work.

![screenshot](https://i.ibb.co/0ZdJykH/Screenshot-from-2019-03-15-22-14-39.png)

## Create a list and loop
Duration 1:00
Next, we need to create a list. We can do this by going into the blue Variables category and then by clicking the "Create Variable" button. Name the variable answers when prompted.

![screenshot](https://i.ibb.co/7Ys42gV/Screenshot-2019-04-14-edublocks-3.png)

Now we need to drag the block that says `answers = 0` into the workspace. Change the 0 to `"Yes", "No", "Maybe"`. Then drag in a `while True:` loop from Basic. From here on, the rest of the code will put inside of this loop.

![screenshot](https://i.ibb.co/N9kNDt9/Screenshot-2019-04-14-edublocks-4.png)

