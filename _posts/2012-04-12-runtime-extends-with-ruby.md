---
layout: post
title: Runtime extends with Ruby
published: true
author: Dawid Sklodowski
author_role: Lead Developer
author_url: http://github.com/dawid-sklodowski
author_avatar: http://www.gravatar.com/avatar/073c19a8d1fd30baa6dba34eaa55fe90.png
excerpt: '<img src="/images/2012-04-12/ruby.jpg" style="float:left; width: 150px; height: 150px; margin: 3px 20px 10px 0px;"/>
Ruby is a dynamic language which supports many ways to organise logic. We can use class inheritance or/and compose our classes by including selected modules (mixins). We can define or un-define methods on the fly. We can even use methods that are not really defined (using method_missing).  Using characters in an RPG game as an example, this code walk-through explains a design pattern for extending objects with new methods at run-time.'
---

Ruby is a dynamic language that supports many ways to organise logic. We can use class inheritance or/and compose our classes by including selected modules (mixins).

<img src="/images/2012-04-12/ruby.jpg" style="float:left; width: 150px; height: 150px; margin: 3px 20px 10px 0px;"/>

We can define or un-define methods on the fly. We can even use methods that aren't really defined (using method_missing). Another powerful feature is the ability to extend an object with new methods at run-time, by including modules in the class or singleton class (if you want to extend only one instance).

To present this design pattern, lets assume that we want to create an application which will entertain our users — an RPG game, where the plot takes place in a fantasy world.

For simplicity we will model a character class. The player can pick one of six different races: dwarf, elf, gnome, hobbit, human or ogre. And they must chose their characters occupation from: priest, programmer, smith, thief, warrior or wizard.

Our base character class is going to have a public method: greeting, where the output will depend on a character's race and occupation. To illustrate this, lets start with a test:

{% highlight ruby %}
#spec/character_spec.rb

require 'spec_helper'
require 'character'

describe Character do
  describe '#greeting' do
    it 'works for ogre warrior' do
      ogre = Character.new(:race=>'ogre', :occupation=>'warrior')
      ogre.greeting.should == 'Grumph! I will kill you!'
    end

    it 'works for elven wizard' do
      elf = Character.new(:race=>'elf', :occupation=>'wizard')
      elf.greeting.should == 'Heil! Did you see my staff?'
    end

    it 'works for hobbit thief' do
      hobbit = Character.new(:race=>'hobbit', :occupation=>'thief')
      hobbit.greeting.should == "Good Morning. Haven't you lost something?"
    end

    it 'works for gnome programmer' do
      gnome = Character.new(:race=>'gnome', :occupation=>'programmer')
      gnome.greeting.should == 'Guten Tag.Do You Know Ruby?'
    end

    it 'works for dwarf smith' do
      dwarf = Character.new(:race=>'dwarf', :occupation=>'smith')
      dwarf.greeting.should == 'Humpf! Your sword needs to be fixed.'
    end

    it 'works for human priest' do
      human = Character.new(:race=>'human', :occupation=>'priest')
      human.greeting.should == 'Good Day. Only Chosen One knows his path!'
    end

    it 'works for human programmer' do
      human = Character.new(:race=>'human', :occupation=>'programmer')
      human.greeting.should == 'Good Day. Do you know Ruby?'
    end
  end
end
{% endhighlight %}

`Character#greeting` is composed of two parts. The first depends on how a given race performs a greeting.  For example an ogre will say Grumph!. The second part is influenced by a character's occupation, e.g. a programmer will ask about Ruby. By combining the above an 'ogre programmer' will greet you with: 'Grumph! Do you know Ruby?'

With our tests in a failing state, lets think about some possible implementations.

The easiest way to make our tests pass is to create just one class – Character – composed of multiple if (or case) statements, each modifying the output of the greeting.  However, it is likely that other attributes could be introduced in the future, resulting in complicated logic that would be difficult to maintain.

Another approach might separate the logic into classes, each inheriting from the Character class. This solution is nicely supported by Rails through Single Table Inheritance (STI). Going with this approach is good for one layer of separation. For instance, when we separate logic based on the character's race, we can create classes corresponding to races such as: Dwarf, Elf, Gnome, Hobbit, Human, Ogre, but this is not our case. We want to separate logic by both race _and_ occupation. This would lead to two layers and 36 classes like following: OgreProgrammer, OgrePriest, GnomeThief, HobbitWizard etc. And this number will grow when even more layers are added. We could end with thousands of classes like FemaleYoungWoodenElfArcher!

