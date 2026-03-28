
# What is Ai?
- Artificial Intelligence is the field of study focused on creating systems that can perform tasks that normally require human intelligence
- Example: 'I work in Ai research'
- When people call Gemini, DeepSeek or Copilot an Ai, this is technically imprecise, they are Ai models (specifially Large Language Models or LLMs)
- However, language evolves and "an AI" is becoming accepted shorthand for "an artificial intelligence system"


# What is an LLM?
- An LLM is a Large Language Model
- An LLM is a specific type of neural network designed to learn languages (training), receive prompts and provide responses (inference)


# What is training?
- Training is when we optimize the weights and biases w/in a neural network


# What is inference?
- Inference is when we use a trained Ai model to generate a response


# How does inference work?
1. The LLM receives a `prompt` (text) & a `temperature` (number)
2. The LLM outputs a `logit` (number) for each token in its vocabulary
    - Higher logit = more confident this token is the optimal next token
    - Example: Token A Logit: `2`, Token B Logit: `-1`, Token C Logit: `0`
3. IF the token w/ the highest logit is the `end-of-sample` token (`</sample>`) THEN the process ends
4. Logits are scaled by temperature
    - `scaled_logit = logit / temperature`
    - Token A Example:
        - `2 / 1.0` -> `2.0` -> Even temperature -> No change to logits
        - `2 / 0.5` -> `4.0` -> Small temperature -> Creates bigger logit spread
        - `2 / 1.5` -> `1.3` -> Big temperature -> Creates more uniform logits
5. Each token goes through a `softmax` function    
    - Softmax ensures the percentage of all tokens w/in the vocabulary, based on their `scaled_logit`, adds up to `100%`
    - Temp `1.0` Example: Token A Probability: `84.4%`, Token B Probability: `4.2%`, Token C Probability: `11.4%`
    - Temp `0.5` Example: Token A Probability: `98.0%`, Token B Probability: `0.2%`, Token C Probability: `1.8%`
    - Temp `1.5` Example: Token A Probability: `64.0%`, Token B Probability: `8.7%`, Token C Probability: `27.3%`
6. Sort tokens from most likely to least likely
7. Choose a random number between `0` & `1` (ex: `0.45`)
8. Token A is chosen b/c the random number falls between `0` & `0.844`
9. IF the randomly chosen token is the `end-of-sample` token (`</sample>`) THEN the process ends
10. This process repeats so the next input becomes the original prompt plus all tokens generated so far


# What is temperature?
- Temperature controls randomness
- Temperature = `0` makes the model always pick the most likely token (deterministic)
- Higher values (ex: `0.8`, `1.2`) make the distribution more uniform, increasing creativity but also risk of incoherence
- Low temperature (e.g., 0.1): The model almost always picks the most likely token. Output is deterministic, repetitive, but safe.

High temperature (e.g., 1.2): Lower probability tokens have a better chance. Output is more creative, surprising, but sometimes nonsensical.

Temperature = 0: Not technically valid (division by zero). Implementations treat this as "always pick the highest logit."

Softmax	Converts logits to probabilities (sum to 1.0)
Temperature	Divides logits before softmax; controls randomness
Scaled distribution	The probability distribution after temperature is applied
Sampling	Choosing a token randomly according to these probabilities
        - Lower = more deterministic
        - Higher = more random


# What is a sample?
- A sample is a training example that includes at least 1 prompt and 1 model response
- A sample may also include code snippets and sources
- Example:
    ```txt
    Prompt: "What's the capital of France?"
    Response: "Paris"
    ```


# What is a multi-turn sample?
- A multi-turn sample is a sample w/ multiple prompts and responses
- A multi-turn sample teaches the model how to:
    - Have a conversation
    - Ask good follow up questions
    - Build on previous responses
- Example:
    ```txt
    Prompt: "What's the capital of France?"
    Response: "Paris"
    Prompt: "What's the population?"
    Response: "Approximately 2.1 million people in the city proper."
    ```


# What is a corpus?
- A corpus is a collection of samples


# What is a token?
- A token is a piece of text that an Ai model understands as a single unit
- Tokens can be:
    - Words
    - Parts of words
    - Punctuation
    - Individual characters
