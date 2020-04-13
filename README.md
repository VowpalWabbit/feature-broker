# Table of contents
* [Introduction](#introduction)
* [Problem Description](#problem-description)
* [Solution: FeatureBroker](#solution-featurebroker)
* [FeatureBroker Library](#featurebroker-library)


# Introduction
Mature software applications are highly componentized and have well-defined API boundaries among components. This encapsulation leads to information (i.e. features) getting siloed. Exposing these features for the purpose of inference (via a machine learning model, say) requires changes to the API surface. Expanding the API on a feature by feature basis is costly and onerous. 

The FeatureBroker library presents a solution to this problem. It manages the conversation between the client code that provides features (inputs) and consumes inference results (outputs) from a CB model. We believe that this is a pervasive problem especially in mature software stacks. 

The goal of open sourcing this library is to reduce the developer friction in adopting [VW](https://github.com/VowpalWabbit), [slim VW](https://github.com/VowpalWabbit/vowpal_wabbit/tree/master/vowpalwabbit/slim) and [Azure Personalization Service](https://azure.microsoft.com/en-us/services/cognitive-services/personalizer/) in software stacks that are built with these constraints.  

# Problem Description
Generalizing a bit further, large software stacks encounter the following problem in making inference during a running process:

* Owners want components to have a well-defined, stable public API surface. Expanding the API on a feature-by-feature basis is onerous.
* Components producing features are often not running in the same thread as components using those features.
* Changes in feature values at runtime (e.g., network type could change
  mid-call) require updating the prediction gracefully.

The idea of a simple shared thread-safe key-value feature store proved
insufficient for two primary reasons:
1. Coherence: Inferencing components need feature values to be stable and consistent. However, feature providing components can update those values asynchronously.
2. Hierarchy: Features may be consumed by multiple models and inferencing components. The most graceful way to handle this was to make the structure hierarchical, and allow it to be "forked" for subcomponents.

# Solution: `FeatureBroker`
The shared structure responsible for handling these problems is a FeatureBroker. It manages the conversation between the client code that provides features(inputs) and consumes inference results (outputs) from a model. Specifically,
the key operations of the `FeatureBroker` are:
1. Binding Inputs: Given a name and type returns an "input pipe" into which a feature providing component can feed values.
2. Associating Models: Register and describe a scheme for transforming inputs to outputs.
3. Binding Outputs: Returns an "output pipe" in which inference consuming models can query for updated values.

Note that binding is distinct from feeding or getting values; the former happens during object set-up, the latter during runtime. As such there are two distinct phases in which the FeatureBroker comes into use:
1. Set-up phase: The model instances are associated with the FeatureBroker with the model's inputs and outputs bound through pipes. A "hierarchical" arrangement of feature brokers may also be employed by forking to set-up "child" contexts and brokers.
2. Usage phase: The inputs are fed via input pipes, and outputs are consumed by output pipes. This distinction is made to ensure that we frontload most of the runtime checking to the set-up phase (where it is self-contained and happens at often predictable times).

# FeatureBroker Library

The library has been publically pubished [here](https://github.com/microsoft/FeatureBroker).