---
layout: post
title: Complex Python lambdas
---

As part of the 2nd programming assignment of CSCI 561 AI class, we had to program AI agents that implement either the [Mimimax](https://en.wikipedia.org/wiki/Minimax) or [Alpha-Beta Pruning](https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning) algorithms to play a fictional board game. Using these algorithms, the agents would essentially generate a series of possible actions, calculate the utility of the board when each action is applied, and somehow choose the best action.  <!-- more -->

Simplifying things, an action is a function that manipulates the board so that it places the player’s piece on an empty square. In other words, a function that takes a board configuration as input and outputs a board with the player’s move applied. The agent needs to generate all the possible actions in any given moment and choose the one that’s best for him. From an abstraction standpoint, it would make sense to use lambda functions to implement the actions.

Lambda functions are what we call anonymous functions, i.e., functions that are not bound to a name. Python has a very powerful implementation of lambdas and you’d normally see it used in conjunction with typical functional concepts like `filter()`, `map()`, and `reduce()`. For example:

	>>> a = [1, 2, 3, 4]
	>>> b = map(lambda x: x**2, a)
	>>> b
	[1, 4, 9, 16]
	>>> k = 5
	>>> c = map(lambda x, offset=k: x + offset, a)
	>>> c
	[6, 7, 8, 9]

As you can see when we define a lambda we specify the arguments—be that inputs or captured variables from the current scope—and a return function. The function part is usually a one-line data manipulation that is immediately returned.  

In our case we want to do more complicated things. First, our function has to do an assignment; we need to assign the player’s piece, say ‘X’, to a square in the board. Secondly, we have to create such lambda for every possible action. The latter is easier. Given that we know all the valid positions  we can place our piece on, we can loop through them in a list comprehension and produce a list of lambdas:

	def actions(board):
		return [lambda b, x=x, y=y, player=board.turn: **magic**
			for (x, y) in possible_moves(board)]

For every coordinate `x, y` that is square of a possible move, we are generating a lambda that takes a board `b`, somehow assigns the player’s piece to that square, and returns the modified board. But how are we going to implement this assignment?  

In Python we can’t immediately do assignments inside a lambda—but there’s a trick. We’ve already used a list comprehension; with it one can populate a list using another one. For example:

	>>> a = [x**2 for x in [1, 2, 3, 4]]
	[1, 4, 9, 16]

What is not evident at first, is that while generating the list `a`, `x` is being assigned to each value of the second list—we can use a list comprehension do assignments inside lambdas! So we can do this:

	>>> [None for b[x, y].status in [player]] 
	[None]

The above will generate a list with just one `None` element, but in doing so it will assign the variable `player` to `b[x, y].status`. Finally, we said we need to return the modified board. Simple, just put the whole board as the last element in our list and return it!

	>>> [[None for b[x, y].status in [player]], b]
	[[None], <__main__.Board object at 0x10801fc50>]
	>>> [[None for b[x, y].status in [player]], b][-1]
	<__main__.Board object at 0x10801fc50>

With that our `actions` function becomes:

	def actions(board):
		return [lambda b, x=x, y=y, player=board.turn: 
				[[None for b[x, y].status in [player]], b][-1]
			for (x, y) in possible_moves(board)]

This function creates a list of lambda functions, each of which gets as input a board `b`, assigns the player’s piece to the specified `x, y` square using the list comprehension, and returns the modified board by putting it as an element in the list and accessing it through the subscript—exactly what we wanted.

This is a very powerful way to write lambdas. Using this method you can implement anything you want. To take it to an extreme, you could even have your whole program inside a lambda!

	(lambda: [
	    ['def'
	        for sys in [__import__('sys')]
	        for math in [__import__('math')]
	
	        for sub in [lambda *vals: None]
	        for fun in [lambda *vals: vals[-1]]
	
	        for echo in [lambda *vals: sub(
	            sys.stdout.write(u" ".join(map(unicode, vals)) + u"\n"))]
	
	        for Cylinder in [type('Cylinder', (object,), dict(
	            __init__ = lambda self, radius, height: sub(
	                setattr(self, 'radius', radius),
	                setattr(self, 'height', height)),
	
	            volume = property(lambda self: fun(
	                ['def' for top_area in [math.pi * self.radius ** 2]],
	
	                self.height * top_area))))]
	
	        for main in [lambda: sub(
	            ['loop' for factor in [1, 2, 3] if sub(
	                ['def'
	                    for my_radius, my_height in [[10 * factor, 20 * factor]]
	                    for my_cylinder in [Cylinder(my_radius, my_height)]],
	
	                echo(u"A cylinder with a radius of %.1fcm and a height "
	                     u"of %.1fcm has a volume of %.1fcm³."
	                     % (my_radius, my_height, my_cylinder.volume)))])]],
	
	    main()])()

But please, don’t do that.