- Spaces are typically attached to the following word & not separate tokens


# What is a tokenizer?
- A tokenizer is a tool that converts text into unique number ID's + vice versa


# What is BPE?
- BPE stands for Byte Pair Encoding
- BPE is a method for deciding how to split text into tokens
- BPE learns from the corpus which character & token combinations appear most frequently together, then merges them into tokens
- BPE training can merge ANY two adjacent tokens, regardless of whether they're special, requested, or learned tokens


# How does BPE work?
- With BPE we split the training corpus into individual characters, spaces and punctuations marks
- Then we count how often each adjacent pair of tokens appears next to each other
- Then we compute the most frequent pair (if the pair includes a special token we do not count it)
- IF the most frequent pair occurs more then `bpe_min_merge_frequency` (defaults to 3) (configurable in `train.xml`) times THEN merge them into a new token and repeat the process ELSE stop merging
- Post merge we have 1 new token (merged tokens) & we keep the unmerged tokens w/in the vocabulary too
- We don't merge special tokens b/c they are structural tokens & it's good for the Ai model to learn them as individual units (ex: `<sample>`, `</sample>`, `<unknown />`)


# What are merge rules?
- Merge rules tell us how to build bigger tokens from smaller ones
- When we get NEW text (not in training corpus) (like a user prompt), we apply the merge rules to tokenize the text
- Example: The sequence has `console.log("hi world")`, vocabulary has `console` & `console.log `so the merge rules will make it so the sequence token is the largest token we have in the vocabulary (`console.log`)
- In training, post merge, the pre merge tokens remain w/in the vocabulary


# What is the vocabulary?
- All the tokens BPE learned w/in the training corpus
- If a prompt is given to the model, & the prompt includes a token that is brand new then BPE will assign the unknown token ID (a special token) for this token (`<unknown />`) 


# What are special tokens?
- Special tokens are pre-added to vocabulary & determined by Poppins
- Structural tokens that define the corpus format (ex: `<sample>`, `</sample>`, `<unknown />`)
- They can NOT be merged with adjacent tokens to form larger tokens
- They appear in the token sequence as single units
- We don't merge special tokens b/c they are structural tokens & it's good for the Ai model to learn them as individual units (ex: `<sample>`, `</sample>`, `<unknown />`)


# What are requested tokens?
- Requested tokens are pre-added to vocabulary & determined by the developer
- Requested tokens can be merged with adjacent tokens to form larger tokens
- They appear in the token sequence as single units (unless merged to make even larger tokens)
- Example: If `console.log` is a requested token then it starts as one token, & during BPE training, if `console.log` + `(` appears frequently then BPE will create a token for `console.log(` while maintaining in the vocabulary the token for `console.log` (requested token) & `(` (learned token)


# What is embedding?
- Embedding is the process of turning a token into a token embedding


# What is a token embedding?
- A token embedding is a vector of numbers that represents the meaning of a token
- Token embeddings are just weights. They are initialized randomly and then updated during training just like every other weight in the network.
- During training, the model learns to move these vectors so that similar tokens have similar vectors.
- The embeddings are learned representations. They start random and become meaningful through training.
- After training:
    - Token `5` might be near tokens `6`, `7`, `eight`
    - Token `apples` might be near `oranges`, `bananas`, `fruit`


# What is a vector?
- A vector is an ordered list of numbers
- Example:
    - `[-1.5, 0, 1, 2, 3]`
    - This is a vector with 5 numbers
    - We call this a 5-dimensional vector because it has 5 numbers in it
    - The first number in the vector is `-1.5` and the last number is `3`


# What is a dimension?
- A dimension is a slot, index or position within a vector (ex: the value is `1` @ dimension `2`)
- A dimension is also the size of a vector (ex: the vector w/ `9` numbers in it is a 9-dimensional vector)


# What is a matrix?
- A matrix is list of vectors that all have the same length (2D grid of numbers with rows and columns)
- Example: 
    ```txt
    [1, 2, 3]
    [4, 5, 6]
    [7, 8, 9]
    ```


