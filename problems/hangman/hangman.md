# Evil Hangman

## tl;dr

Implement a program that allows someone to play Evil Hangman against the computer.

	$ python hangman.py
	WELCOME TO EVIL HANGMAN >=)
	I have a word in my mind of 8 letters.
	Guess a letter: a
	That's not in the word >=)
	Guess a letter: n
	It's in the word :(
	_____N__
	Guess a letter: ...

## Background

It's hard to write computer programs to play games. When we as humans sit down to play a game, we can draw on past experience, adapt to our opponents' strategies, and learn from our mistakes. Computers, on the other hand, blindly follow a preset algorithm that (hopefully) causes them to act somewhat intelligently. Though computers have bested their human masters in some games, most notably chess and go, the programs that do so often draw on hundreds of years of human game experience and use extraordinarily complex algorithms and optimizations to out-calculate their opponents.

While there are many viable strategies for building competitive computer game players, there is one approach that has been fairly neglected in modern research --- cheating. Why spend all the effort trying to teach a computer the nuances of strategy when you can simply write a program to play dirty and win handily all the time? In this assignment, you will build a mischievous program that bends the rules of Hangman to trounce its human opponent time and time again. In doing so, you'll cement your skills with abstract data types and iterators, and will hone your general programming savvy. Plus, you'll end up with a piece of software which will be highly entertaining. At least, from your perspective.

In case you aren't familiar with the game Hangman, the rules are as follows:

1. One player chooses a secret word, then writes out a number of dashes equal to the word length.
2. The other player begins guessing letters. Whenever she guesses a letter contained in the hidden word, the first player reveals each instance of that letter in the word. Otherwise, the guess is wrong.
3. The game ends either when all the letters in the word have been revealed or when the guesser has run out of guesses.

Fundamental to the game is the fact the first player accurately represents the word she has chosen. That way, when the other players guess letters, she can reveal whether that letter is in the word. But what happens if the player doesn't do this? This gives the player who chooses the hidden word an enormous advantage. For example, suppose that you're the player trying to guess the word, and at some point you end up revealing letters until you arrive at this point with only one guess remaining:

	DO–BLE

There are only two words in the English language that match this pattern: "doable" and "double." If the player who chose the hidden word is playing fairly, then you have a fifty-fifty chance of winning this game if you guess 'A' or 'U' as the missing letter. However, if your opponent is cheating and hasn't actually committed to either word, then there is no possible way you can win this game. No matter what letter you guess, your opponent can claim that she had picked the other word, and you will lose the game. That is, if you guess that the word is "doable," she can pretend that she committed to "double" the whole time, and vice-versa.

Let's illustrate this technique with an example. Suppose that you are playing Hangman and it's your turn to choose a word, which we'll assume is of length four. Rather than committing to a secret word, you instead compile a list of every four-letter word in the English language. For simplicity, let's assume that English only has a few four-letter words, all of which are reprinted here:

	ALLY  BETA  COOL  DEAL  ELSE  FLEW  GOOD  HOPE  IBEX

Now, suppose that your opponent guesses the letter 'E.' You now need to tell your opponent which letters in the word you've "picked" are E's. Of course, you haven't picked a word, and so you have multiple options about where you reveal the E's. If you'll notice, every word in your word list falls into one of five "word families:"

- `----`, which contains the word `ALLY`, `COOL`, and `GOOD`.
- `-E--`, containing `BETA` and `DEAL`.
- `--E-`, containing `FLEW` and `IBEX`.
- `E--E`, containing `ELSE`.
- `---E`, containing `HOPE`.

Since the letters you reveal have to correspond to some word in your word list, you can choose to reveal any one of the above five patterns. There are many ways to pick which pattern to reveal – perhaps you want to steer your opponent toward a smaller family with more obscure words, or toward a larger family in the hopes of keeping your options open. In this assignment, in the interests of simplicity, we'll adopt the latter approach and always choose the largest of the remaining word families. In this case, it means that you should pick the pattern `----`. This reduces your word list down to

	ALLY  COOL  GOOD

and since you didn't reveal any letters, you would tell your opponent that her guess was wrong.

Let's see a few more examples of this strategy. Given this three-word word list, if your opponent guesses the letter `O`, then you would break your word list down into two families:

- `-OO-`, containing COOL and GOOD.
- `----`, containing ALLY.

The first of these families is larger than the second, and so you choose the pattern `-OO-`, revealing two O's in the word and reducing your list down to

	COOL GOOD

But what happens if your opponent guesses a letter that doesn't appear anywhere in your word list? For example, what happens if your opponent now guesses 'T'? This isn't a problem. If you try splitting these words apart into word families, you'll find that there's only one family --- the one with the pattern `----` in which T appears nowhere and which contains both COOL and GOOD. Since there is only one word family here, it's trivially the largest family, and by picking it you'd maintain the word list you already had.

