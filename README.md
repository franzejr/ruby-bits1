ruby-bits1
==========

##### Conditional Return

```ruby
search = "Contra"
games = ["Super Mario Bros.", "Contra", "Metroid", "Mega Man 2"]
search_index = games.find_index(search)
search_result = if search_index
   "Game #{search} found: #{games[search_index]} at index #{search_index}."
else
"Game #{search} not found."
end
puts search_result
```



##### Short Circuit Evaluation

Before:

```ruby
def search_index(games, search_term)
  search_index = games.find_index(search_term)

  if search_index
    search_index
  else
    "Not Found"
  end
end
```


After:

```ruby
def search_index(games, search_term)
  games.find_index(search_term) || "Not Found" 
end
```


##### Optional Arguments
Before

```ruby
def new_game(name, year, system)
  {
    name: name,
    year: year,
    system: system
  }
end
game = new_game("Street Figher II", nil, nil)
```

After

```ruby
def new_game(name, year = nil, system = nil)
  {
    name: name,
    year: year,
    system: system
  }
end
game = new_game("Street Figher II")
```

##### Hash Arguments

Before

```ruby
def new_game(name, year=nil, system=nil)
  {
    name: name,
    year: year,
    system: system
  }
end
game = new_game("Street Figher II", "SNES", 1992)
```

After
```
def new_game(name, options = {})
  {
    name: name,
    year: options[:year],
    system: options[:system]
  }
end

game = new_game("Street Figher II",year: 1992,system:"SNES")
```

##### Raise Exception

Before

```ruby
class InvalidGameError < StandardError; end
def new_game(name, options={})
  {
    name: name,
    year: options[:year],
    system: options[:system]
  }
end
begin
  game = new_game(nil)
rescue InvalidGameError => e
  puts "There was a problem creating your new game: #{e.message}"
end
```

After

```ruby
class InvalidGameError < StandardError; end
def new_game(name, options={})
  unless name
    raise InvalidGameError
  end
    {
    name: name,
    year: options[:year],
    system: options[:system]
  }
end
begin
  game = new_game(nil)
rescue InvalidGameError => e
  puts "There was a problem creating your new game: #{e.message}"
end
```


##### Splat Operator

[https://endofline.wordpress.com/2011/01/21/the-strange-ruby-splat/]


Before

```ruby
def describe_favorites(games)
  for game in games
    puts "Favorite Game: #{game}"
  end  
end
describe_favorites(['Mario', 'Contra', 'Metroid'])
```

After

```ruby
def describe_favorites(*games)
  for game in games
    puts "Favorite Game: #{game}"
  end  
end

describe_favorites('Mario', 'Contra', 'Metroid')
```


##### Inheritance

```ruby
class Game
  attr_accessor :name, :year, :system
  attr_reader :created_at
 
  def initialize(name, options={})
    self.name = name
    self.year = options[:year]
    self.system = options[:system]
    @created_at = Time.now
  end
end
```

Adding an attribute to the child class
```ruby
class ArcadeGame < Game
  attr_accessor :weight
  def initialize(name,options={})
    super(name,options)
    self.weight = options[:weight]
  end
end
class ConsoleGame < Game
end
```

##### Hashes

```ruby
def exclude_character(games, character)
  games.except(character)
end

games = {
  mario: ["Super Mario World", "Super Smash Bros. Melee"],
  luigi: ["Luigi's Mansion"],
  yoshi: ["Yoshi's Island", "Yoshi's Story"]
}
puts exclude_character(games, :yoshi)
```


```ruby
def difference_between(player1, player2)
  player1.diff(player2)
end

mario_favorite = {
  sports: "Mario Sports Mix",
  action: "Super Mario World"
}

luigi_favorite = {
  sports: "Golf",
  action: "Super Mario World"
}

puts difference_between(mario_favorite, luigi_favorite)
```



##### Creating class methods for a module

```ruby
class Game
  class << self
  include GameUtils
  end
end

Game.find_all_from_user("Gregg")
```


##### Object extend
[http://www.railstips.org/blog/archives/2009/05/15/include-vs-extend-in-ruby/]
[http://stackoverflow.com/a/5008349/1311732]

```ruby
game = Game.new("Contra")
game.extend(Playable)
game.play
```

##### Hook Methods

```ruby
module LibraryUtils
  
  def self.included(base)
    base.extend(ClassMethods)
  end

  def add_game(game)
  end

  def remove_game(game)
  end

  module ClassMethods
    def search_by_game_name(name)
    end
  end
end

class AtariLibrary
  include LibraryUtils
end
```

##### ActiveSupport::Concerns

```ruby
module LibraryUtils

  extend ActiveSupport::Concern
  
  included do
    load_game_list
  end

  def add_game(game)
  end

  def remove_game(game)
  end

  module ClassMethods
    def search_by_game_name(name)
    end

    def load_game_list
    end
  end
end
```


##### Yield and blocks

```ruby
class Library
  attr_accessor :games

  def initialize(games = [])
    self.games = games
  end

  def each_on_system(system)
    games.each do |game|
      yield if game.system == system
    end
  end
end
```

##### Refactoring with blocks


Before

```ruby
class Game
  attr_accessor :name, :year, :system
  attr_reader :created_at

  def initialize(name, options={})
    self.name = name
    self.year = options[:year]
    self.system = options[:system]
    @created_at = Time.now
  end

  def play
    begin
      emulator = Emulator.new(system)
      emulator.play(self)
    rescue Exception => e
      puts "Emulator failed: #{e}"
    end
  end

  def screenshot
    begin
      emulator = Emulator.new(system)
      emulator.start(self)
      emulator.screenshot
    rescue Exception => e
      puts "Emulator failed: #{e}"
    end
  end
end
```

After


```ruby
class Game
  attr_accessor :name, :year, :system
  attr_reader :created_at

  def initialize(name, options={})
    self.name = name
    self.year = options[:year]
    self.system = options[:system]
    @created_at = Time.now
  end

  def play
   	emulate {
      @emulator.play(self)
      }
  end

  def screenshot
    emulate {  
      @emulator.start(self)
      @emulator.screenshot 
      }
  end
  
  private
  def emulate
    begin
      @emulator = Emulator.new(system)
      yield @emulator
    rescue Exception => e
      puts "Emulator failed: #{e}"
    end
  end
end
```