# What is embedding dimension?
- Embedding dimension is the length of a token embedding vector
- The embedding dimension annotation is `embedding_dim` or `d`
- More embedding dimensions = more expressive power = more memory / computation required


# What is a weight?
- Weights are learned parameters (numbers)
- Weights are updated during training & fixed during inference
- Weights control how much each input contributes
- Raw Weights are `f32` (big numbers that require 4 bytes to store)
- Quantized Ternary Weights are `-1`, `0` or `1` (require 2 bits to store)


# What is Ternary?
- Ternary means 3 possible values


# What is Quantization?
- 32-bit floating point numbers (`f32`) can represent most numbers with high precision, example:
    - `0.000000001`
    - `3.1415926535`
- Precise (`f32`) numbers take up 4 bytes (32 bits) for every value
- Quantization is the process of taking numbers that need many bits (`3.1415926535`) and mapping them to numbers that need fewer bits (`3.14`)


# What is Ternary Quantization?
- Ternary Quantizaion is a technique where we keep raw weights in `f32` & create quantized weights that can be `-1`, `0` or `+1`
- Raw values require 32 bits but quantized values (what we use in inference) only require 2 bits to tells us the value (`0b01` tells us `-1`, `0b00` tells us `0` & `0b10` tells us `+1`)


# Why the 0b in bit encoding?
- `0b` is a prefix that tells the computer "there are binary digits coming next"


# Why does 0b01 equal -1?
- Not an outside of Poppins rule, just a choice, the on and off bits can represent whatever values we want them to be


# A weight of +1 means?
- This input matters a lot


# A weight of 0 means?
- Ignore this input entirely


# A weight of -1 means?
- This input matters, but in the OPPOSITE direction


# What is machine learning?
- Machine learning is a field w/in Ai where computer systems perform tasks based on what its lerned from data rather than based on explicit programming to perform the task
- Example:
    - With traditional programming a human writes a function to identify cats
    - With machine learning a model attempts to identify a cat, the model is told how right or wrong it's prediction is, the model adjusts weights & biases based on correct proximity & then repeats till it's good at identifying cats 


# What is gradient descent?
- Gradient descent is the process of optimizing weights
- With machine learning, at the begining of training weights are random numbers
- Then a prediction is made
- Then we compute the error
- Then we adjust the weights to reduce error
- How much we adjust the weights is based on the learning rate


# What is a gradient?
- The gradient tells us the direction and magnitude to change the raw weight
- A positive gradient means increase the weight & a negative gradient means decrease the weight


# What is a learning rate?
- The learning rate is a small number (ex: 0.001) that controls how much we trust the gradient
- If the learning rate is too large, weights jump around and never settle (divergence)
- If the learning rate is too small, training takes forever


# What is a neuron?
- A neuron has 1 weight vector and 1 bias number
- A neurons accepts an input vector and provide an output number
- Calculation:
    ```txt
    output = activation((weight₁ * input₁) + (weight₂ * input₂) + ...  + (weightₙ * inputₙ) + bias)
    ```
- A neuron's weight vector size = number of inputs coming into it
- A neuron provides an `output` number
- The layer the neuron is w/in provides an `output` vector

What Stops Every Neuron from Doing the Same Job?
The short answer: Nothing stops them initially, but competition and redundancy shape the final distribution.
Neurons start with random weights. They begin slightly different from each other:
They don't all start sensitive to the same things.

After training, Neuron A's weights have become such that:

When the input is a number token (embedding looks like [0.8, 0.1, -0.2, 0.5]), output is high positive

When the input is a non-number token (embedding looks different), output is low or negative

Neuron A has learned to be a "number detector" because its weights create a high dot product with number embeddings.

The neuron's output (a single number) indicates number-ness

The embedding vector as a whole, when multiplied by the neuron's weights, produces a high output for numbers

Early layers learn simple features because they only see the raw input. Later layers learn complex features because they see the simple features already extracted.

Layer 1 receives token embeddings. These are just numbers. They don't say "this is a number" or "this is a verb." They are raw, unprocessed vectors. Layer 1's job: find patterns directly in these raw numbers.

Layer 2 doesn't see the raw embedding. It sees what Layer 1 produced: Now Layer 2 is working with already-extracted features like "number-ness" (0.9), "verb-ness" (0.1), etc.

