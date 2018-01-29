[**原文链接**](https://towardsdatascience.com/a-zero-math-introduction-to-markov-chain-monte-carlo-methods-dcba889e0c50) | [**编辑该文档**](https://github.com/DAGfans/TranStudy/new/master/Blogs/a-zero-math-introduction-to-markov-chain-monte-carlo-methods.md) | [**原始译文**](http://www.iotachina.com/lingshuxuejichulijiemcmcmengtekaluomaerkefulian.html) | [**译文作者**](http://www.iotachina.com/author/tigermumu)

* * *

# A Zero-Math Introduction to Markov Chain Monte Carlo Methods
``
零数学基础理解MCMC（蒙特卡洛马尔科夫链） 
``
* * *
For many of us, Bayesian statistics is voodoo magic at best, or completely subjective nonsense at worst. Among the trademarks of the Bayesian approach, Markov chain Monte Carlo methods are especially mysterious. They’re math-heavy and computationally expensive procedures for sure, but the basic reasoning behind them, like so much else in data science, can be made intuitive. That is my goal here.

``
对于大多数人来说，贝叶斯统计方法简直就像巫语一样难懂或者是彻底的胡言论语。在众多贝叶斯实现方法中，马尔可夫链蒙特卡洛（MCMC）方法由于包含大量数学计算，显得尤其神秘。但他们背后的基本原理，就像数据科学中的其他东西一样，可以直观理解，也就是我的目标。
``


So, what are Markov chain Monte Carlo (MCMC) methods? The short answer is:
``
那么，什么是马尔可夫链蒙特卡罗（MCMC）方法？ 简单说就是：
``
> MCMC methods are used to approximate the posterior distribution of a parameter of interest by random sampling in a probabilistic space.
> 
> MCMC方法用于通过在概率空间中随机采样来近似兴趣参数的后验分布。

In this article, I will explain that short answer, without any math.

First, some terminology. A **_parameter of interest_** is just some number that summarizes a phenomenon we’re interested in. In general we use statistics to estimate parameters. For example, if we want to learn about the height of human adults, our parameter of interest might be average height in in inches. A **_distribution_** is a mathematical representation of every possible value of our parameter and how likely we are to observe each one. The most famous example is a bell curve:

``在本文中，我将用非数学知识来简短回答。``

``
首先，介绍一些术语。兴趣参数（parameter of interest）是我们感兴趣现象的一些数字。通常使用统计学评估参数。例如，如果我们想了解人类成年人的身高，我们的兴趣参数可能是精确到以英寸为单位的平均身高。分布是参数的每个可能值的数学表示，以及，我们有多大可能观察每个参数。 最着名的例子是钟形曲线：
``

![](https://cdn-images-1.medium.com/max/1600/0*ZnhfX8oUe0biQRR6.)


Courtesy【引用】 [M. W. Toews](https://commons.wikimedia.org/wiki/File:Standard_deviation_diagram.svg)

In the **_Bayesian_** way of doing statistics, distributions have an additional interpretation. Instead of just representing the values of a parameter and how likely each one is to be the true value, a Bayesian thinks of a distribution as describing our _beliefs_ about a parameter. Therefore, the bell curve above shows we’re pretty sure the value of the parameter is quite near zero, but we think there’s an equal likelihood of the true value being above or below that value, up to a point.

As it happens, human heights do follow a normal curve, so let’s say we believe the true value of average human height follows a bell curve like this:

``
在贝叶斯统计方式中，分布有另外一种解释。 贝叶斯不是仅仅代表参数的值以及每个参数为真的可能性有多高，贝叶斯认为分布描述了我们对参数的信念。因此，上面的钟形曲线表明我们非常确定参数的值接近于零，但是我们认为真实值高于或低于该值的可能性是相等的。
``

``
事实上，人的身高确实遵循正态分布曲线，所以我们假设我们认为的平均身高的真实值遵循如下的钟形曲线：
``

![](https://cdn-images-1.medium.com/max/1600/0*tlqbgInNAYyi3dgx.)


Clearly, the person with beliefs represented by this graph has been living among giants for years, because as far as they know, the most likely average adult height is 6'2" (but they’re not super confident one way or another).

Lets imagine this person went and collected some data, and they observed a range of people between 5' and 6'. We can represent that data below, along with another normal curve that shows which values of average human height _best explain the data:_

``
显然，这个图表所代表的信仰者已经生活在巨人之间多年，因为据此图所知，最有可能的平均成年身高是6’2“（但他们并不是超级自信）。
``

``
让我们假设这个信仰者收集了一些数据，他们观察到了一些身高在5”到6“之间的人。 我们可以用下图来表示这些数据，表明哪个身高的平均值最能解释这些数据：
``

![](https://cdn-images-1.medium.com/max/1600/0*kkaO7QpZeGOg9DRf.)

In Bayesian statistics, the distribution representing our beliefs about a parameter is called the **_prior distribution_**_,_ because it captures our beliefs _prior_ to seeing any data. The **_likelihood distribution_** summarizes what the observed data are telling us, by representing a range of parameter values accompanied by the likelihood that each each parameter explains the data we are observing. Estimating the parameter value that maximizes the likelihood distribution is just answering the question: what parameter value would make it most likely to observe the data we have observed? In the absence of prior beliefs, we might stop there.

``
在贝叶斯统计中，代表我们关于参数的信念的分布被称为先验分布，因为它是在看到任何数据之前反映出的我们的信念。可能性分布概括了观察到的数据以及对应的参数值所观察数据的可能性。估算最大可能性分布的参数值只是回答了一个问题：什么参数值会使它最有可能反映出到我们观察到的数据？如果没有先验分布，我们可能就此止步。
``


The key to Bayesian analysis, however, is to combine the prior and the likelihood distributions to determine the **_posterior distribution_**. This tells us which parameter values maximize the chance of observing the particular data that we did, taking into account our prior beliefs. In our case, the posterior distribution looks like this:

``
然而，贝叶斯分析的关键是将先验分布和可能性分布结合起来确定后验分布。这告诉我们，考虑到我们先验分布状态，哪个参数值最大限度地反应出我们所做的特定数据。在我们的例子中，后验分布是这样的
``


![](https://cdn-images-1.medium.com/max/1600/0*3M3HCM8goJjlS-KX.)


Above, the red line represents the posterior distribution. You can think of it as a kind of average of the prior and the likelihood distributions. Since the prior distribution is shorter and more spread out, it represents a set of belief that is ‘less sure’ about the true value of average human height. Meanwhile, the likelihood summarizes the data within a relatively narrow range, so it represents a ‘more sure’ guess about the true parameter value.

``
上面的红线代表后验分布。你可以把它看成是先验分布和可能性分布的平均值。由于先验分布数据少，分布更广，它代表一组”不确定“的平均身高真值的信念。与此同时，可能性分布会在一个相对较小的范围内对数据进行总结，因此它代表了对真值的“更确定”的猜测。
``


When the prior the likelihood are combined, the data (represented by the likelihood) dominate the weak prior beliefs of the hypothetical individual who had grown up among giants. Although that individual still believes the average human height is slightly higher than just what the data is telling him, he is mostly convinced by the data.

``
当先验分布和可能性分布被合并时，数据(由可能性分布表示)支配了假设存在于巨人中成长起来的个体微弱的先验信念。尽管该个体仍然相信人类的平均身高略高于数据值，但他可能会被数据说服。
``


In the case of two bell curves, solving for the posterior distribution is very easy. There is a simple equation for combining the two. But what if our prior and likelihood distributions weren’t so well-behaved? Sometimes it is most accurate to model our data or our prior beliefs using distributions which don’t have convenient shapes. What if our likelihood were best represented by a distribution with two peaks, and for some reason we wanted to account for some really wacky prior distribution? I’ve visualized that scenario below, by hand drawing an ugly prior distribution:

``
在两个钟形曲线的情况下，求解后验分布非常简单。有一个简单的方程式把两者结合起来。但如果我们的先验分布和可能性分布表现不好呢？有时候，用那些非简化形状的分布来建模我们的数据或先验信念是最准确的。如果可能性分布最好由两个峰值的分布来表示时呢？由于某种原因，我们想要解释一些非常古怪的先验分布时呢？我已经想象了下面的场景，手工绘制了一个丑陋的先验分布:
``


![](https://cdn-images-1.medium.com/max/1600/0*PcWai087HhpJtbkm.)

>Visualizations rendered in Matplotlib, enhanced using MS Paint<br>在Matplotlib中呈现的可视化，使用MS Paint进行了增强


As before, there exists _some_ posterior distribution that gives the likelihood for each parameter value. But its a little hard to see what it might look like, and it is impossible to solve for analytically. Enter MCMC methods.

``
如前所述，存在一些后验分布，为每个参数值提供可能性分布。但是要想知道它看起来是什么样子有点难，而且不可能解析。这时MCMC该上场了。
``

MCMC methods allow us to estimate the shape of a posterior distribution in case we can’t compute it directly. Recall that MCMC stands for Markov chain Monte Carlo methods. To understand how they work, I’m going to introduce Monte Carlo simulations first, then discuss Markov chains.

``
MCMC方法允许我们直接评估后验分布的形状，而无需进行计算。为理解它工作原理，我首先介绍蒙特卡洛模拟，然后讨论马尔可夫链。
``

* * *

Monte Carlo simulations are just a way of estimating a fixed parameter by repeatedly generating random numbers. By taking the random numbers generated and doing some computation on them, Monte Carlo simulations provide an approximation of a parameter where calculating it directly is impossible or prohibitively expensive.

``
蒙特卡罗模拟只是一种通过重复生成随机数来估计一个固定参数的方法。通过生成随机数并对其进行计算，蒙特卡罗模拟提供了一个近似的参数，直接计算它是不可能或者计算量巨大。
``

Suppose that we’d like to estimate the area of the follow circle:

``假设我们想评估下圆的面积:``

![](https://cdn-images-1.medium.com/max/1600/0*OVzsV_Mw2OGi1mQ4.)


Since the circle is inside a square with 10 inch sides, the area can be easily calculated as 78.5 square inches. Instead, however, we can drop 20 points randomly inside the square. Then we count the proportion of points that fell within the circle, and multiply that by the area of the square. That number is a pretty good approximation of the area of the circle.

``
由于圆形所处的正方形边长为10英寸，所以面积可以很容易计算出78.5平方英寸。然而，我们可以在正方形内随机投掷20个点。然后我们计算在圆内的点的比例，然后乘以正方形的面积。这个数字将非常近似于圆的面积。
``

![](https://cdn-images-1.medium.com/max/1600/0*OuLZBu6HPdhignVB.)


Since 15 of the 20 points lay inside the circle, it looks like the circle is approximately 75 square inches. Not too bad for a Monte Carlo simulation with only 20 random points.

``
因为20个点中的15个都在圆圈内，所以看起来这个圆大约是75平方英寸。对于一个只有20个随机点的蒙特卡罗模拟来说，结果不算太坏。
``

Now, imagine we’d like to calculate the area of the shape plotted by the [Batman Equation](http://mathworld.wolfram.com/BatmanCurve.html):

``
现在，想象一下我们想要计算蝙蝠侠方程绘制的形状的面积
``

![](https://cdn-images-1.medium.com/max/1600/0*5ahw9HtOjbMK0tmO.)

Here’s a shape we never learned an equation for! Therefore, finding the area of the bat signal is very hard. Nevertheless, by dropping points randomly inside a rectangle containing the shape, Monte Carlo simulations can provide an approximation of the area quite easily!

``
我们从未学过任何方法来求解这样形状的面积！但是，通过在包含随机形状的矩形内随机放置点，蒙特卡罗模拟可以很容易地提供该区域的近似值！
``

Monte Carlo simulations aren’t only used for estimating the area of difficult shapes. By generating a lot of random numbers, they can be used to model very complicated processes. In practice, they’re used to forecast the weather, or estimate the probability of winning an election.

``
蒙特卡罗模拟不仅用于估计困难形状的面积。通过生成大量随机数，它还可以用对非常复杂的过程进行建模。在实践中，蒙特卡洛模拟被用来预测天气，或者估算赢得选举的可能性。
``

* * *

The second element to understanding MCMC methods are Markov chains. These are simply sequences of events that are probabilistically related to one another. Each event comes from a set of outcomes, and each outcome determines which outcome occurs next, according to a fixed set of probabilities.

``
理解MCMC方法的第二个元素是马尔科夫链链。它是由简单的存在概率关联的事件序列构成。每个事件都来自一组结果，根据一组固定的概率，每一个结果决定了接下来发生的结果。
``

An important feature of Markov chains is that they are **_memoryless_**: everything that you would possibly need to predict the next event is available in the current state, and no new information comes from knowing the history of events. A game like [Chutes and Ladders](http://jakevdp.github.io/blog/2017/12/18/simulating-chutes-and-ladders/) exhibits this memorylessness, or **_Markov Property,_** but few things in the real world actually work this way. Nevertheless, Markov chains are powerful ways of understanding the world.

``
马尔科夫链的一个重要特征是它的无记忆性：可能需要预测下一个事件的所有因素在当前状态下是可用的，并且从事件历史中没有得到新的信息。像Chutes和Ladders这样的游戏展示了这种无记忆性，或者称为马尔科夫的特性，但现实世界中很少有东西是这样运作的。然而，即便如此，马尔可夫链仍然是理解世界的强大方式。
``

In the 19th century, the bell curve was observed as a common pattern in nature. (We’ve noted, for example, that human heights follow a bell curve.) Galton Boards, which simulate the average values of repeated random events by dropping marbles through a board fitted with pegs, reproduce the normal curve in their distribution of marbles:

``在19世纪，钟形曲线被认为是自然界的一种常见模式。(例如，我们注意到，人类身高遵循钟形曲线)，Galton board，通过将弹珠放入装有钉子的木板上，模拟重复随机事件的平均值，在弹珠分布上呈现出了钟形曲线:``

![](https://cdn-images-1.medium.com/max/1600/0*HDeFoQPFGs9ueUI0.)


Pavel Nekrasov, a Russian mathematician and theologian, [argued](https://www.americanscientist.org/article/first-links-in-the-markov-chain) that the bell curve and, more generally, the law of large numbers, were simply artifacts of children’s games and trivial puzzles, where every event was completely independent. He thought that interdependent events in the real world, such as human actions, did not conform to nice mathematical patterns or distributions.

``
俄罗斯数学家和神学家Pavel Nekrasov认为，钟形曲线，或者一般称为，大数定律，只不过是儿童游戏的产物，每个事件都是完全独立的。他认为，现实世界中的事件都是相互依赖的，如人类行为，并不符合良好的数学模型或分布。
``

Andrey Markov, for whom Markov chains are named, sought to prove that non-independent events may also conform to patterns. One of his best known examples required counting thousands of two-character pairs from a work of Russian poetry. Using those pairs, he computed the conditional probability of each character. That is, given a certain preceding letter or white space, there was a certain chance that the next letter would be an A, or a T, or a whitespace. Using those probabilities, Markov was ability to simulate an arbitrarily long sequence of characters. This was a Markov chain. Although the first few characters are largely determined by the choice of starting character, Markov showed that in the long run, the distribution of characters settled into a pattern. Thus, even interdependent events, if they are subject to fixed probabilities, conform to an average.

``
Andrey Markov，试图证明非独立事件也可能遵循某种模式。他最著名的例子之一就是要数出俄罗斯诗歌作品中成千上万的两对字符。使用这些两对字符，他计算了每个字符的条件概率。也就是说，给定前一个字母或空格，下一个字母可能是a, T，或空格。利用这些概率，Markov能够模拟任意长的字符序列。这就是马尔可夫链。虽然最初的几个字符很大程度上是由初始字符的选择而决定的，但Markov指出，从长远来看，字符的分布形成了一个特定模式。因此，即使是相互依赖的事件，如果它们服从于固定的概率，也要遵循平均值。
``

For a more useful example, imagine you live in a house with five rooms. You have a bedroom, bathroom, living room, dining room, and kitchen. Lets collect some data, assuming that what room you are in at any given point in time is all we need to say what room you are likely to enter next. For instance, if you are in the kitchen, you have a 30% chance to stay in the kitchen, a 30% chance to go into the dining room, a 20% chance to go into the living room, a 10% chance to go into the bathroom, and a 10% chance to go into the bedroom. Using a set of probabilities for each room, we can construct a chain of predictions of which rooms you are likely to occupy next.

``
举一个更有用的例子，想象你住在一个有五个房间的房子里。你有一间卧室、一间浴室、一间客厅、一间餐厅和一间厨房。然后我们收集一些数据，假设知道你当前所处房间和时间就能预测下一个你所处的房间的概率。举例来说，如果你是在厨房里，你有30%的机会待在厨房里，有30%的几率进入餐厅，20%的机会到客厅里去，10%的几率进入浴室，和10%的机会进入卧室。利用每一个房间的概率，我们可以构建一个预测链，评估你可能出现的下一个房间。
``

Making predictions a few states out might be useful, if we want to predict where someone in the house will be a little while after being in the kitchen. But since our predictions are just based on one observation of where a person is in the house, its reasonable to think they won’t be very good. If someone went from the bedroom to the bathroom, for example, its more likely they’ll go right back to the bedroom than if they had come from the kitchen. So the Markov Property doesn’t usually apply to the real world.

``
如果我们想预测某人在离开厨房后所处的房间，假设几个状态而做出的预测则可能是有用的。但是因为我们的预测只是基于一个人在房子里的观察，所以认为预测结果不会很好则是合理的。例如，如果有人从卧室走到浴室，相比从厨房走到浴室，他们更可能直接回到卧室。所以马尔可夫特性通常不适用于现实世界。
``

Running the Markov chain for thousands of iterations, however, does give the long-run prediction of what room you’re likely to be in.  More importantly, this prediction isn’t affected at all by which room the person began in! Intuitively, this makes sense: it doesn’t matter where someone is in the house at one point in time in order to simulate and describe where they are likely to be in the long-term, or _in general_. So Markov chains, which seem like an unreasonable way to model a random variable over a few periods, can be used to compute the long-run tendency of that variable if we understand the probabilities that govern its behavior.

``
然而，运行马尔可夫链进行数千次迭代，确实可以对您可能进入的房间进行长期预测。更重要的是，这个预测并没有受到这个人从哪个房间开始的影响！从直觉上讲，这是有道理的：时间因素对于模拟和描述某人在家中可能在长期或总体上的位置概率并不重要。所以马可夫链不是在几个周期内建模一个随机变量的合理方法，但如果我们理解控制其行为的概率，就可以用来计算这个变量的长期趋势。
``

* * *

With some knowledge of Monte Carlo simulations and Markov chains, I hope the math-free explanation of how MCMC methods work is pretty intuitive.

``
我希望通过介绍一些蒙特卡洛模拟和马尔科夫链的知识，可以使零数学基础的人更好理解MCMC的方法。
``

Recall that we are trying to estimate the posterior distribution for the parameter we’re interested in, average human height:

``
让我们回到原来的问题，即评估平均身高的后验分布：
``

![](https://cdn-images-1.medium.com/max/1600/0*XeUN0u_-EPh6MMCV.)


>I am not a visualization expert, nor apparently am I any good at keeping my example within the bounds of common sense: my example of the posterior distribution seriously overestimates average human height.<br>我不是一个可视化的专家，我也没有把这个例子放在常识的范围之内：请不要在意我的后验分布例子严重高估了人的平均身高，这一小问题。


We know that the posterior distribution is somewhere in the range of our prior distribution and our likelihood distribution, but for whatever reason, we can’t compute it directly. Using MCMC methods, we’ll effectively _draw samples from the posterior_ distribution, and then compute statistics like the average on the samples drawn.

``
我们知道某种程度上后验分布是在先验分布和可能性分布范围内的，但无论怎样，我们都无法直接计算它。利用MCMC方法，我们可以有效地从后验分布中抽取样本，然后计算出样本的均值。
``

To begin, MCMC methods pick a random parameter value to consider. The simulation will continue to generate random values (this is the Monte Carlo part), but subject to some rule for determining what makes a good parameter value. The trick is that, _for a pair of parameter values,_ it is possible to compute which is a better parameter value, by computing how likely each value is to explain the data, given our prior beliefs. If a randomly generated parameter value is better than the last one, it is added to the chain of parameter values with a certain probability determined by _how much_ better it is (this is the Markov chain part).

``
首先，MCMC方法选择一个随机参数值来考虑。模拟过程将继续生成随机值(这是蒙特卡罗部分)，但是要遵循一些规则来确定什么是好的参数值。关键在于，对于一对参数值，可以通过计算每个值对数据进行解释的可靠性来计算，从而确定更好的参数值。如果一个随机生成的参数值比上一个参数值好，那么它就会被添加到参数值链中，出现参数值好的概率取决于它最后结果的优劣(这是马尔科夫链部分)。
``

To explain this visually, lets recall that the height of a distribution at a certain value represents the probability of observing that value. Therefore, we can think of our parameter values (the x-axis) exhibiting areas of high and low probability, shown on the y-axis. For a single parameter, MCMC methods begin by randomly sampling along the x-axis:

``
为了直观地解释这一点，让我们回想一下，在某个值上分布的高度代表了观察到这个值的概率。因此，X轴（参数）对应的Y轴（概率）的值可高可低。对于单个参数，MCMC方法会首先在X轴上随机取样:
``

![](https://cdn-images-1.medium.com/max/1600/0*fqwsnMwAXnAdWxDH.)


>Red points are random parameter samples<br>红点代币随机参数样板

Since the random samples are subject to fixed probabilities, they tend to converge after a period of time in the region of highest probability for the parameter we’re interested in:

``
由于随机样本的概率是固定的，所以在我们感兴趣参数的最高概率区域中，它们趋向于收敛。
``

![](https://cdn-images-1.medium.com/max/1600/0*aYE_6eWzDWfVCOsQ.)


Blue points just represent random samples after an arbitrary point in time, when convergence is expected to have occurred. Note: I’m stacking point vertically purely for illustrative purposes.

``
蓝点只代表采样收敛后，在任意时间点之后的随机样本。 注意：为了说明的目的，我是垂直叠加的。
``

After convergence has occurred, MCMC sampling yields a set of points which are samples from the posterior distribution. Draw a histogram around those points, and compute whatever statistics you like:

``
在收敛发生之后，MCMC采样区间产生一组来自后验分布的样本点。 在这些点周围绘制直方图，并计算您喜欢的任何统计数据：
``


![](https://cdn-images-1.medium.com/max/1600/0*fMEvHsN-xlVzLw1H.)


Any statistic calculated on the set of samples generated by MCMC simulations is our best guess of that statistic on the true posterior distribution.

``
根据MCMC方法模拟生成的样本集计算出的任何统计数据分布特征，都是我们最可能接近真实后验分布统计量的结果。
``

MCMC methods can also be used to estimate the posterior distribution of more than one parameter (human height _and_ weight, say). For _n_ parameters, there exist regions of high probability in n-dimensional space where certain sets of parameter values better explain observed data. Therefore, I think of MCMC methods as randomly sampling inside a _probabilistic_ space to approximate the posterior distribution.

``
MCMC方法也可以用来评估多个参数的后验分布（比如说人的身高和体重）。对于n个参数，在n维空间中存在高概率的区域，其中某些参数值组合更好地解释观察到的数据。因此，我认为MCMC方法本质就是在概率空间内随机采样以近似后验分布。
``

* * *

Recall the short answer to the question ‘what are Markov chain Monte Carlo methods?’ Here it is again as a TL;DR:

``
回想一下“什么是马尔可夫链蒙特卡罗方法？”这个问题的简短答案。
``

> MCMC methods are used to approximate the posterior distribution of a parameter of interest by random sampling in a probabilistic space.<br>MCMC方法用于通过在概率空间中的随机采样来近似出兴趣参数的后验分布。

I hope I’ve explained that short answer, why you would use MCMC methods, and how they work. The inspiration for this post was a talk I gave as part of General Assembly’s Data Science Immersive course in Washington, DC. The goals of that talk were to explain Markov chain Monte Carlo methods to a non-technical audience, and I’ve tried to do the same here. Leave a comment if you think this explanation is off the mark in some way, or if it could be made more intuitive.

``
我希望我已经解释的简短答案，能让你理解为什么要使用MCMC方法，以及它们是如何工作的。这篇文章的灵感来源于我在华盛顿特区举行的Data Science Immersive课程。这个演讲的目的是向非技术观众解释马尔可夫链蒙特卡洛方法，我也试图通过此文完成同样的目标。如果您认为这个解释不符合标准，或者可以更直观一些，欢迎留下评论。
``