There are two possible outcomes of this game. First, your opponent might be smart enough to pare the word list down to one word and then guess what that word is. In this case, you should congratulate her --- that's an impressive feat considering the scheming you were up to! Second, and by far the most common case, your opponent will be completely stumped and will run out of guesses. When this happens, you can pick any word you'd like from your list and say it's the word that you had chosen all along. The beauty of this setup is that your opponent will have no way of knowing that you were dodging guesses the whole time --- it looks like you simply picked an unusual word and stuck with it the whole way.

## Specification

Your assignment is to write a computer program which plays a game of Hangman using this "Evil Hangman" algorithm. In particular, your program should do the following:

1. Read the file `dictionary.txt`, which contains the full contents of the Official Scrabble Player's Dictionary, Second Edition. This word list has over 120,000 words, which should be more than enough for our purposes.

2. Prompt the user for a word length, reprompting as necessary until she enters a number such that there's at least one word that's exactly that long. That is, if the user wants to play with words of length -42 or 137, since no English words are that long, you should reprompt her.

3. Prompt the user for a number of guesses, which must be an integer greater than zero. Don't worry about unusually large numbers of guesses – after all, having more than 26 guesses is clearly not going to help your opponent!

4. Prompt the user for whether she wants to have a running total of the number of words remaining in the word list. This completely ruins the illusion of a fair game that you'll be cultivating, but it's quite useful for testing (and grading!)

5. Play a game of Hangman using the Evil Hangman algorithm, as described below:

	1. Construct a list of all words in the English language whose length matches the input length.

	2. Print out how many guesses the user has remaining, along with any letters the player has guessed and the current blanked-out version of the word. If the user chose earlier to see the number of words remaining, print that out too.

	3. Prompt the user for a single letter guess, reprompting until the user enters a letter that she hasn't guessed yet. Make sure that the input is exactly one character long and that it's a letter of the alphabet.

	4. Partition the words in the dictionary into groups by word family.

	5. Find the most common "word family" in the remaining words, remove all words from the word list that aren't in that family, and report the position of the letters (if any) to the user. If the word family doesn't contain any copies of the letter, subtract a remaining guess from the user.
	
	6. If the player has run out of guesses, pick a word from the word list and display it as the word that the computer initially "chose."

	7. If the player correctly guesses the word, congratulate her.

	   Ask if the user wants to play again and loop or exit accordingly.

Your program will consist of three major parts.

1. The Lexicon class: Lexicon objects are used to retrieve words for the game from a dictionary.

2. The Hangman class: a *Hangman* object will include all of the logic needed to play the Evil Hangman game. It will keep track of the current status of the game, and it will be able to update the status of the game when a letter is guessed. However, a Hangman object will not directly interact with the user (the person playing the game).

3. The user interface: this is a piece of code that interacts with the user. It displays messages to the user about the game, and prompts her for new guesses. This piece of code will use the Hangman class to keep track of the game itself.

For the Lexicon and Hangman classes, we will prescribe how they should work, like in previous assignments. They can be checked with `check50`. For the user interface, you have some freedom, but be careful to stick to the specification.

Finally, here are some general tips and tricks that might be useful:

- Letter position matters just as much as letter frequency. When computing word families, it's not enough to count the number of times a particular letter appears in a word; you also have to consider their positions. For example, "BEER" and "HERE" are in two different families even though they both have two E's in them. Consequently, representing word families as numbers representing the frequency of the letter in the word will get you into trouble.

- Watch out for gaps in the dictionary. When the user specifies a word length, you will need to check that there are indeed words of that length in the dictionary. You might initially assume that if the requested word length is less than the length of the longest word in the dictionary, there must be some word of that length. Unfortunately, the dictionary contains a few "gaps." The longest word in the dictionary has length 29, but there are no words of length 27 or 26. Be sure to take this into account when checking if a word length is valid.

## Steps

### 0. Before you get started

Testing for this assignment will again work automatically. Because you are going to write classes, it is now possible to test each class's functionality separately from the other parts of the program.

### 1. The `Lexicon` class

The first thing to implement is a class called `Lexicon`, which has the responsibility of managing the full word list and extracting words of a given length. It can be loaded once and asked for words whenever a new game is started.

Download the lexicons via:

	cd ~/module8
	wget https://prog2.mprog.nl/course/problems/hangman/dictionaries.zip
	unzip dictionary.zip

Create a file called `hangman.py` and add a `Lexicon` class. This class should have two methods: `__init__()` to initialize, and `get_words()` to extract a list of words with a  specific length to play Hangman:

    class Lexicon:

        def __init__(self):
            self.words = []
            # Load the dictionary of words.
            TODO

        def get_words(self, length):
            # Return a list of all words from the dictionary of the given length.
            TODO