Layer 1 input: [0.8, 0.1, -0.2, 0.5]  // raw embedding for token "5"
Layer 2 input: [0.9, 0.1, 0.2, 0.0]  // Layer 1's output for token "5"

Layer 1: Simple Features
- Nueron 1: "is this a number?"
- Nueron 2: "is this a noun?"
- Nueron 3: "is this a verb?"
- Nueron 4: "is this punctuation?"
Now Layer 2 sees these outputs. It can learn combinations: (ex: "number followed by noun?")
Complexity = Composition of Simplicity
Layer N can only learn features that are combinations of what Layer N-1 learned.
Layer neurons learn through It learns through gradient descent:
Start with random weights in all layers

The loss signal pushes early layers to learn useful simple features (because they help later layers)

Later layers learn to combine those simple features into complex ones
Gradient descent naturally pushes simple features earlier, complex features later


# Why Simple Features Must Be in Early Layers
- You cannot build complex features without simple features as building blocks.




Higher loss = worse prediction

The Backward Pass (This is where amplification happens)
The code computes: "How much did each weight contribute to this loss?"




# What is a layer?
- Collection of neurons that process data simultaneously
- Receive input from the same previous layer

Have independent weight vectors

Produce outputs that go to the same next layer


# What is the Embedding Table?
- Vocabulary lookup table
- Map's token IDs w/ embedding vectors
- Also called an Embedding Layer
- The embedding step (lookup vectors for each token in the sequence) is the first step w/in the embedding table in the forward pass


# What is the Attention layer?
- 


# What is the FFN layer?
- 


# What is FFN?
- FFN stands for Feed-Forward Network


# HERE

# What is deep learning?
- Deep learning is a machine learning architecture w/ many layers (3 to hundreds)
- Each layer transforms the data
- Each layer learns different patterns
- Each layer builds on the previous layer's representations


# What is an activation function?
- An activation function transforms a `logit` into an `output`
- Activation is applied per neuron
- Each neuron's `logit` pass through the activation function independently


# What is a logit?
- Aka a `weighted sum` or `score`
- When predicting the next token, the model computes logits for all tokens w/in its vocabulary
- Larger logits mean the model thinks that token is more likely
- Logits aren't probabilities (they don't sum to 1 & they can be negative)
- A logit is a single number that a neuron produces before activation
- The logit tells you how confident the neuron is (ex: cat or not cat)
    - Large and positive → confident it IS a cat
    - Large and negative → confident it IS NOT a cat
    - Near zero → unsure
- Calculation:
    - `dot product + bias`
    - `logit = (weight₁ * input₁) + (weight₂ * input₂) + ...  + (weightₙ * inputₙ) + bias`


# Why is an activation function essential?
- An activation function is essential b/c multiple linear layers collapse into one
- Example w/o activation:
    ```txt
    // layer 1 w/o activation
    logit₁ = (weight₁ * input₁) + bias₁

    // layer 2 w/o activation
    logit₂ = (weight₂ * logit₁) + bias₂

    // place the calculation from layer 1 into layer 2
    logit₂ = (weight₂ * ((weight₁ * input₁) + bias₁)) + bias₂

    // rewrite layer 2 as weight₂ multiplying into each additive property (distribution rule in mathematics)
    logit₂ = (weight₂ * (weight₁ * input₁)) + (weight₂ * bias₁) + bias₂

    // In mathematics, any number of nested linear function can be simplified into a single linear function
    // A single linear function stops layers from being distinct
    logit₂
    ```
- Example w/ an activation function (σ):
    ```txt
    // layer 1 w/ activation
    logit₁ = σ((weight₁ * input₁) + bias₁)

    // layer 2 w/ activation
    logit₂ = σ((weight₂ * logit₁) + bias₂)

    // place the calculation from layer 1 into layer 2
    logit₂ = σ((weight₂ * σ((weight₁ * input₁) + bias₁)) + bias₂)

    // distributive property is not allowed here (ex: weight₂ * bias₁)
    // weight₂ can't distribute into the (weight₁ * input₁) or bias₁ b/c they are w/in a function
    // An activation function is a non linear operation (it's not just simple addition & multiplication)
    // An activation function breaks the distributive property & helps layers be distinct
    ```


