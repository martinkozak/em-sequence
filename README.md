EventMachine Sequencer
======================

**em-sequence** provides third approach to [EventMachine][4] lightweight 
concurrency. Runs declared methods and blocks in sequence, each in one 
tick. So allows calls chaining with keeping the EventMachine multiplexing 
facility on. 

See an example. For first, let's define some (of sure, slightly 
non-sense) calculator class:

```ruby
class Calculator
    def some_method(var, &block)
        block.call(1, var)
    end
    
    def other_method(x, y, &block)
        block.call(x + y)
    end
    
    def another_method(z, &block)
        block.call(z ** 2)
    end
end
```

And then declare and run multiplexed code:

```ruby
EM::run do
    EM::Sequence::new(Calculator::new).declare {
    
        # variable declaration
        variable :var, 3
 
        # method call declaration
        some_method(:var) { [:x, :y] }              #   | TICK 1
                                                    #   V
        # inline block declaration and definition
        block(:x, :y) do |x, y|                     #   | TICK 2
            {:x => x + 1, :y => y + 1}              #   |
        end                                         #   V
 
        # some other methods
        other_method(:x, :y) { :x }                 #   V TICK 3
        another_method(:x)                          #   V TICK 4
        
    }.run! do |result|                              #   | TICK 5
        puts result.inspect                         #   |
    end                                             #   V
end
```

It will print out the number `36`. It's the same as linear 
non-multiplexed (and non-callbacked calculator):

```ruby
# variable declaration                              #   | TICK 1
calc = Calculator::new                              #   |
var = 3                                             #   |
                                                    #   |
# method call declaration                           #   |
x, y = calc.some_method(var)                        #   |
                                                    #   |
# inline block declaration and definition           #   |
x += 1                                              #   |
y += 1                                              #   |
                                                    #   |
# some other methods                                #   |
x = calc.other_method(x, y)                         #   |
result = calc.another_method(x)                     #   |
                                                    #   |
puts result.inspect                                 #   V
```

If you don't expect any result, you can simply call:

```ruby
EM::Sequence::run(Calculator::new) do
    # some declaration, see above
end
```

Copyright
---------

Copyright &copy; 2011 &ndash; 2015 [Martin Poljak][3]. See `LICENSE.txt` for
further details.

[2]: http://github.com/martinkozak/em-sequence/issues
[3]: http://www.martinpoljak.net/
[4]: http://rubyeventmachine.com/
