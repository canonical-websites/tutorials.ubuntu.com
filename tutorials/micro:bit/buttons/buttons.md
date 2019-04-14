---
id: buttons
summary:  Create a simple program that uses the 2 buttons on the micro:bit to scroll text when a certain button is pressed. 
categories: micro:bit
tags: projects,easy, micro:bit, buttons
difficulty: 2
status: published
published: 2019-05-14
author: Joshua Lowe
feedback_url: https://github.com/AllAboutCode/learn.edublocks.org/issues

---

# Using the buttons

## Overview

In this tutorial we are going to create a simple program that uses the 2 buttons on the micro:bit to scroll text when a certain button is pressed. 

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

## Imports and Loops
Duration: 1:00

Now its time to build our code. We can drag our code blocks from the EduBlocks toolbar which is on the left hand side of the screen. The pink blocks can be found in the basic menu. This will form the start of the code. The first block will import the code we need to control stuff on our micro:bit whilst the second block will create a forever loop

![screenshot](Screenshot_2019-04-14 edublocks.png)

## Create a list and loop #5E12F8
Duration: 2:00

Next, we need to create a list. We can do this by going into the blue Variables category and then by clicking the "Create Variable" button. Name the variable answers when prompted.

![screenshot](https://i.ibb.co/7Ys42gV/Screenshot-2019-04-14-edublocks-3.png)

Now we need to drag the block that says `answers = 0` into the workspace. Change the 0 to `"Yes", "No", "Maybe"`. Then drag in a `while True:` loop from Basic. 

![screenshot](https://i.ibb.co/N9kNDt9/Screenshot-2019-04-14-edublocks-4.png)

positive
: **NOTE:**
From here on, the rest of the code will put inside of this loop.

## Check for a shake
Duration: 2:00

Now we need to check if the micro:bit has been shaken. Grab an `if` block from basic and then from Accelerometer, get a block which states `accelerometer.was_gesture('shake')` and put it inside of the if block where it says `True`. 

![screenshot](https://i.ibb.co/T1ffTJW/Screenshot-2019-04-14-edublocks-5.png)

Whatever we put inside this if statement will run when the micro:bit is shaken. We want to scroll a random choice from our answer list. To do this, get a block from "Display" which states `display.scroll(0)`, drag this inside of the if statement. Now we want to replace the `0` with some code that will pick a random choice from our answer list. Replace `0` with `random.choice(answers)`. This section should now look like this:

![screenshot](https://i.ibb.co/gZ6W94Z/Screenshot-2019-04-14-edublocks-6.png)

## Final Code
Duration: 1:00

You've now finished all of the code! It's time to check to see if we haven't missed a step or made a mistake. Now is your chance to check your code compared to the image below to check if it's all right.

![screenshot](https://i.ibb.co/XLzbq9t/Screenshot-2019-04-14-edublocks-7.png)

## Download your code
Duration: 3:00

It's now time to download your code!

Connect the micro:bit to your computer using a micro USB cable. Your micro:bit will show up on your computer as a drive called 'MICROBIT'. 

![screenshot](https://i.ibb.co/QvWrrNh/ezgif-com-video-to-gif.gif)

To download our code onto the microbit. Click the DOWNLOAD HEX button in the navigation bar at the top of EduBlocks. This will download a 'hex' file, which is a compact format of your program that your micro:bit can read. 

![screenshot](https://i.ibb.co/d2zrVgQ/Screenshot-2019-04-14-edublocks-8.png)

Once your code has downloaded, head over to your downloads folder where you'll see a file named `microbit-edublocks.hex`. Drag this onto the `MICROBIT` drive and you'll see a yellow flashing light on the back of your micro:bit. Once it's finished flashing, you're code will now run!

![screenshot](https://i.ibb.co/j3H14WJ/ezgif-com-video-to-gif-1.gif)

## Test your code
Duration: 1:00

You should now be able to test your code.
Shake the micro:bit and it'll scroll a random choice from our answers list.
Congratulations, you've made a working Fortune Teller!

![screenshot](https://pbs.twimg.com/media/DI9ZGudXcAEVQEF.png)

### What you've learnt

  - Learnt how to create a list
  - Learnt how to use logical statements
  - Learnt how to use a loop
  - Learnt how to choose a random option from a list