# What is dropout?
- Dropout randomly zeros neurons, prevents overfitting & forces robust, redundant representations
- Dropout lets us randomly set a fraction (`dropout_rate`) of neuron outputs to zero during training
- Neural networks can memorize training data instead of learning general patterns, which leads to individual neurons become too specialized but dropout forces the network to be redundant, ensuring no single neuron can becomes essential to a quality output
- No dropout required w/ Ternary Models b/c they inherently have less capacity to overfit, already have a strong form of regularization so the quantization itself prevents the model from learning overly specific patterns


# What is normalization?
- Normalization is a function that shifts and scales a set of values to have stable statistics
- Normalization adjusts all neuron outputs using global statistics
- Normalization stabilizes training, enables higher learning rates & prevents gradient issues


# What is Overfitting?
- Overfitting is when a model learns the training data perfectly but fails to generalize to new, unseen data
- A model that overfits:
    - Performs perfectly on training data
    - Fails on real-world, unseen data
    - Has memorized rather than learned


# How to prevent overfitting?
- More data: Harder to memorize, must generalize
- Dropout: Forces redundant representations
- Regularization: Penalizes large weights
- Early stopping: Stop before validation loss rises
- Data augmentation: Creates more varied examples


# What does linearity mean?
- Linearity means, if we double the input, the output doubles
- The relationship is a straight line (or hyperplane in higher dimensions)


# What is a linear function?
- A linear functions output changes proportionally to its input
- Example: Pre activation / Logit Calculation: `logit = (weight₁ * input₁) + (weight₂ * input₂) + ...  + (weightₙ * inputₙ) + bias`


# What is a non-linear function?
- A non-linear functions output does NOT change proportionally to its input
- The relationship between the input & output is a curve, not a straight line.
- Example: Pre activation / Logit Calculation: `logit = (weight₁ * input₁) + (weight₂ * input₂) + ...  + (weightₙ * inputₙ) + bias`


# What is inclusive OR?
- Inclusive OR includes the both possibility
- Inclusive OR means true if either and true if both
- Example: In order to get a job at XYZ company, experience with C++ or Java is mandatory


# What is exclusive OR?
- Exclusive OR excludes the both possibility
- Exclusive OR means true if either but not true if both
- Example: When you buy a car from XYZ company, you get $2500 cashback or accessories worth $2500.


# What is an output?
- An output is a vector that is provided by a layer after aligning inputs w/ ternary weights
- The number of neurons w/in a layer is equal to the `output_size`
- During attention & ffn compress the `output_size` is equal to the `embedding_dim`
- During ffn expand the `output_size` is equal to the `embedding_dim * 4`


# Got an output calculation example?
```txt
input = [2.0, 1.5, 0.5]

weights = [
    [1, 0, -1],   // neuron 0
    [0, 1, -1],   // neuron 1
    [-1, 1, -1],  // neuron 2
]

output[0] = (2.0×1) + (1.5×0) + (0.5×-1) = 2.0 + 0.0 - 0.5 = 1.5
output[1] = (2.0×0) + (1.5×1) + (0.5×-1) = 0.0 + 1.5 - 0.5 = 1.0
output[2] = (2.0×-1) + (1.5×1) + (0.5×-1) = -2.0 + 1.5 - 0.5 = -1.0

output = [1.5, 1.0, -1.0]  // length 3
```


# What is an input?
- An input is a matrix of numbers (length = embedding dimension, height = token length) that comes from somewhere, that somewhere might be the:
    - Original sequence
    - Output of a previous layer
- A model moves the input through layers to comprehend the input & then predict the next token


# What is a sequence?
- A sequence is an ordered list of tokens


# What is a weighted input?
- A weighted input is the result of multiplying an input by its corresponding weight
- Calculation: `xₙ * wₙ`


# What is a neural network?
- A neural network is a machine learning technique that connects layers of neurons
- Example: The transformer architecture
- A neural network is a mathematical function that transforms an input into an output through a series of calculations
- At the beginning of training a neural networks weights and biases are random & through training these numbers get good enough to produce quality outputs


