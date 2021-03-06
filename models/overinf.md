---
layout: model
title: Overinformativeness
model-status: code
model-category: Reasoning about Reasoning
model-tags: language, pragmatics, overinformativeness
model-language: webppl
model-language-version: v0.9.7
---

This is a model of the production of referring expressions, based on the vanilla model of Frank & Goodman 2012, but equipped with a relaxed semantics. It is described further in [Degen et al (under review)](https://arxiv.org/abs/1903.08237). 

The context assumed in the models below is the following: 

![Image of critical context](../assets/img/size-sufficient.png "Fig 1a")

The speaker's task is to produce a referring expression that allows a listener to identify the target, the small blue pin. A referring expression including a size adjective (*the small pin*) is strictly speaking sufficient for uniquely establishing reference to the target, yet speakers often "overmodify" with color, producing referring expressions like  *the small blue pin*. This overmodification phenomenon is what the second model below captures via the relaxed semantics mechanism.

## Model

### Vanilla model

We start with a model that has a Boolean semantics for color and size terms.

~~~~
var alpha = 1
var costWeight = 1
var c = 0

var states = ["big_blue","big_red","small_blue"]

var utterances = ["big", "small", "blue", "red", "big_blue", "small_blue", "big_red"]     	

var statePrior = function() {
	return uniformDraw(states)
};

var utterancePrior = function() {
  return uniformDraw(utterances)
};

var meaning = function(utterance, obj){
  _.includes(obj, utterance)
}

var cost = {
	big: 0,
	small: 0,
	blue: 0,
	red: 0,
	big_blue: c,
	small_blue: c,
	big_red: c
}

// literal listener
var literalListener = cache(function(utt) {
  return Infer({method:"enumerate"},
               function(){
    var state = statePrior()
    condition(meaning(utt,state))
    return state
  })
});

// pragmatic speaker
var speaker = cache(function(state) {
  return Infer({method:"enumerate"},
               function(){
    var utt = utterancePrior()
    factor(alpha * literalListener(utt).score(state) - costWeight * cost[utt])
    return utt
  })
});

display("speaker who wants to communicate big blue object:")
viz.table(speaker("big_blue"))

display("speaker who wants to communicate big red object:")
viz.table(speaker("big_red"))

display("speaker who wants to communicate small blue object:")
viz.table(speaker("small_blue"))
~~~~

Change the cost. Convince yourself that increasing the cost of complex utterances quickly drives their probability to 0, i.e., "overinformative" expressions won't be produced.


## Relaxed semantics model

~~~~
var alpha = 30
var costWeight = 1
var size_semvalue = 0.8
var color_semvalue = 0.99
var size_cost = 0
var color_cost = 0

var states = [
	{size: "big", color: "blue"},
	{size: "small", color: "blue"},
	{size: "big", color: "red"}]

var utterances = ["big", "small", "blue", "red", "big_blue", "small_blue", "big_red"]     

var colors = ["red", "blue"]
var sizes = ["big", "small"]	

var statePrior = function() {
	return uniformDraw(states)
};

var utterancePrior = function() {
  return uniformDraw(utterances)
};

// assumes that 2-word utterances consist of SIZE_COLOR, in that order
var meaning = function(utt, obj) {
  var splitWords = utt.split('_')
  if (splitWords.length == 1) {
    var word = splitWords[0]
    if(_.includes(colors, word))
      return word == obj.color ? color_semvalue : 1-color_semvalue;
    else if (_.includes(sizes, word))
      return word == obj.size ? size_semvalue : 1-size_semvalue;
  } else if (splitWords.length == 2) {
    var size_value = splitWords[0] == obj.size ? size_semvalue : 1-size_semvalue;
    var color_value = splitWords[1] == obj.color ? color_semvalue : 1-color_semvalue;
    return size_value*color_value
  } else 
    console.error("bad utterance length: "+splitWords.length)
};

var cost = {
	big: size_cost,
	small: size_cost,
	blue: color_cost,
	red: color_cost,
	big_blue: size_cost+color_cost,
	small_blue: size_cost+color_cost,
	big_red: size_cost+color_cost
}

// literal listener
var literalListener = cache(function(utt) {
  return Infer({method:"enumerate"},
               function(){
    var state = statePrior()
    factor(meaning(utt,state))
    return state
  })
});

// pragmatic speaker
var speaker = cache(function(state) {
  return Infer({method:"enumerate"},
               function(){
    var utt = utterancePrior()
    factor(alpha * literalListener(utt).score(state) - costWeight * cost[utt])
    return utt
  })
});


display("speaker who wants to communicate big blue object:")
viz.table(speaker({size: "big", color: "blue"}))

display("speaker who wants to communicate big red object:")
viz.table(speaker({size: "big", color: "red"}))

display("speaker who wants to communicate small blue object:")
viz.table(speaker({size: "small", color: "blue"}))

display("literal listener who observes 'big':")
viz.table(literalListener("big"))

display("literal listener who observes 'small':")
viz.table(literalListener("small"))

display("literal listener who observes 'blue':")
viz.table(literalListener("blue"))

display("literal listener who observes 'red':")
viz.table(literalListener("red"))

display("literal listener who observes 'big blue':")
viz.table(literalListener("big_blue"))

display("literal listener who observes 'big red':")
viz.table(literalListener("big_red"))

display("literal listener who observes 'small blue':")
viz.table(literalListener("small_blue"))
~~~~

1. Play around with the semantic value of size and color. For what parameter values can you generate the reported size/color asymmetry in the probability of producing an "overinformative" expression? When does the asymmetry disappear or reverse?

2. How does increasing size and color cost change speaker behavior?

3. Add additional objects to the context. How does this affect speaker behavior?