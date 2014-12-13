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



