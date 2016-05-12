# Reverse Test-Driven Development

by That TDD Fellow

Oleksii Fedorov

[@waterlink000](https://twitter.com/waterlink000)



## Code <- Test Relationship


Production code is the most important part


Test suite makes sure production code is correct


Test suite enables easy refactoring


Test suite gives courage to introduce a change (new feature, bug fix, etc.)


Test suite is coupled to the code it tests



## Verifying knowledge coverage

How can one verify if specific knowledge in production code is covered by test suite?


### Knowledge in Production Code?


```ruby
variable_name = some.value(from, other, source)
```

this is the knowledge


```ruby
if <knowledge>
...
```


```ruby
if ...
  <knowledge>
...
```


```ruby
...
elsif <knowledge>
...
```


```ruby
...
else
  <knowledge>
end
```


```ruby
something.doAction(this, is, a, knowledge, too)
```


```ruby
<knowledge>.each do |item|
  ...
end
```


```ruby
....each do |item|
  <knowledge>
end
```


```ruby
return <knowledge>
```


### Got it. How to verify it?


### Break it.


Introduce a very small change to the knowledge. Test suite should fail.

If it doesn't - knowledge is not covered well enough.



## Code -> Test Relationship


Knowledge in the production code should be verified by test suite


Knowledge change is an act of verification if test suite is correct (or thorough enough)


Knowledge change (Mutation) is a test for the test.



## Mutational testing


1. Narrow scope to single granular piece of knowledge
2. Break this knowledge (simple change, Mutation)
3. Make sure there is a test suite failure
4. Restore knowledge to the original state



## Reverse TDD


1\. Narrow scope to some manageable knowledge and isolate it (method/function/class/module)


2\. Read the code and try to understand one granular piece of knowledge it is doing


3\. Write a test to verify your assumption


4\. Make sure it passes (by fixing your assumption, or fixing production code (bugs))


5\. Apply Mutational Testing to each related granular piece of knowledge to verify that your understanding (and the test is correct)

(this may introduce more tests)


6\. Go back to 2


### Recap

1. Narrow scope and isolate it
2. Read, try to understand, pick granular piece of knowledge
3. Write a test
4. Make sure test passes
5. Apply Mutational Testing repeatedly
6. Go back to 2



## I want to see it in action!

*(step-by-step example)*


### Narrow & Isolate


Means of isolation:

- Extract completely to module/class/package of its own.
- Duplicate the code under the test and put it into function/method(s) with
  different distinguishable name.


#### Our scope


![original code](original_code.png)


### Step 1: Duplicate code to different method

```ruby
class User
  def notifications
    notifications = Database
      .where("notifications") do |x|
    ...
  end

  # full copy of User#notifications
  def notifications__isolated__
    notifications = Database
      .where("notifications") do |x|
    ...
  end
end
```


### Step 2: Read, try to understand and pick knowledge to test

```ruby
(x[1][0] == "followed_notification" && x[1][2] == id.to_s) ||
...
```

```ruby
if kind == "followed_notification"
  {
    kind: kind,
    follower: User.find(values[1].to_i),
    user: User.find(values[2].to_i),
  }
elsif ...
```


### Step 3: Write your first test

```ruby
it "looks like it loads some notifications from the database" do
  user = User.new(email: "john@example.org", password: "welcome")
  Database.insert("notifications", [
    "followed_notification", "986", user.id.to_s]
  )

  notifications = user.notifications__isolated__

  expect(notifications.count).to eq(1)
end
```


### Step 4: Make sure test passes

```
F

Failures:

  1) User#notifications looks like it loads some notifications from the database
     Failure/Error: user = User.new(email: "john@example.org", password: "welcome")

     NoMethodError:
       undefined method `+' for nil:NilClass
     # ./lemon.rb:475:in `insert'
     # ./lemon.rb:57:in `initialize'
     # ./lemon_spec.rb:58:in `new'
     # ./lemon_spec.rb:58:in `block (3 levels) in <top (required)>'

Finished in 0.00228 seconds (files took 0.11769 seconds to load)
1 example, 1 failure
```


```ruby
module Database
  def self.insert(table, values)
    ...
    # Fails with NoMethodError when there are no rows in table
    id = table.map { |row| id = row[0] }.max + 1
    ...
  end
end
```


Fix

```ruby
# Fall back to 1, when there are no rows in table
id = (table.map { |row| id = row[0] }.max || 0) + 1
```


```
.

Finished in 0.02343 seconds (files took 0.11584 seconds to load)
1 example, 0 failures
```


### Step 5: Apply Mutational Testing repeatedly


Break first granular piece of knowledge:

```ruby
# (x[1][0] == "followed_notification" && x[1][2] == id.to_s) ||
...
```

And run tests:

```
F

Failures:

  1) User#notifications looks like it loads some notifications from the database
     Failure/Error: expect(notifications.count).to eq(1)

       expected: 1
            got: 0

       (compared using ==)
     # ./lemon_spec.rb:63:in `block (3 levels) in <top (required)>'

Finished in 0.03511 seconds (files took 0.11877 seconds to load)
1 example, 1 failure
```


Great. This mutation says, that our tests are good to go. Other options:

- replace condition with `false`
- replace condition with `true`


Replacing condition with `true` does this:

```
.

Finished in 0.02343 seconds (files took 0.11584 seconds to load)
1 example, 0 failures
```


Now we have to either:

- change our understanding if it is not what we expect, or
- change our test to cover that, or
- add more tests.


In this case adding another test cuts it:

```ruby
it "ignores records of invalid kind" do
  user = User.new(email: "john@example.org", password: "welcome")
  Database.insert("notifications", [
    "invalid", "986", user.id.to_s
  ])

  notifications = user.notifications__isolated__

  expect(notifications.count).to eq(0)
end
```


```
.F

Failures:

  1) User#notifications ignores records of invalid kind
     Failure/Error: expect(notifications.count).to eq(0)

       expected: 0
            got: 1

       (compared using ==)
     # ./lemon_spec.rb:131:in `block (3 levels) in <top (required)>'

Finished in 0.21531 seconds (files took 0.11582 seconds to load)
2 examples, 1 failure
```


## Q & A



## Thank you

Twitter: [twitter.com/waterlink000](https://twitter.com/waterlink000)

Github: [github.com/waterlink](https://github.com/waterlink)

Blog: [tddfellow.com](http://tddfellow.com)