# What is a model?
- An Ai model is an instance of a neural network that has been trained w/ samples,  has learned weights and biases and can provide quality outputs (next token predictions) when given a prompt


# What is Attention?
- Attention computes, how much each token should pay attention to all other tokens w/in a sequence
- Each token w/in the sequence is given 3 vectors, the query, key and value vectors
- Attention refers to the weights (probabilities) that determine how much information to take from all visible tokens


# What are Attention scores?
- Logits
- Raw dot products (Q·K), gives us a single value score for each token


# What are Attention weights?
- Attention weights are attention scores after softmax
- Attention weights are probabilities that sum to 1
- Attention weights tells us 'this token contributes `attention_weight` percent of its `Value` to the `output` for the current token'


# What is Attention output?
- Attention output is the weighted sum of values using the attention weights


# What is a Transformer?
- A Transformer is an deep learning architecture where each token w/in a sequence is aware of all other tokens w/in the sequence (Attention)


# What is a Query vector?
- A Query vector is given to a token an answers (what is this token looking for in other tokens)
- A Query vector helps us search for related tokens w/in a sequence by comparing the Query vector of the current token w/ the Key vector of other tokens
- During inference we get a query vector for the last token and compare to all other tokens
- During training we get a query vector for all tokens w/in the ai response and compare to all other tokens simultaneously


# What is a Key vector?
- A Key vector contains information about what a token offers to others
- We match the Key vector w/ the Query vector to determine if there is a relationship between 2 tokens


# What is a Value vector?
- A Value vector contains the actual data that will be passed forward if this token is selected


# What is token selection?
- Token selection in attention identifies which past tokens are most relevant to the current token


# What is token prediction?
- Token prediction in attention identifies what token is most likely to come after the current token


# What is training?
- Training is the process of creating a model that makes useful next token predictions
- In the training process we show the model samples and let it learn from its mistakes (adjust its weights and biases)


# What is a hidden state?
- Math anotation is `h`
- The hidden state is the "current understanding" of the input as it flows through the model
- Input tokens start as embeddings (not yet hidden states)
- After passing through the first transformer layer, they become hidden states
- Each layer transforms the hidden state further
- A hidden state is a token vector after it has passed through at least one layers
- Hidden b/c
    - Internal
    - Not directly visible
    - Intermediate representations
- Identifies what the model “knows” about the sequence