In our code, we will use a **list** to store the master word list. That's why we have `self.words = []` at the top of the initializer method.

Now, implement those two methods.

> Note that the loading of words was demonstrated in last week's [Python lecture](/lectures/python)! It uses a **set** to store words, but you can modify it to use a list instead. Recall how to add items to a list?

### 2. Testing the `Lexicon`

Before we move on to the next step, we want to test if the class is working correctly. For example, try to get words of length 8 and see if the result seems reasonable. Start Python *interactively* using:

	python -i hangman.py

If you followed step 0 correctly, it should just load the Lexicon class and do nothing else. Then you could try some of the following.

	lex = Lexicon()
	words = lex.get_words(8)
	print(len(words))
	print(words.pop())
	print(words.pop())
	print(words.pop())

Note that the `pop` method for a list removes the last element from the list and returns it. So `print(words.pop())` removes the last element from the list of length-8 words and prints it.

Check if everything is in order. Is the number of words reasonable? It should be more than 10000 for length 8? Are each of the three random words actually 8 letters long? To add to this, in the description above, you can find some oddities that you might verify, too (e.g. how many words are there of length 27?).

> You should not put testing code like the above in `hangman.py` as you might have done in earlier assignments. This is because `check50` should be able to load your program and perform its own tests. Your tests would interfere with the checks.

You can now test using `check50` for the first time!

### 3. The `Hangman` class

So now we have a class to manage the master word list. We can also create a class that manages playing a game of Hangman. Let's think about what is needed to "play" a game.

First of all, a game is played based on a particular word length. Also, we decide upfront how many guesses will be allowed. These two are the only pieces of information that a Hangman game object needs to get started. This means that we know how it may be eventually initialized:

    game = Hangman(length=8, num_guesses=5)

Your code should follow the following structure. It is up to you to fill in the methods.

    class Hangman:
        def __init__(self, length, num_guesses):
            # Initialize the game.
            pass

        def guess(self, letter):
            # Update the game for a guess of letter. Return True if the letter
            # is added to the pattern, return False if it is not.
            pass

        def pattern(self):
            # Return a string of the current game pattern. Use underscores in 
            # place of missing letters. Example: "_AN_MAN".
            pass

        def guessed_string(self):
            # Produce a string of all letters guessed so far, in the order they
            # were guessed.
            pass

        def consistent_word(self):
            # Produce a word that is consistent with the current pattern.
            pass

        def finished(self):
            # Return True if the game is finished, otherwise False.
            pass

        def won(self):
            # Return True if the game is finished and the player has won, 
            # otherwise False.
            pass

        def lost(self):
            # Return True if the game is finished and the player has lost, 
            # otherwise False.
            pass

        def __str__(self):
            # Return a string representation of the game with some relevant
            # statistics.
            pass

After instantiating a new game, the resulting object should be able to respond to the following actions:

- Guessing a letter
- Producing the current pattern
- Producing the letters that have been guessed so far
- Producing a word that is consistent with the current pattern (to display to the user once the game has been lost)
- Checking if the game is finished
- Checking if the game has been won
- Checking if the game has been lost

To fully implement the game, the object should at least take care of tracking:

- Guessed letters up until now
- The set of currently remaining words
- The number of guesses remaining

Note that you can track other things, like the total number of guesses, or the current pattern (even though this is implicit in the word list and the guessed letters).

Purely for testing purposes, we would like to implement the `__str__` method, which allows us to call `print(game)` and look at a couple of stats. The resulting print could look something like the following:

	letters guessed are "aemnid", 201 words remaining, game not won

It's up to you to think about how you want to partition words into word families. Think about what data structures would be best for tracking word families and the master word list. Would an associative array work? How about a stack or queue? Thinking through the design before you start coding will save you a lot of time and headache.

Don't explicitly enumerate all potential new patterns. If you are working with a word of length `n`, then there are `2**n` possible patterns, and thus word families, for each letter. However, most of these families don't actually appear in the English language. For example, no English words contain three consecutive U's, and no word matches the pattern `E-EE-EE--E`. Rather than explicitly generating every pattern whenever the user enters a guess, see if you can generate patterns only for words that actually appear in the word list. One way to do this would be to scan over the word list, storing each word in a table mapping patterns to words in the corresponding family.

### 4. Testing the `Hangman` game

Let's test our game logic. We should be able to start a new game, and repeatedly guess letters. This is a perfect opportunity to use the `__str__` method, which gives us basics stats about the game --- which we use to verify its state.

