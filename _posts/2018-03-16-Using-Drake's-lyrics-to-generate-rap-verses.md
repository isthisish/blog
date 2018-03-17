---
published: true
layout: post
title: Using Drake's lyrics to generate rap verses
excerpt_separator:  <!--more-->
---

Markov chains are a very interesting concept and are used to model and understand real world processes like music, speech and text. In this post we will go over a simple task to explain how Markov Models work.

From [Wikipedia](https://en.wikipedia.org/wiki/Markov_model):
`In probability theory, a Markov model is a stochastic model used to model randomly changing systems. It is assumed that future states depend only on the current state, not on the events that occurred before it (that is, it assumes the Markov property). Generally, this assumption enables reasoning and computation with the model that would otherwise be intractable. For this reason, in the fields of predictive modelling and probabilistic forecasting, it is desirable for a given model to exhibit the Markov property`

So what does this mean?

Before understanding how Markov Chains work and what they are, we need to understand what a Markov Process is.

`A process is a Markov process if one can make predictions for the future of the process based solely on its present state just as well as one could knowing the process's full history, hence independently from such history; i.e., conditional on the present state of the system, its future and past states are independent.` [1] 

 For example, if you made a Markov chain model of a baby's behavior, you might include "playing," "eating", "sleeping," and "crying" as states, which together with other behaviors could form a 'state space': a list of all possible states. In addition, on top of the state space, a Markov chain tells you the probabilitiy of hopping, or "transitioning," from one state to any other state---e.g., the chance that a baby currently playing will fall asleep in the next five minutes without crying first. [2]


This [website](http://setosa.io/ev/markov-chains/) has some excellent examples and visualizations to help you understand how Markov Chains work.

Given this knowledge of Markov Chains we can now apply them to our problem of creating a model that learns Drake's Lyrics and then spits out some rap verses.

To get started, I used [this website](https://www.axs.com/drake-s-5-best-lyrics-verses-56089) to see which songs have "good" lyrics and then compiled them all into a file.

- With markov chains we can model a system given the transition probabilities. 
- When we are at a certain state i.e. a word in this case, we can look at our learned dictionary of words to see what words appear after what words and choose randomly from those words.
- So we basically look at text and see what words appear after certain words.
- For a markov chain of order 1, we would only look for 1 word. For 2 order we would look at for e.g. What appears after "all of" in sample text below. for order 1 we would only look after "all".

```
# Let's start with a simple sentence.
sample_string = "all of my homies write all of their code in C and on their linux machines while carrying my guns"
```
```
class Markov_chain(object):
    def __init__ (self, order):
        self.order = order
        self.group_size = self.order + 1
        self.text = None
        self.graph = {}
        
    def train (self, filename=''):
        if filename != '':
            file = open(filename,'r')
            self.text = file.read().split()
            file.close()
        else:
            self.text = sample_string.split() #use sample_string if no file is provided
        
        self.text = self.text + self.text[:self.group_size] 
        for i in range(0, len(self.text) - self.group_size):
            key = tuple(self.text[i :i + self.order] )
            value = self.text[i + self.order]
            if key in self.graph:
                self.graph[key].append(value)
            else:
                self.graph[key] = [value]
```
So what's happening in the piece of code above.

Other than initialization, we:

- We take beginning of text and append it to the end. We do this because usually we won't have last word in a sentence come before some word in a short piece of text. In the above example, we
don't have any word that comes after guns. 

- Key would be words that come before the word we are looking at (based on order we are using) if it is order 1 we are looking at 1 word.

- Value is the word that comes after key. In the sample text when we start, key is 'all' and value is 'of'.

- If the key already exists in our dictionary we add the newly encountered word to the list of values

```
    def generate (self,length):
        #start at a random place 
        start_i = random.randint(0, len(self.text) - self.order)
        result = self.text[start_i : start_i + self.order]
        
        for i in range(length):
            state = tuple(result[len(result) - self.order :])
            next_word = random.choice(self.graph[state])
            result.append(next_word)
        return " ".join(result[self.order : ] )
```

- We start at a random place.

- The state we are in is the last few word(s) (based on our order). For order 1 it is just the last word.

- Next we randomly pick a word from the words that appeared after our current state.

- For eg: 'of' has words 'my' and 'their' in the list. so we randomly pick from this list

- We create a Markov_chain of order 1 and use our lyrics text file.
```
marko = Markov_chain(1)
marko.train('Drake_lyrics.txt')
print(marko.generate(100))
```

And this is what the output looks like!

`
little when I'm ready, what's really much explaining Story stayed the bottom now we're here Started from the money, come and find me, myself and a sprained ankle boy into the outcome Labels want the bottom now we're here Started from the bottom now we're here Started from the whole team here, Started from the bottom now the whole team fucking here getting naked I apologize it right back" Nigga, I just hope that Tuck my brothers, I might be yelling out, "Money over everything, money on my business Killing all this shit, I know good and a gun CJ's
`

It generates pretty believable lyrics for a rap song in 2018 ;) 

Because Drake tends to repeat a lot of words sometimes we might end up with a phrase that is directly from a real song. Solution for now is to use order of 1. As an improvement of this program, we should deal at sentence level and only add sentences that are not present in training text.

The complete code and lyrics document is available [here](https://github.com/harpreetsingh91)

This tutorial is based on/inspired by http://il.pycon.org/2016/static/sessions/omer-nevo.pdf
and https://lauris.github.io/text-generation-markov-chain


References:

[1] https://en.wikipedia.org/wiki/Markov_chain 

[2] [http://setosa.io/ev/markov-chains/](http://setosa.io/ev/markov-chains/)


