## Testing in Ruby

### Getting Started
Tests are written in their own files and are run separately from the 'production code' of the system that does the actual work. To use the built-in testing framework that comes in Ruby, you have to write test files that follow this pattern.

``` rb
require 'test/unit'

# The class name should be the name of the class you are testing followed by Test
class MyClassTest < Test::Unit::TestCase

  # The method name be the method you are testing and what you are expecting, must start with the word test
  def my_test_method_returns_5
  end

end
```

### Writing a Test
When writing a test, you want to follow a few guidelines. Each test needs to set up a situation in your code. Then it needs to act on that situation and `assert` a result. This style is usually referred to as a 'Given', 'When', 'Then' style of testing.

``` rb
  def test_greater_than_five_returns_true
    my_num = 10 #GIVEN (my_num is 10)
    result = greater_than_five?(my_num) #WHEN greater_than_five? is called with my_num
    assert_equal(true, result) #THEN expect that the result of calling our method should return true
  end
```

### Using Assertions
Assertions are making a statement about your code that must be true for the test to pass. The assertions you'll be using most often are.

  - `assert(truthy_value)`
  - `assert_equal(expected_value, actual_value)`
  - `assert_not_nil(not_nil_value)`
  - `assert_raise(ErrorClassName) do end`

You can read the full list [here](http://ruby-doc.org/stdlib-2.1.2/libdoc/test/unit/rdoc/Test/Unit/Assertions.html#method-i-assert_raise).

### Running Tests
To run your tests, simply pass the name of the file to Ruby: `ruby spec/card_test.rb` and you'll be able to see the output.


### Full Example
Let's say I have a class called Card. This is how my Card class and Card test class may look.

This file would be `lib/card.rb`:

``` rb
class Card
  attr_accessor :value, :suit

  def initialize(value, suit)
    @value = value
    @suit  = suit
    @face_up = false
  end

  def display
    " #{value} #{suit} "
  end

  def flip
    if @face_up
      @face_up = true
    else
      @face_up = false
    end
  end

  def is_face_up?
    @face_up
  end
end
```

This file would be `spec/card_test.rb`:

``` rb
require 'test/unit'
require_relative '../lib/card'

class CardTest < Test::Unit::TestCase
  def test_can_create_card
    assert(Card.new(10, :hearts))
  end

  def test_display_returns_suit_and_value
    card = Card.new(10, :hearts)
    card_display = card.display
    assert_equal(" 10 hearts ", card_display)
  end

  def test_card_can_be_flipped
    card = Card.new(10, :hearts)
    assert_equal(false, card.is_face_up?)
    card.flip
    assert_equal(true, card.is_face_up?)
  end
end
```


### Additional Resources
  - [Unit Testing in Ruby](http://en.wikibooks.org/wiki/Ruby_Programming/Unit_testing)
  - [Ruby The Hard Way: Unit Testing (not for the faint of heart)](http://learnrubythehardway.org/book/ex47.html)
  - [What is TDD?](http://c2.com/cgi/wiki?TestDrivenDevelopment)
  - [TDD Flow Chart Graphic](http://luizricardo.org/wordpress/wp-content/upload-files/2014/05/tdd_flow.gif)
  - [TDD Katas](https://github.com/garora/TDD-Katas)
