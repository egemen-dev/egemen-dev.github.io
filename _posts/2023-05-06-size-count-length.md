# Size, Count, and Length Walk into a Rails App: Who Wins?

In Ruby on Rails we got three similar looking methods `count()`, `length()` and `size()`. Let's see what they do under the hood.


Consider we got a variable that holds all the users.
```
  users = User.all
```

### COUNT
```
  users.count

  => SELECT COUNT(*) FROM “users”
```
`count()` always performs `SQL COUNT` query, no matter how many times you call it.

<br>

### LENGTH
```
  users.length

  => SELECT "users".* FROM "users"
```
`length()` checks if the data is present in the memory. If not, it loads the users data to the memory via SQL query and then calculates how many records are in the collection - no additional query gets executed to get the size of the collection. Note that it is not going to perform another SQL query next time you call it because the data is already stored in the memory. It's only going to output the size of the collection. So, it might come in handy if you're going to use the data later on.

<br>

### SIZE
```
  users.size

  => SELECT COUNT(*) FROM "users"
```
`size()` will check if the data is present in the memory. If the data is present, it calculates the size of the collection like `lenght()`. Otherwise it performs an `SQL COUNT` query.

<br>

### TAKE AWAY 🍔
Winner is the Rails. `size()` and `length()` methods are distinguished when the data is not present in the memory. `size()` performs `SQL COUNT` query in order to give you the result while `length()` loads the data via SQL query, stores it in the memory and then outputs the size of the collection. Their behaviours are equivalent when the data is already loaded.


Source: [rails api docs](https://api.rubyonrails.org/)