Again, test your game interactively by running `python` and then `from hangman import *`, and entering the following commands, or a variation thereof:

	game = Hangman(8, 6)
	game.guess("e")
	print(game)
    game.guess("a")
	print(game)
    print(game.finished())
    print(game.consistent_word())
    print(game.pattern())
    game.guess("o")
    game.guess("i")
    game.guess("u")
    print(game.pattern())
    print(game)

### Intermezzo: handling exceptions

What happens when you want to create a Hangman game that does not follow the specifications? For example, what should happen if someone uses your class like the following:

	game = Hangman(-5, 6)

Try it yourself! Most likely, your code will indeed try to create a hangman game with a word of length -5. But that is not going not work, ever.

Because the `Hangman` object does not interface directly with the user, it makes no sense for it to reprompt the user for new input. However, we still need to be able to signal when something is wrong.

When you write code that does not interface with a person, you should still be able to deal with faulty input. To do so, Python (and many other languages) uses *exceptions*. Exceptions are like a warning signal: "Something went wrong! I'm stopping here!"

Try running the following Python code to see an exception being created:

    my_list = [1, 2, 3, 4]
    my_value = my_list[5]

There is no element in the list at index 5, so something goes wrong. You should get a message about an `IndexError`, which is indeed an "exception". It makes no sense to continue computation, because we don't know what the value of `my_value` should be. So the program halts.

In order for your Hangman class to be easy to work with, it should deal with wrong inputs using exceptions, instead of just going on. This means that running `game = Hangman(-5, 6)` should cause some kind of exception. For example:

    class Hangman:
        def __init__(self, length, num_guesses):
            if length < 1:
                raise Exception("Hangman word needs to have positive length.")
            if num_guesses < 1:
                raise Exception("You need at least one guess to play Hangman.")
            # ... and here follows other code.

Note that `check50` for this problem expects that such checks, with accompanying exceptions, are present in your code!


### 5. Implementing user interaction

While the `Hangman` class has all you need to play Hangman, someone who does not know your program won't understand that you have to write things like `game = Hangman(8, 6)` to start a game and `game.guess("e")` to guess a letter. So, let's make a **user interface**.

Your user interface should at least:

1. Prompt the user for how many letters the Hangman word should have. If the input is not a positive integer, or there is no word with that many letters, repeat the prompt until you get correct input.

2. Prompt the user for how many guesses she should get until she loses. This should be a positive integer.

3. Prompt the user for whether she wants to see detailed statistics of the game while playing (the statistics you put into the `__str__` method).

4. Play the game: repeatedly do the following

    1. Prompt the user for a guess. The guess should be a single letter that
       has not yet been guessed.

    2. Show an updated pattern, and the number of guesses remaining.

    3. Show detailed game statistics, if she asked for those.

    4. If the game has finished, either congratulate the player (on a win), or
       tell the player the Hangman word (any word that is consistent with the
       current pattern). Then ask the player if she wants to play again.

Like in "Game of Cards", the game code should be added inside an `if __name__ == '__main__':` condition. This ensures that that code will not run when checking, but will run when you test the program yourself using `python hangman.py`.

> Note that the program shown in the introduction at the top of the assignment is not a valid solution; it is just an illustration.

## Testing

	check50 minprog/cs50x/2019/hangman

## Extensions

The algorithm outlined in this handout is by no means optimal, and there are several cases in which it will make bad decisions. For example, suppose that a player has exactly one guess remaining and that computer has the following word list:

	DEAL   TEAR   MONK

If the player guesses the letter 'E' here, the computer will notice that the word family for the pattern `-E--` has two elements and the word family for the pattern `----` has just one. Consequently, it will pick the family containing DEAL and TEAR, revealing an E and giving the player another chance to guess. However, since the player has only one guess left, a much better decision would be to pick the family `----` containing MONK, causing the player to lose the game.

There are several other places in which the algorithm does not function ideally. For example, suppose that after the player guesses a letter, you find that there are two word families, the family `--E-` containing 10,000 words and the family `----` containing 9,000 words. Which family should the computer pick? If the computer picks the first family, it will end up with more words, but because it revealed a letter the player will have more chances to guess the words that are left. On the other hand, if the computer picks the family `----`, the computer will have fewer words left but the player will have fewer guesses as well. More generally, picking the largest word family is not necessarily the best way to cause the player to lose. Often, picking a smaller family will be better.

After you implement this assignment, take some time to think over possible improvements to the algorithm. You might weight the word families using some metric other than size. You might consider
having the computer "look ahead" a step or two by considering what actions it might take in the future.

If you implement something interesting, make sure to document your partition method well by describing your changes in detail!

## Submitting

To submit this assignment, you need to submit all of your source files. If you have created your own evil algorithm (see the section Extensions), then include a short description in a text file of what you've written.
