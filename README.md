natural
=======

"Natural" is a general natural language facility for nodejs. Stemming,
classification, phonetics and some inflection are currently supported.

It's still in the VERY (and I mean VERY) early stages, and I'm VERY (yes, again,
VERY) interested in bug reports, contributions and the like.

At the moment most algorithms are English-specific but long-term some diversity
is in order.

Installation
------------

If you're just looking to consume natural without your own node application
please install the NPM

    npm install natural
    
If you're interested in contributing to natural or just hacking it then by all
means fork away!
    
Stemmers
--------

Currently stemming is supported via the Porter and Lancaster (Paice/Husk)
algorithms.

    var natural = require('natural');
    
this example uses a porter stemmer. "word" is returned.

    console.log(natural.PorterStemmer.stem("words")); // stem a single word
    
attach() patches stem() and tokenizeAndStem() to String as a shortcut to
PorterStemmer.stem(token). tokenizeAndStem() breaks text up into single words
and returns an array of stemmed tokens.

    natural.PorterStemmer.attach();
    console.log("i am waking up to the sounds of chainsaws".tokenizeAndStem());
    console.log("chainsaws".stem());

the same thing can be done with a lancaster stemmer

    natural.LancasterStemmer.attach();
    console.log("i am waking up to the sounds of chainsaws".tokenizeAndStem());
    console.log("chainsaws".stem());

Naive Bayes Classifier
----------------------

    var natural = require('natural'), 
    	classifier = new natural.BayesClassifier();

you can train the classifier on sample text. it will use reasonable defaults to
tokenize and stem the text.

    classifier.train([{classification: 'buy', text: "i am long qqqq"},
                  {classification: 'buy', text: "buy the q's"},
                  {classification: 'sell', text: "short gold"},
                  {classification: 'sell', text: "sell gold"}
    ]);

outputs "sell"

    console.log(classifier.classify('i am short silver'));

outputs "buy"

    console.log(classifier.classify('i am long copper'));

    classifier = new natural.BayesClassifier();

the classifier can also be trained on and classify arrays of tokens, strings, or
any mixture. arrays let you use entirely custom data with  your own
tokenization/stemming if any at all.

    classifier.train([{classification: 'hockey', text: ['puck', 'shoot']},
                  {classification: 'hockey', text: 'goalies stop pucks.'},
                  {classification: 'stocks', text: ['stop', 'loss']},
                  {classification: 'stocks', text: 'creat a stop order'}
                  ]);

    console.log(classifier.classify('stop out at $100'));
    console.log(classifier.classify('stop the puck, fool!'));
    
    console.log(classifier.classify(['stop', 'out']));
    console.log(classifier.classify(['stop', 'puck', 'fool']));

A classifier can also be persisted and recalled so you can reuse a training.

    var classifier = new natural.BayesClassifier();
    
    classifier.train([{classification: 'buy', text: ['long', 'qqqq']},
                  {classification: 'buy', text: "buy the q's"},
                  {classification: 'sell', text: "short gold"},
                  {classification: 'sell', text: ['sell', 'gold']}
    ]);
        
persist to a file on disk named "classifier.json"

    classifier.save('classifier.json', function(err, classifier) {
        // the classifier is saved to the classifier.json file!
    });
    
and to recall from the classifier.json saved above:

    natural.BayesClassifier.load('classifier.json', function(err, classifier) {
        console.log(classifier.classify('long SUNW'));
        console.log(classifier.classify('short SUNW'));
    });

Phonetics
---------

Phonetic matching (sounds-like) matching can be done with either the SoundEx or
Metaphone algorithms

    var natural = require('natural'),
        metaphone = natural.Metaphone, soundEx = natural.SoundEx;

    var wordA = 'phonetics';
    var wordB = 'fonetix';

test the two words to see if they sound alike

    if(metaphone.compare(wordA, wordB))
        console.log('they sound alike!');
        
the raw phonetics are obtained with process()

    console.log(metaphone.process('phonetics'));

attaching will patch String with useful methods

    metaphone.attach();

soundsLike is essentially a shortcut to Metaphone.compare

    if(wordA.soundsLike(wordB))
        console.log('they sound alike!');
        
the raw phonetics are obtained with phonetics()

    console.log('phonetics'.phonetics());   

same module operations apply with SoundEx

    if(soundEx.compare(wordA, wordB))
        console.log('they sound alike!');

    console.log(soundEx.process('phonetics'));

the same String patches apply with soundex

    soundEx.attach();

    if(wordA.soundsLike(wordB))
        console.log('they sound alike!');
        
    console.log('phonetics'.phonetics());
    
Inflectors
----------

Nouns can be pluralized, singularized and counted with inflectors

    var natural = require('natural'),
    nounInflector = new natural.NounInflector;
    
to pluralize a word (outputs "radii")

    console.log(nounInflector.pluralize('radius'));

to singularize a word (outputs "beer")

    console.log(nounInflector.singularize('beers'));

like many of the other features String can be patchedto perform the operations
directly. the "Noun" suffix to the methods is necessary as verbs will be
supported in the future.

    nounInflector.attach();
    console.log('radius'.pluralizeNoun());
    console.log('beers'.singularizeNoun());   

counters can also be produced from integers with CountInflector
    
    countInflector = natural.CountInflector;

outputs "1st"

    console.log(countInflector.nth(1));

outputs "111th"

    console.log(countInflector.nth(111));


Copyright
---------

Copyright (c) 2011 Chris Umbel