The solution I would like to present uses run-time extends with Ruby. We create a module for each race and occupation and some logic to glue things together.

Lets start with Character class:

{% highlight ruby %}
#lib/character.rb

class Character
  include Character::Race
  include Character::Occupation

  def greeting
    "#{race_greeting} #{occupation_greeting}"
  end

  def race_greeting
    raise 'Not implemented'
  end

  def occupation_greeting
    raise 'Not implemented'
  end
end
{% endhighlight %}

The Character class implements `greeting`, which depends on two other methods: `race_greeting` and `occupation_greeting`. Those two methods are expected to be implemented in the modules included in lines: 4 and 5. Also those methods are defined in the Character class, but they raise an error to indicate that they should be defined elsewhere.

Lets continue with implementation of modules that were included in Character class, Race and Occupation:

{% highlight ruby %}
#lib/character/race.rb

class Character
  module Race

    def initialize(options={})
      @race = options[:race]
      include_race
      super if defined? super
    end

    def race_module
      ActiveSupport::Inflector::constantize("Character::Race::#{@race.capitalize}")
    end

    private
    def include_race
      singleton_class = class << self;self;end;
      singleton_class.send(:include, race_module)
    end
  end
end


#lib/character/occupation.rb

class Character
  module Occupation

    def initialize(options={})
      @occupation = options[:occupation]
      include_occupation
      super if defined? super
    end

    def occupation_module
      ActiveSupport::Inflector::constantize("Character::Occupation::#{@occupation.capitalize}")
    end

    private
    def include_occupation
      singleton_class = class << self;self;end;
      singleton_class.send(:include, occupation_module)
    end
  end
end
{% endhighlight %}

Those two modules look similar and can be refactored, but we will examine that later. For now take a look at the Occupation module. 

