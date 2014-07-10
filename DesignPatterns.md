##Design Patterns Save the Day
Oftentimes while writing software, we'll come upon a class in our system that does the thing that we're trying to implement...*almost*. It's basic algorithm behaves the same way, but the specifics of each step (of the algorithm) may be different. Our first thought may be create a new class for our new functionality, which duplicates a lot of the code that we already have. This approach is certainly not [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself), and it just doesn't feel right.

For example, let's say that we want to write a program that allows our users to play either Tic Tac Toe or Chess. We create our Tic Tac Toe game, and it works just great. But as we're ramping up to start writing the Chess class, we realize that though the specific rules of these two games are very different, the fundamental **Game Loop** that drives each is the same:

1. We **set up the game** by creating a game board and assigning each player a pieces (either X's and O's or black and white).

2. We allow the first **player to take a turn**.

3. We **switch to the other player**, and allow them to take a turn as well.

4. We then alternate between each player **until the game is over**.

Instead of rewriting all of that code for each game, wouldn't it be great if we could just reuse the general algorithm, and change the specific details of each game? We can, meet the **Template Method**.

####The Template Method Pattern

The goal of the Template Method is to extract all of the generic parts of our algorithm, and put them into a base class. Then, we can inherit this behavior into our subclasses, and override the more specific implementation details as needed.

Let's try it with our game program that we talked about above. We'll create an abstract ```Game``` class which takes care of the **Game Loop**:
 ```
class Game
  def play_the_game
    set_up_game
    current_player = 0
    until game_over?
      player_turn(current_player)
      current_player = (current_player + 1) % 2 #this switches players
    end
    display_the_winner
  end
end
```

Then, we can create our ```TicTacToe``` and ```Chess``` classes, both of which inherit from ```Game```. In the Template Method, the base class acts as an *abstract class*. Which means, that though it mentions our methods: ```set_up_game```, ```game_over?```, ```player_turn``` and ```display_the_winner```, it doesn't actually define them. This job is left up to each of the subclasses:

```
class TicTacToe < Game
  def set_up_game
    #create the Tic Tac Toe board
    #assign an X or O for each player
  end

  def player_turn(current_player)
    #get the current player's move
  end

  def game_over?
    #return true if there's a winner or a tie
    #return false if the game is still in progress
  end

  def display_the_winner
    #prints out who wins - X or O
  end
end
```
```
class Chess < Game
  def set_up_game
    #create the Chess board
    #assign a color piece for each player
  end

  def player_turn(current_player)
    #get the current player's move
  end

  def game_over?
    #return true if there's a checkmate
    #return false if no winner yet
  end

  def display_the_winner
    #prints out who wins - black or white
  end
end
```

To play either of these games, we can just instantiate a new game object:
```
ttt_game = TicTacToe.new
chess_game = Chess.new
```
And then call our play_the_game method on our new game objects:
```
ttt_game.play_the_game
chess_game.play_the_game
```
By using the Template Method, we've eliminated the need for duplication as well as created a nice structure that allows us extend this program easily. For example, if we want to add some other games to our program, like Connect Four we can just create a ```ConnectFour``` class that also inherits from ```Game```!


-----------------------------


Now, let's focus our attention specifically on the Tic Tac Toe game. Let's say that if the user is playing against the computer, that they can choose to either play against an unbeatable computer (that uses the minimax algorithm), or a beatable computer that just randomly chooses its next move.

We could potentially just include all of this logic right in a ```ComputerPlayer``` class, by using an if/elsif statement to determine how the computer chooses its next move:
```
class ComputerPlayer
  def make_a_move(strategy)
    if strategy == "unbeatable"
      unbeatable_strategy
    elsif strategy == "random"
      random_strategy
    end
  end

  def unbeatable_strategy
    #minimax algorithm
  end

  def random_strategy
    #randomly chooses a move
  end
end
```
And this method may work just fine for now... but what if we decide that we want to add several new strategies for the computer to use when taking a turn. Perhaps we want the computer to always lose, or maybe we want to create a brand new unbeatable algorithm. If this were the case, we would not only have to add these new strategy functions, but also change our current ```get_move``` logic - which breaks the Open-Closed Principle. So, instead, let's try the Strategy Method.

####The Strategy Pattern

The Strategy Pattern allows us to select how we would like an algorithm to behave, at runtime. Rather than hardcoding the behavior into our code. In other words, we're able to create a "family of algorithms" that can be used interchangabley, depending on the scenario.

In our example, we want to be able to choose which strategy our ```ComputerPlayer``` in going to use when making a move. Our "family of algorithms" will include a ```MinimaxStrategy``` and a ```RandomStrategy```. We'll want to be able to plug them into the ```ComputerPlayer``` in the same way, so they must all have the same interface with the ```ComputerPlayer```.

First, let's look at our ```ComputerPlayer```. When a ```ComputerPlayer``` object is created, it will be initialized with a ```game_strategy``` object passed in. The ```game_strategy``` is saved to an instance variable, which we can then call ```get_move``` on.
```
class ComputerPlayer
  attr_accessor :game_strategy

  def initialize(game_strategy)
    @game_strategy = game_strategy
  end

  def make_a_move
    @game_strategy.get_move
  end
end
```
Now, the only thing that is required of the strategy objects (```MinimaxStrategy``` and ```RandomStrategy```) is for them to both have a ```get_move``` method defined within them, which encapuslates the specific alogrithm for that strategy:

```
class MinimaxStrategy
  def get_move
    #logic to get the best move using the minimax algorithm
  end
end
```
```
class RandomStrategy
  def get_move
    #gets a random move
  end
end
```

Then, we can create two different ```ComputerPlayer``` instances. One which utilizes the minimax strategy, and the other which uses the random strategy.

```
unbeatable_computer_player = ComputerPlayer.new(MinimaxStrategy.new)
unbeatable_computer_player.make_a_move

random_computer_player = ComputerPlayer.new(RandomStrategy.new)
random_computer_player.make_a_move
```

The benefit with this pattern comes in how easy it to extend the ```ComputerPlayer``` behavior: we can easily new strategy objects to our system. And, if we ever want to alter the computer's behavior, we can just pass that new strategy to our ```ComputerPlayer```. This change is completely encapsulated in the specific strategy objects, and the ```ComputerPlayer``` doesn't need to know a thing!


-----------------------

Resources:

* [Agile Software Development: Principles, Patterns, and Practices by Uncle Bob](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445)

* https://github.com/nslocum/design-patterns-in-ruby

* http://en.wikipedia.org/wiki/Template_method_pattern

* http://reefpoints.dockyard.com/ruby/2013/07/10/design-patterns-template-pattern.html

* http://en.wikipedia.org/wiki/Strategy_pattern

* http://reefpoints.dockyard.com/2013/07/25/design-patterns-strategy-pattern.html


