---
title: Fight Chain
description: Using a Breadth First Search algorithm to map out UFC fights
date: 2024-09-27T00:40:04+10:00
draft: false
tags: [UFC, MMA, fight analysis, breadth-first search, algorithm, data visualization, web application]
---
# The Idea
It's been ages since I've posted something. One of my hobbies is to watch Mixed Martial Arts, and the question often comes up that if fighter X beats fighter Y, and fighter Y beats fighter Z, could fighter X beat fighter Y?

An example of this is an upcoming fight in October, Khamzat Chimaev v Robert Whittaker. Chimaev has beaten Gilbert Burns, who has beaten Stephen Thompson, who has beaten Robert Whittaker.

Obviously that doesn't really make sense to use this to predict fight outcomes due to the different styles in mixed martial arts, but I did always have the idea of making a webapp that finds these chains.

# The Data

I found a csv version of the data at [this github repository](https://github.com/Greco1899/scrape_ufc_stats). I will be updating the spreadsheet i found, but currently it only goes up to UFC 293. 

It's a CSV file, so pulling the data from it is quite simple.

# The code
The algorithm used is a Breadth First Search algorithm. It does this by first initializing the fighters that Fighter A has beaten in the queue, and each path in the queue represents a sequence of successful fights. 

The algorithm then undergoes 'exploration', in which it checks if the next fighter is fighter B, and in that instance it can stop there as it already knows there is a direct chain. However, if there is no direct chain, it will then retreive all the fighters that this fighter has beaten (their neighbours in the fight graph) and appends each to the current path, which it will then explore next.

It will keep continuing until it finds a fighter that beat fighter B, or until there are no more chains to explore.

![Breadth First Search algo](https://upload.wikimedia.org/wikipedia/commons/4/46/Animated_BFS.gif)

Using an algorithm like this, it is important to ensure that you don't follow loops (ie if fighter G has beaten fighter H, and fighter H has beaten fighter G, the algorithm would get stuck in a loop), so the easiest way to resolve this is to check if the fighter has already been visited (since their chain has already been explored).


# The website
Check it out, but I didn't put much effort into making it look pretty. I want to someday develop this a bit further and possibly use it to find patterns in future fights / sport outcomes:

https://rat.blue/fight_chain.php

Find the source code [here](https://github.com/rat-blue/fight-chain).