The initialize method sets an instance variable @occupation and then calls `include_occupation`, which includes the chosen occupation to the object's singleton class (this means that this module is available only for this object, not for all Character's objects). The `occupation_module` method returns the module to be included (using ActiveSupport's constantize).

Finally the super call in the `initialize` method calls initialize in any other module/class through inheritance. This is important, because it calls not only the `initialize` method of Character class, but it also `initialize` defined in all modules, which had been included before the described one was included. It assures that both: `initialize` defined in the Race module and in the Occupation module are both called.  The Race module works in the same way.  

The last thing we need to implement are the modules for each race and occupation. Since they are quite similar, I’m only going to list two here:

{% highlight ruby %}
#lib/character/race/ogre.rb

class Character
  module Race
    module Ogre
      def race_greeting
        'Grumph!'
      end
    end
  end
end


#lib/character/race/human.rb

class Character
  module Race
    module Human
      def race_greeting
        'Good Day.'
      end
    end
  end
end

#lib/character/occupation/programmer.rb

class Character
  module Occupation
    module Programmer
      def occupation_greeting
        'Do you know Ruby?'
      end
    end
  end
end


#lib/character/occupation/wizard.rb

class Character
  module Occupation
    module Wizard
      def occupation_greeting
        "Did you see my staff?"
      end
    end
  end
end
{% endhighlight %}

Implementing all required modules and requiring ActiveSupport in the Character class, makes our tests pass.

{% highlight ruby %}
#lib/character.rb

require 'rubygems'
require 'active_support'
{% endhighlight %}


Changing existing objects at runtime
------------------------------------

So far we’ve implemented a structure that allows us to set character’s race and occupation at object creation, using the new method.
However, this doesn’t fulfil our need, we need to be able to change the existing character’s occupation and race (this is some kind of magic) at run-time. This can be easily achieved by improving our modules. Lets write some tests first:

{% highlight ruby %}

#spec/character_spec.rb

describe Character do
  context 'attribute readers' do
    it 'are being set during initialization' do
      ogre = Character.new(:race=>'ogre', :occupation=>'warrior')
      ogre.race.should == 'ogre'
      ogre.occupation.should == 'warrior'
    end

    it 'are changeable' do
      character = Character.new(:race=>'ogre', :occupation=>'warrior')
      character.race = 'elf'
      character.occupation = 'smith'
      character.race.should == 'elf'
      character.occupation.should == 'smith'
    end
  end
  
  describe '#greeting' do
    #Some code removed for clarity
    it 'works for hobbit priest who was dwarf thief' do
      character = Character.new(:race=>'dwarf', :occupation=>'thief')
      character.race = 'hobbit'
      character.occupation = 'priest'
      character.greeting.should == 'Good Morning. Only Chosen One knows his path!'
    end
  end
end
{% endhighlight %}

To make this pass we need to add two methods to the Character and Occupation classes:

{% highlight ruby %}

#lib/character.rb

class Character
  module Race

    def self.included(base)
      base.send(:attr_reader, :race)
    end

    def race=(value)
      @race = value
      include_race
    end

    #Rest of code removed for clarity.
  end
end
{% endhighlight %}

The first method is called when the module is included in another module or class (parent class/module is being held by variable base) (this is the Character class in our case). This method just sets an attribute reader for the race variable on the Character class. 

The second method is an attribute writer, which assigns the value to an object’s instance variable, and then includes the appropriate race module.

Playing with it even more
-------------------------

To make things complicated, lets implement gnome’s speaking dialect for our characters. Gnomes are very smart and they have a lot to say, so their dialect should speak faster. Gnomes will omit the pauses in between words and use special accents to substitute that. They will say for example:
HowAreYouDoing? instead of How are you doing?.

To depict our needs lets start with modification of character’s tests:

{% highlight ruby %}
#spec/character_spec.rb

describe Character do
  describe '#greeting' do
    it 'works for gnome programmer' do
      gnome = Character.new(:race=>'gnome', :occupation=>'programmer')
      gnome.greeting.should == 'GutenTag.DoYouKnowRuby?'
    end
    #Rest of tests omited for clarity.
  end
end
{% endhighlight %}

To implement that we need to modify the Character class again:

{% highlight ruby %}

#lib/character.rb
class Character
  include Character::Race
  include Character::Occupation

  def greeting
    if race_module.methods.include?(:race_modifier)
      race_module.race_modifier(clean_greeting)
    else
      clean_greeting
    end
  end

  def clean_greeting
    "#{race_greeting} #{occupation_greeting}"
  end
  #Rest of code omitted for clarity
{% endhighlight %}

So now, if `race_modifier` has been implemented in `race_module` then it should be used, otherwise an unmodified `clean_greeting` will be returned. Finally we implement this `race_modifier` in the Gnome module:

{% highlight ruby %}

#lib/character/race/gnome.rb

class Character
  module Race
    module Gnome
    
      def race_greeting
        'Guten Tag.'
      end
      
      def self.race_modifier(value)
        value.split(' ').map(&:capitalize).join
      end
      
    end
  end
end
{% endhighlight %}

This simple example of speech modification for gnomes illustrates how easily and cleanly logic can be extended using this run-time extends design pattern.

Some refactoring
----------------

So far we have a lot of code duplication. The Race and Occupation modules look almost identical. We can address by creating a Common module to be included in a Race and Occupation class:

{% highlight ruby %}
#lib/character/race.rb

class Character
  module Race
    include Common
  end
end

#lib/character/occupation.rb

class Character
  module Occupation
    include Common
  end
end

#lib/character/common.rb

module Common
  def self.included(base)
    base_name = base.name.split('::').last.downcase
    base.send(:attr_reader, base_name.to_sym)
    base.class_eval <<-EOS
      def initialize(options={})
        @#{base_name} = options[:#{base_name}]
        include_#{base_name}
        super if defined? super
      end

      def #{base_name}=(value)
        @#{base_name} = value
        include_#{base_name}
      end

      def #{base_name}_module
        ActiveSupport::Inflector::constantize("Character::#{base_name.capitalize}::\#{@#{base_name}.capitalize}")
      end

      private
        def include_#{base_name}
          singleton_class = class << self;self;end;
          singleton_class.send(:include, #{base_name}_module)
        end
    EOS
  end
end
{% endhighlight %}

This code uses the `included` hook to read the name of the including module and then, set a reader for the attribute (with that name) and define its methods (using class_eval).

Wrapping up
-----------

Complete code from this blog post can be found here:
[http://github.com/dawid-sklodowski/runtime-extends](http://github.com/dawid-sklodowski/runtime-extends "Runtime Extends with Ruby")
