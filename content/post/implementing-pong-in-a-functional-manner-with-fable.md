---
title: "Implementing Pong in a functional manner with Fable"
date: 2016-12-26T00:00:00+02:00
tags: [ "f-sharp", "fable", "game", "pong" ]
---
> This post is part of the [F# Advent Calendar in English 2016](https://sergeytihon.wordpress.com/2016/10/23/f-advent-calendar-in-english-2016/). Please check out the other posts as well.

For the last few weeks I’ve been playing around with [Fable](http://fable.io/).
As a F# enthusiast who had to deal with a lot of JavaScript code during his studies, I was quite curious what Fable was all about.
For those of you who don’t know what Fable is, here is a quote from their website:
> Fable brings together the power of the F# compiler and Babel to make JavaScript a true backend for F#. 
It works directly on F# source code, no compilation needed. 
Fable optimizes F# code to generate as clean JavaScript as possible. – ([fable.io](http://fable.io/))

Beeing motivated by [Super Fable Mario](http://fable.io/samples/mario/index.html), I thought creating a simple game myself might be a good way to start with Fable. 
As you can read by the title, I chose [Pong](https://en.wikipedia.org/wiki/Pong) as my starting project. So without further ado, let’s start with the actual game.

## Defining the model
When thinking about the Pong game, there are basically three types of models:
* **Paddles**: The paddles have got a position and a specific size.
* **Ball**: The ball has got a position and a size as well. But it has also got speed and an angle.
* **Game status**: Some kind of storage containing information about the current score.

To implement those models, selecting record types seemed appropriate to me.
<script src="https://gist.github.com/oopbase/be25233fbba4f9529f20f657ff9bbc88.js"></script>
As you can see in line 9, the ball stores a *PongElement* itself, since the ball has almost got the same properties as the paddles.
The *GameStatus* does also contain the boolean flag *active* to indicate whether the game is currently running. 
The rest of the model is quite straightforward I guess. Obviously it is straightforward. It is F# code, isn’t it? 😉

## Controlling the paddles
To control a paddle there are only two functions necessary.
A *canMove*-function, to indicate whether a paddle can move in a certain direction, and an actual *move*-function to move the paddle.
<script src="https://gist.github.com/oopbase/915b6d2a2e04d293e9e48eb1a9420ab1.js"></script>
The parameter direction is a tuple *(int * int)*. When the first parameter of the tuple is set to 1, we want the paddle to move up.
When the second parameter of the tuple is set to 1, we want the paddle to move down.
By using pattern matching we can check which value of the tuple is set to 1.
Since every object is immutable, we either return a copy of the current paddle with its new Y-position (line 10 and 11) or simply return the input-paddle if no movement is allowed.

## Ball movement
As described in the model, the ball has got some speed value and an angle it is flying with. Using simple trigonometry, a function to move the ball may look as follows.
<script src="https://gist.github.com/oopbase/1c36805fa455a95a206574b08749b71c.js"></script>
So everytime the *moveBall* function is called, a new instance of the ball record is being returned with a new position and an adjusted speed value.

## Collision detection
Before implementing the *checkCollision*-function, let’s start visualizing the collision by using a discriminated union.
<script src="https://gist.github.com/oopbase/10143a718eceff10802913a5411980d4.js"></script>
This discriminated union describes the whole collision system of the game: 
There can be no collision (**None**), the **Top** or **Bottom** of the canvas may be hit, the **Left** or **Right** part of the canvas may be hit (so a player scored) or 
finally a paddle was hit (**LeftPaddle** & **RightPaddle**). The following function takes the paddles and the ball as input parameters and returns the found type of collision.
<script src="https://gist.github.com/oopbase/487bce70485c386edac58a25fb34ced8.js"></script>
With this function, implementing a final collision-function to determine the new angle of the ball is straight forward again. (Thanks to pattern matching)
<script src="https://gist.github.com/oopbase/6e6e158c0b63a90a2567f33b7b23a407.js"></script>
When hitting either the top or the bottom of the canvas, we negate the value of the angle (angle of incidence is equal to the angle of reflection). 
When hitting the left or right part of the canvas, we simply keep the input angle, since evaluating the score isn’t done here. 
To actually calculate the angle when a paddle is hit, we use yet another function.
<script src="https://gist.github.com/oopbase/68807b5d8a187dd2b4b27c1fe8299535.js"></script>
For the calculation, we determine the relative intersection where the ball hit the paddle. Afterwards we normalize that value. 
So for example, if the paddle is 20 pixels high, that value will be between -10 and 10. 
Therefore we can dynamically calculate the angle depending on the impact. As seen in the collision function, the determineAngle 
parameter of this calculation-function is a function itself. Depending on which paddle is hit, we have to use a slightly modified 
calculation of the final angle. As you can see in line 5, we’ve also got a special case we have to deal with. If the right paddle got 
hit in the exact center, so normalizedRelativeIntersectionY = 0. && hitRightPaddle, we will have to return Pi as the new angle, since 
the radiant value of Pi is equal to 180°.

## Keyboard interaction
The whole code you’ve seen until now was plain F# code. To actually interact with the game, we finally need the Fable libraries. Let’s start with the Keyboard.
<script src="https://gist.github.com/oopbase/db6a398a3513e71a107c783d1a5608d6.js"></script>
The basic idea of the keyboard module is to store the pressed keys in a mutable set. 
By adding an *EventListener* for the *KeyDown-* and *KeyUp-Event* of the HTML document, we can identify 
which key is being pressed. So if the W-Key is pressed, the decimal value 87 (ASCII value for W) is stored into the set. Therefore the 
leftControlsPressed-function will return a tuple containing the values (1, 0).

## Window
The final part of the game is drawing the elements. We use a HTML canvas element and some encapsulated functions to draw the paddles, the scores and the ball.
<script src="https://gist.github.com/oopbase/33b7d1ab022d12b92c229ec7adf42bd0.js"></script>

## Game loop
Now we have defined everything which is necessary to play the game. We can now initialize our paddles and the ball. 
We also use a recursive *update*-function to render the new positions of the elements.
<script src="https://gist.github.com/oopbase/febfefade6a11a4a7f2fcf3f7c4bb51d.js"></script>
And that’s it. We can now use Fable to transpile our F# files. We then simply need an HTML-file which contains a canvas and includes the transpiled script. 
Since I forked the Fable repository, the whole code of the game can be found [right here](https://github.com/oopbase/Fable/tree/pong-sample/samples/browser/pong).