# What is the final hidden state?
- The final hidden state (last layer's output) is what gets multiplied by output weights to predict the next token


# What is an Output Projection Vector?
- Annotation: `W_out[i]`
- An Output Projection Vector is a vector of weights of `embedding_dim` length for a token that identifies what hidden state pattern predicts this token
- Each token w/in the model's vocabulary has an Output Projection Vector
- When we multiply the Output Projection Vector with the hidden state, we get a score indicating how well the hidden state matches the token


# What is an Output Projection Matrix?
- Annotation: `W_out`
- Output Projection Matrix is `embedding_dim` length and `vocab_size` height (`[vocab_size, hidden_dim]`)


# What is a Linear Layer Row?
- A Linear Layer Row is a vector of `input_dim` length that identifies what hidden state pattern predicts this neuron
- Each neuron w/in a layer has a Linear Layer Row
- When we multiply the Linear Layer Row with the hidden state, we get a score indicating how well the hidden state matches the neuron


# What is a Linear Layer Matrix?
- A Linear Layer Matrix is `input_dim` length and `output_dim` height (`[output_dim, input_dim]`)


# What is a bias?
- Bias controls the neuron's "baseline" activation when all inputs are zero
- A bias is single number added during the output calculation
- Fixed during training
- The bias is a constant number added after the weighted sum
- Each token has bias and each neuron has a bias
- Tells us How likely a token / neuron is in general
- Small bias -> token / neuron rarely appears
- High bias -> token / neuron appears often in many contexts


# What is the output bias?
- An output bias is computed during training, is a unique value for each token and identifies baseline tendencies for a token
- High bias -> token appears often in many contexts
- Small bias -> token rarely appears


# What is the origin?
- Where x, y & z meet


# What is a basis axis?
- A basis axis is a unique direction from the origin that aligns w/ a dimension
- Unique meaning no 2 basis axis w/in a vector share the same orientation
- Models distribute meaning (nouniness, verbiness, pronouniness, animalness) across dimensions (basis axes)
- What each dimension represents is not human defined, only the number of allowed dimensions (embedding dimension) is human defined


# What is a latent feature?
- A latent feature is a pattern the model discovered during training that:
    - Is not explicitly named
    - We did not manually define
    - Exists only as numbers inside the network
- Each dimension w/in a vector captures a pattern in the data, we do not know what that pattern is and there is no guarantee that it corresponds to a clean human concept (ex: animalness)
- The model discovers useful internal dimensions automatically
- Every dimension in embeddings, hidden states & neuron outputs is a latent feature



# What is orientation?
- Orientation is what way an arrow points from the origin
- Independent of magnitude


# What is softmax?
- Softmax is the process of converting logits to probabilities
- Softmax helps the much bigger score dominate
- `probability = eulers_num^(logit) / sum(eulers_num^(all_logits))`
- Numerator is Euler's number raised to the logit for one token
- Denomenator is the sum of Euler's number raised to the logit for all tokens
- Example:
    ```txt
    Vocabulary is 3 tokens
    Token 0: "apple"
    Token 1: "banana"  
    Token 2: "cherry"

    logits = [2.0, 1.0, 0.1]

    apple_numerator = e^(2.0) = 7.389
    banana_numerator = e^(1.0) = 2.718
    cherry_numerator = e^(0.1) = 1.105

    denominator = sum(e^(all_logits)) = 7.389 + 2.718 + 1.105 = 11.212
    ```


Temperature is applied before softmax. It divides the raw logits by temperature (T):

Raw Logits
Token A: 2.5
Token B: 1.2
Token C: 0.8
Token D: -0.5

Temperature Scaling
Temperature is applied before softmax. It divides the raw logits by temperature (T):
scaled_logit = logit / T

Softmax (Convert to Probabilities)
Token A: 0.65 (65% chance)
Token B: 0.18 (18% chance)
Token C: 0.12 (12% chance)
Token D: 0.05 (5% chance)

- T = 1.0	No change	2.5 → 2.5 → probability 0.65
- T = 0.5	Makes differences more extreme (more confident)	2.5 → 5.0 → probability 0.95
- T = 1.5	Makes differences smaller (more uniform)	2.5 → 1.67 → probability 0.45

The code does this:
We sort tokens from most likely to least likely so the cumulative probability builds from highest to lowest.

Generate a random number between 0 and 1

Figure out which token's range the random number falls into

So if token A is 0-0.65 and the random number is 0.32 then token A is chosen

# Why sort?
- Not sorting works but is more computationally expensive
- Most random numbers (65% of them) will land in Token A's range. The code finds the token quickly because the most likely tokens are checked first.


# What is SwiGLU?
- SwiGLU stands for Swish Gated Linear Unit
- SwiGLU is an activation function used in feed-forward networks


# What is the ReLU?
- ReLU stands for Rectified Linear Unit
- ReLU is an activation function
- IF `logit > 0` THEN `output = logit` ELSE `output = 0`
- `output = ReLU(logit) = max(0, logit)`


# What is the ReLU?
- ReLU stands for Rectified Linear Unit
- ReLU² stands for ReLU squared
- ReLU is an activation function
- Raise the ReLU to the power of 2
- ReLU² creates sparsity (many zeros) in the activations which is critical for ternary quantization


# What is Sigmoid
- Sigmoid is an activation function
- Output is always between 0 and 1
- `output = sigmoid(logit) = 1 / (1 + eulers_num^(-logit))`


# What is sparsity?
- Sparsity means most values w/in a vector are zero
- Dense vector = 0% sparsity = `[1,2,3]`
- Sparse vector = 50% sparsity = `[0,1,0,2]`
- Very sparse vector = 75% sparsity = `[0,0,0,1]`
- When a vector is sparse we can:
    - Store only the non-zero values (less memory)
    - Compute only where values are non-zero (less work)
