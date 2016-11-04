---
layout: post
title:  "Creating Computer-Generated Poetry"
date:   2016-10-31 10:27:24 -0400

---
This is the notebook that Sven Anderson and I used for our "Family Weekend" class over the weekend. We began the class with two poems from an NPR article called, "[Human or Machine: Can you tell who wrote these poems?](http://www.npr.org/sections/alltechconsidered/2016/06/27/480639265/human-or-machine-can-you-tell-who-wrote-these-poems)"

The question of what constitutes poetic style has and will continue to produce varied and complicated answers, but, at bottom, it comes down to choice. This experiment in generating "poetry" represents one way of simulating this process of choice by randomly picking words according to their frequency in a particular poet's corpus. (Just to note, some of the functions in this script are in a separate file (`functions.py`) to make it easier to read. You can access those functions and the accompanying text files [here](https://www.dropbox.com/sh/zme9hxf35ewdtvm/AADiV4PHy7i0xtF8uOBl__Xfa?dl=0).)

### Objective: We will be able to articulate specific features of poetic writing that can distinguish computer-generated poems from ones written by people. 

In this class, we'll be working with python code in this <a href="http://jupyter.org">Jupyter notebook</a> interface. What's great about this platform is that we can easily move between formatted text (as illustrated here), code, images, and pretty much any other type of digital object. 

We will use this notebook to produce a randomly-generated poem based on a particular poet's style. 

![lincolnCloud]({{site.baseurl}}/assets/wordCloud.png)
*A word cloud generated from Abraham Lincoln's *Gettysburg Address*. The size of the words correspond to their frequency in the speech.*

#### How might we mathematically represent authorial choice?
We can start by attempting to simulate the process by which particular authors make stylistic choices -- for instance, how they choose which words to use. 

In programming, we produce simulations using **algorithms**, which are sets of rules for the computer to follow. They are basically computational recipes for turning a given input into an output. Here we create an algorithm in a **function** for calculating the tip for a meal.


```python
def tipCalculator (mealTotal): ### Our "mealTotal" is our input. 
    return mealTotal * 0.20 + mealTotal ### We add 20% of the total to the input to get our output

## We've created the function, now we can run it: 
print(tipCalculator(50.00))
```

    60.0


#### Now let's think about how we might simulate poetic word choice to produce randomly generated poetry. 


```python
# Load some functionality and some pre-computed probabilities.
import functions 
import pickle 
dickinsonProbs = pickle.load(open('dickinsonProbs.p', 'rb'))
```


```python
print(dickinsonProbs['the'])
print(dickinsonProbs['heart'])
```

    0.06506088945704097
    0.0009419363520150709


These numbers tell us the likelihood of picking each word if we were to draw them at random from the text: the more frequent the word in the text, the greater chance of choosing it. 


```python
from random import random 
def randomWord(probs):
    '''Given probs, a dictionary of word probabilities, this
    returns a word according to how frequently that word is found
    in the dictionary.'''
    rnum = random()
    sumprob = 0.0
    for k in probs.keys():
        sumprob += probs[k]
        if sumprob > rnum:
            return k
    return k
```

We can simulate this process by using a function that randomly chooses words from Dickinson's poetry, based on her word usage. 


```python
print(randomWord(dickinsonProbs))
```

    drum



```python
# what happens if 10 is replaced by 20?
for i in range(10): 
    print(randomWord(dickinsonProbs))
```

    since
    but
    them
    most
    few
    sinks
    one
    repeal
    the
    delight


This frequency-based approach gives us a way to simulate the poet's process of choosing words. But after we choose a random word what comes next? 


```python
dickinsonBigrams = pickle.load(open('dickinsonBigrams.p', 'rb'))
shakespeareProbs = pickle.load(open('shakeProbs.p', 'rb'))
shakespeareBigrams = pickle.load(open('shakeBigrams.p', 'rb'))
```


```python
print(shakespeareBigrams['compare'])
```

    {'thou': 0.3333333333333333, 'with': 0.16666666666666666, 'myself': 0.16666666666666666, 'thee': 0.16666666666666666, 'them': 0.16666666666666666}


We can choose the next word based on which words are likely to follow the first word, according to the poet's usage. In Shakespeare's Sonnets, *thou* is the most likely word to follow *compare*. 

With this process of randomly choosing words likely to follow one another we can build entire poems. In the following algorithm, we combine this process with a method of making the poems rhyme by matching the last word of every odd line with a random rhyming word in every even line. 


```python
def generatePoem(probs, bigrams, lineLength, poemLength):
    poemLines = [] 
    # create poemLines from probabilities
    for i in range(poemLength):
        line = functions.generateFromBigrams(probs, bigrams, lineLength)
        poemLines.append(line)
    lineCount = 1 
    newLines = [] 
    for line in poemLines: 
        if lineCount % 2 == 1: # an odd line sets the rhyme
            rhymedLine = line 
            newLines.append(rhymedLine)
            lineCount += 1 
        elif lineCount % 2 == 0: # an even line must match rhyme of 
            rhymingLine = line  # preceding line
            try: 
                if rhymedLine[-1] in functions.pronounce.keys(): 
                    newWord = functions.rhyme(rhymedLine[-1], 3)[0]
                    rhymingLine[-1] = newWord # fix the rhyme to match
                    newLines.append(rhymingLine)
                    lineCount += 1
            except: 
                newLines.append(rhymingLine)
                lineCount += 1 
    # Now we concatenate to a single string.
    fullPoem = [] 
    for line in newLines:
        line.append('\n')
        fullLine = ' '.join(word for word in line)
        fullPoem.append(fullLine)
    return ''.join(line for line in fullPoem)
```

Now we can generate our own "poems" from the word choice of either Shakespeare or Emily Dickinson.

### Generate Shakespearean poetry

For this code to work, change the hashtags to numbers -- the first for the number of words you want in each line and the second for the number of lines you want in the poem. 

(I really like the concluding couplet of the Shakespearean poem.)

```python
print(generatePoem(shakespeareProbs, shakespeareBigrams, 8, 14 ))
```
    praises worse than at that i not so affined 
    that our time and eyes have might to give 
    in my bosoms ward but let me do forgive 
    yore those children nursed deliverd from serving with thee 
    which in the strength and they look what thee 
    from the first the morning have been mine eye 
    thy fair assistance in loves fire shall in ai 
    yet the perfumed tincture of my desire keep open 
    they see barren tender feeling but yet eyes alagappan 
    doubting the face should transport me that is kind 
    enforced to my deepest sense to his brief affined 
    and gives thee back the past i have often 
    not my love thy store when days when acetaminophen 
    


### Generate Dickinsonian Poery


```python
print(generatePoem(dickinsonProbs, dickinsonBigrams, ##, ##))
```

    for heaven she had come nor simplified her 
    consummate plush how cold i had the her 
    off for pearl then of love is finished 
    you lost was there came out time abolished 
    held but internal evidence gives us is overcome 
    he stayed away upon the merchant smiled branscome 