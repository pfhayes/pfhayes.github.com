---
layout: post
title: "Dollar Cost Averaging"
category: "Scala"
tags: ["scala", "finance", "math"]
---
{% include JB/setup %}

Is it possible to take advantage of random fluctuations in a stock price to improve your investment's return?

Dollar cost averaging is a strategy that attempts to do just that.
Instead of investing a lump sum of money into a security, we stagger that investment over multiple periods by investing an equal dollar amount every period.
So, if we have $1000 to invest, we might instead opt to invest $100 every month for 10 months.

The premise of this strategy is that we will purchase more shares when the price is low, and fewer when the price is high.
But does this really yield higher returns?
Let's find out!

### Simulation

If we know the prices of a security at each investment opportunity, then we can calculate what our return would be if we invested as a lump sum, or using dollar cost averaging.
Here's some sample scala code:

{% highlight scala %}
def dollarCostValue(initialCapital: Double, prices: List[Double]): Double = {
  val closingPrice = prices.last
  val dailySpend = initialCapital / prices.length
  val sharesOwned = prices.map(dailySpend / _).sum
  sharesOwned * closingPrice
}

def lumpSumValue(initialCapital: Double, prices: List[Double]): Double = {
  val startingPrice = prices.head
  val closingPrice = prices.last
  val sharesOwned = initialCapital / startingPrice
  sharesOwned * closingPrice
}
{% endhighlight %}

It's clear that the performance of each strategy depends on the prices.
But if we knew ahead of time what the price would be on each day, then we could pick the best day to invest all of our money.
So what's the best strategy when the price fluctuates each day in a manner that can't be predicted?

### Some Simple Models

We can use a random walk to give a reasonable sample of prices.
There are multiple methods of randomly producing stock prices (some better than others), but we can see how the two strategies perform on a couple different walks.

{% highlight scala %}
val random = new Random()

// Price either goes up or down a fixed amount each day
def simpleWalkPrices(initialPrice: Double, deviance: Double): Iterator[Double] = {
  Iterator.iterate(initialPrice)(price =>
    if (random.nextBoolean()) {
      price + (deviance * initialPrice)
    } else {
      price - (deviance * initialPrice)
    })
}

// Price goes up or down a random amount each day
def randomWalkPrices(initialPrice: Double, deviance: Double): Iterator[Double] = {
  val range = (initialPrice * deviance)
  Iterator.iterate(initialPrice)(price => {
    val change = random.nextDouble() * 2 * range - range
    price + change
  })
}

// Price goes up or down a random percent each day
def percentChangePrices(startingPrice: Double, deviance: Double): Iterator[Double] = {
  val range = deviance
  Iterator.iterate(initialPrice)(price => {
    val change = random.nextDouble() * 2 * range - range
    price * (1 + change)
  })
}
{% endhighlight %}

Now, we can compare the expected value of investing with either method on any of these streams of prices.
The specific numeric constants were chosen were chosen fairly arbitrarily, but we get the same results with just about any selected values.

{% highlight scala %}
val initialPrice = 100.0
val initialCapital = 1000.0
val deviance = 0.03
val days = 100
val numTrials = 10000

def simulate(
  strategy: (Double, List[Double]) => Double,
  generator: (Double, Double) => Iterator[Double]
): Double = {
  val trials = for {
    t <- (1 to numTrials)
  } yield {
    val prices = generator(initialPrice, deviance).take(days).toList
    strategy(initialCapital, prices)
  }
  trials.sum / numTrials
}
{% endhighlight %}

Let's look at the results!

{% highlight scala %}
def compare(generator: (Double, Double) => Iterator[Double]): Double = {
  simulate(lumpSumValue, generator) / simulate(dollarCostValue, generator)
}

println("Simulated ratio for simpleWalk: " + compare(simpleWalkPrices))
println("Simulated ratio for randomWalk: " + compare(randomWalkPrices))
println("Simulated ratio for percentChange: " + compare(percentChangePrices))

// Simulated ratio for simpleWalk: 0.9962410919205108
// Simulated ratio for randomWalk: 1.0044757886081357
// Simulated ratio for percentChange: 0.9989441478975456
{% endhighlight %}

From this simulation, it looks like dollar cost averaging does not outperform a lump sum investment.

### An Alternate Model

The above models don't necessarily capture all the properties we might expect from a fluctuating stock price.
Another common method of simulating stock prices is with a [log-normal distribution][1].
A log-normal distribution has the property that the price is as likely to double as it is to halve (and as likely to triple as it is to be thirded, etc.).

[1]: http://en.wikipedia.org/wiki/Log-normal_distribution

{% highlight scala %}
// Price is multipled by either N or 1/N
def ratioChangePrices(startingPrice: Double, deviance: Double): Iterator[Double] = {
  Iterator.iterate(initialPrice)(price => {
    if (random.nextBoolean()) {
      price * (1 + deviance)
    } else {
      price / (1 + deviance)
    }
  })
}

// Log-normal distribution
def logNormalPrices(startingPrice: Double, deviance: Double): Iterator[Double] = {
  Iterator.iterate(initialPrice)(price => {
    price * (math.exp(random.nextGaussian() * deviance))
  })
}
{% endhighlight %}

Unlike the previous models, these models have a positive expected drift over time.
If a stock costs $20 on the first day, and is equally likely to cost $10 or $40 the next day, then the expected value on the second day is $25.

{% highlight scala %}
println("Simulated ratio for ratioChange: " + compare(ratioChangePrices))
println("Simulated ratio for logNormal: " + compare(logNormalPrices))

// Simulated ratio for ratioChange: 1.028479119173095
// Simulated ratio for logNormal: 1.0233932140835074
{% endhighlight %}

We can see that here, lump sum investing beats dollar cost averaging!

### But Why?

Whenever the security has a positive trend, dollar cost averaging will do worse than the lump sum investment.
This is because money invested in later periods spends less time invested, and doesn't get as much time to take advantage of the likely increase in value.

When price fluctuations are truly random, each day's transition is independent of any previous days.
While we might think that we are buying "more shares when the price is low", there are no guarantees the price will
"bounce back up" afterwards.

If you have reason to believe that, on average, the market increases in value, then dollar cost averaging is an inferior strategy.

### Conclusions

Dollar cost averaging is a fine investment strategy for at least one reason.
For most of us, the strategy lines up well with how we get paid (on a monthly or weekly schedule).
Investing some part of every paycheque is a great way to make sure we are saving money for when we need it.
But there's no reason to believe that this strategy will help you take advantage of market fluctuations in any meaningful way.
Better luck next time!

