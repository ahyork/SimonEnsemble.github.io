---
layout: post
title: Markov Chains
snippet: Markov Chains are powerful estimation tools that give a probability distribution for a next state only based on the current state. They are especially useful for understanding multivariate systems where an analytical solution cannot be reached.
tags: [math, Julia]
author: Arthur York
---

Markov chains are useful probability tools that work by defining real world
phenomenon as a series of states and the transitions between them. Given the
current state only, a Markov chain has enough information to create a probaility
distribution of the states that can immediately follow. To better understand
them, we will work with an imaginary town that experiences four types of
weather: sunny, cloudy, rainy, or snowy. Weird as it may be, the most accurate
forecast for any day is based solely on the previous day's weather. The graph
shown in figure 1 is the model for the weather in this town.

All of the code I wrote for experimenting with Markov chains can be found
[here](https://github.com/ahyork/markov_chains).

{:.centerr}
<figure>
    <img src="/images/markov/markov_model.png" alt="Markov chain model for weather">
    <figcaption>Fig 1. A Markov chain model for the weather in our fictional town.</figcaption>
</figure>

## The Markovian Property

In order for a model to accurately depict a Markov chain, that model must follow
the Markovian property. This property requires that the current state holds all
of the information about the previously visited states. This is because a Markov
chain is memoryless and only "remembers" the current state; as soon as the model
transitions to a new state it forgets where it has been. This means that a
Markov chain must be designed in such a way that every state holds all of the
information it needs to predict the next state.

The model shown in figure 1 for our weather example has the Markovian property
because each state holds enough information to predict the next state. The only
relevant information for determing the next day's weather is the current day's
weather, so we can set each state to be a different type of weather. Then each
state holds enough information to predict the next state.

If the weather in this town is actually dependent on the previous week of
weather, the above model would no longer hold the Markovian property. Our model
only keeps information from the previous day and we would need the previous
seven. Therefore, the current state does not contain the necessary information to
predict the next state and fails to satisfy the Markovian property. We can
modify our system to track all of these transitioning states, but that would
quickly become unwieldy.

## Implementing a Markov Chain

We have established that our Markov chain model in figure 1 holds the Markovian
property, so we can move ahead and implement it. 

Our model is a weighted directed graph so I will be using an adjacency matrix to
implement it in julia. The following code snippet is a 4x4 adjacency matrix that
represents the graph from figure 1. The rows and columns correspond to different
states: 1 is sunny, 2 is cloudy, 3 is rainy, and 4 is snowy. Each column holds
the probability density of which state the model will transition to, where the
rows represent the possible states. The sum across each column is equal to 1
because at any given state it MUST transition to another state (this can include
transitioning back to itself).

I chose to make the columns sum to one because julia is a column major language.
This matrix can also be written where the rows sum to one, it just needs to be
consitent throughout the implementation.

{% highlight julia %}
const markov_model = [0.5 0.3 0.1 0.0;
                      0.4 0.2 0.3 0.2;
                      0.1 0.4 0.4 0.7;
                      0.0 0.1 0.2 0.1]
{% endhighlight %}

We can now use this matrix to calculate probability distributions. Let's say
that today (the current state) is cloudy (column 3), and we want to create a
forecast for tomorrow's weather. We can look down the third column in the matrix
to find the probabilities of seeing each type of weather tomorrow. The model
shows there is a 10% chance of it being sunny, 30% of it staying cloudy, 40%
chance it will rain, and a 20% chance it will snow. 

## Markov Chain Monte Carlo Algorithms

Suppose now that our fictional town is competing for the title of rainiest town
in the United States, but since they only remember the previous day's weather
they haven't kept any records! They want to find the average number of days each
year that their town experiences rain but their model doesn't show that. They
can use a Markov chain Monte Carlo simulation to calculate this average. 

The Markov chain Monte Carlo simulation will use the probability distribution
for the current state to randomly select the next state. The potential states
will occupy ranges of values from 0 to 1, and a random number between 0 and 1
will be generated. The range this value falls into will show the next state. The
following code is used to transition between states. This process can be
continued as many times as we specify.

{% highlight julia %}
unction move(state::Int)

    new_state = 1
    probability = markov_model[new_state, state]
    move_choice = rand()
    # iterates through probabilities to find the "pot" that was selected
    while move_choice > probability && probability < 1.0
        new_state += 1
        probability += markov_model[new_state, state]
    end

    return new_state
end
{% endhighlight %}

For the purposes of finding the average number of days in a year, we can run a
simulation with 10,000 cycles and then find the probability density function for
the weather.

{% highlight julia %}
# Seeded Random to get same results
Random.seed!(1234)
# runs a Markov chain Monte Carlo simulaiton with 10,000 cycles
progression = mcmc_weather(10000)
prob_density = calculate_probability_density(progression)

# Probability of sun: 0.230700
#   Days of sun in a year: 84
# Probability of clouds: 0.287100
#   Days of clouds in a year: 105
# Probability of rain: 0.369800
#   Days of rain in a year: 135
# Probability of snow: 0.112400
#   Days of snow in a year: 41
{% endhighlight %}

With the data collected from a 10,000 cycle simulation there are an average of
135 days per year that our imaginary town experiences rain. This number is an
estimate, and will become more accurate the more samples we take.

### Using Eigenvectors to find the steady state of a Markov chain model

Because we have an adjacency matrix representing our model, we can use the
eigenvectors to analytically solve for the steady state. I found a great
[blog post](https://medium.com/@andrew.chamberlain/using-eigenvectors-to-find-steady-state-population-flows-cd938f124764)
that does a great job of explaining this method.

After going through this, I wrote up some code to run this calculation for
myself and verify that my Monte Carlo simulation was working correctly. As you
can see below, the Monte Carlo method did a great job of finding the steady
state of our system!

{% highlight julia %}

S = eigvecs(markov_model)
S⁻¹ = inv(S)
Λ = diagm(0 => eigvals(markov_model))

u₀ = [ 1, 0, 0, 0 ] # initial state is sunny
c = S⁻¹ * u₀ # create this term to remove an S⁻¹ from the equation

steady_state = S[:,1] * c[1]

# Probability of sun: 0.242921
# Probability of clouds: 0.284650
# Probability of rain: 0.360656
# Probability of snow: 0.111773

{% endhighlight %}

## Markov Chain Monte Carlo vs. Metropolis Hastings 

We are able to use a Markov chain Monte Carlo simulation in this instance
because we know the exact probabilities of tranisitioning to different states.
If we instead studied the temperature in our town and only knew that the
probability of tomorrow's weather (the next state) was proportional to the
current temperature we wouldn't be able to construct a probability distribution 
based on today's temperature (the current state). Instead we could use a
Metropolis Hastings algorithm. 

Metropolis Hastings algorithms still work as ways of stepping through Markov
chain models, but are slightly different than the Monte Carlo method previously
discussed. Because we cannot construct a probability distribution from a given
state, we cannot generate a random number to determine the next state. Instead
we will propose a new state, and compare their probabilities. In the temperature
example we would compare the temperature of the current state and the
temperature of the proposed state. We will limit this ratio to only exist
bewteen 0.0 and 1.0, so we can then generate a random number in that range to
determine if the transition is accepted. 

This ratio makes intuitive sense because if the proportional probability of the
suggested move is much greater than the current state (the ratio is $\geq$ 1) it
will almost always accept the transition. However if the probability of the
current state is much greater (the ratio is very close to 0), then it is very
unlikely that it will transition.
