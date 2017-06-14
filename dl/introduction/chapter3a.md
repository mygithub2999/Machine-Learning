# 第三章 改进神经网络的学习方法（上）

当一个高尔夫球员刚开始学习打高尔夫时，他们通常会在挥杆的练习上花费大多数时间。慢慢地他们才会在基本的挥杆上通过变化发展其他的击球方式，学习低飞球、左曲球和右曲球。类似的，我们现在仍然聚焦在反向传播算法的理解上。这就是我们的“基本挥杆”——神经网络中大部分工作学习和研究的基础。本章，我会解释若干技术能够用来提升我们关于反向传播的初级的实现，最终改进网络学习的方式。

本章涉及的技术包括：更好的代价函数的选择——[交叉熵](http://neuralnetworksanddeeplearning.com/chap3.html#the_cross-entropy_cost_function) 代价函数；四中规范化方法（L1 和 L2 规范化，dropout 和训练数据的人工扩展），这会让我们的网络在训练集之外的数据上更好地泛化；更好的[权重初始化方法](http://neuralnetworksanddeeplearning.com/chap3.html#weight_initialization)；还有[帮助选择好的超参数的启发式想法](http://neuralnetworksanddeeplearning.com/chap3.html#how_to_choose_a_neural_network's_hyper-parameters)。同样我也会再给出一些简要的[其他技术介绍](http://neuralnetworksanddeeplearning.com/chap3.html#other_techniques)。这些讨论之间的独立性比较大，所有你们可以随自己的意愿挑着看。另外我还会在代码中实现这些技术，使用他们来提高在第一章中的分类问题上的性能。

当然，我们仅仅覆盖了大量已经在神经网络中研究发展出的技术的一点点内容。此处我们学习深度学习的观点是想要在一些已有的技术上入门的最佳策略其实是深入研究一小部分最重要那些的技术点。掌握了这些关键技术不仅仅对这些技术本身的理解很有用，而且会深化你对使用神经网络时会遇到哪些问题的理解。这会让你们做好在需要时快速掌握其他技术的充分准备。

## 交叉熵代价函数
---
我们大多数人觉得错了就很不爽。在开始学习弹奏钢琴不久后，我在一个听众前做了处女秀。我很紧张，开始时将八度音阶的曲段演奏得很低。我很困惑，因为不能继续演奏下去了，直到有个人指出了其中的错误。当时，我非常尴尬。不过，尽管不开心，我们却能够因为明显的犯错快速地学习到正确的东西。你应该相信下次我再演奏肯定会是正确的！相反，在我们的错误不是很好的定义的时候，学习的过程会变得更加缓慢。

理想地，我们希望和期待神经网络可以从错误中快速地学习。在实践中，这种情况经常出现么？为了回答这个问题，让我们看看一个小例子。这个例子包含一个只有一个输入的神经元：

![](images/41.png)

我们会训练这个神经元来做一件非常简单的事：让输入$$1$$转化为 $$0$$。当然，这很简单了，手工找到合适的权重和偏差就可以了，不需要什么学习算法。然而，看起来使用梯度下降的方式来学习权重和偏差是很有启发的。所以，我们来看看神经元如何学习。

为了让事情确定化，我会首先将权重和偏差初始化为$$0.6$$和 $$0.9$$。这些就是一般的开始学习的选择，并没有任何刻意的想法。一开始的神经元的输出是 $$0.82$$，所以这离我们的目标输出 $$0.0$$ 还差得很远。点击右下角的“运行”按钮来看看神经元如何学习到让输出接近 $$0.0$$ 的。注意到，这并不是一个已经录好的动画，你的浏览器实际上是正在进行梯度的计算，然后使用梯度更新来对权重和偏差进行更新，并且展示结果。设置学习率  $$\eta=0.15$$ 进行学习一方面足够慢的让我们跟随学习的过程，另一方面也保证了学习的时间不会太久，几秒钟应该就足够了。代价函数是我们前面用到的二次函数，$$C$$。这里我也会给出准确的形式，所以不需要翻到前面查看定义了。注意，你可以通过点击 “Run” 按钮执行训练若干次。

![1](images/42.png)

![2](images/43.png)

![3](images/44.png)

![4](images/45.png)

> 我们这里是静态的例子，在原书中，使用的动态示例，所以为了更好的效果，请参考[原书的此处动态示例](http://neuralnetworksanddeeplearning.com/chap3.html)。

正如你所见，神经元快速地学到了使得代价函数下降的权重和偏差，给出了最终的输出为 $$0.09$$。这虽然不是我们的目标输出 $$0.0$$，但是已经挺好了。假设我们现在将初始权重和偏差都设置为 $$2.0$$。此时初始输出为 $$0.98$$，这是和目标值的差距相当大的。现在看看神经元学习的过程。点击“Run” 按钮：

![1](images/46.png)

![2](images/47.png)

![3](images/48.png)

![4](images/49.png)

![5](images/50.png)

虽然这个例子使用的了同样的学习率 $$\eta=0.15$$，我们可以看到刚开始的学习速度是比较缓慢的。对前 $$150$$ 左右的学习次数，权重和偏差并没有发生太大的变化。随后学习速度加快，与上一个例子中类似了，神经网络的输出也迅速接近$$0.0$$。

> 强烈建议参考[原书的此处动态示例](http://neuralnetworksanddeeplearning.com/chap3.html)感受学习过程的差异。

这种行为看起来和人类学习行为差异很大。正如我在此节开头所说，我们通常是在犯错比较明显的时候学习的速度最快。但是我们已经看到了人工神经元在其犯错较大的情况下其实学习很有难度。而且，这种现象不仅仅是在这个小例子中出现，也会再更加一般的神经网络中出现。为何学习如此缓慢？我们能够找到缓解这种情况的方法么？

为了理解这个问题的源头，想想神经元是按照偏导数（$$\partial C/\partial w$$ 和 $$\partial C/\partial b$$）和学习率（$$\eta$$）的乘积来改变权重和偏差的。所以，我们在说“学习缓慢”时，实际上就是说这些偏导数很小。理解他们为何这么小就是我们面临的挑战。为了理解这些，让我们计算偏导数看看。我们一直在用的是二次代价函数，定义如下

![](images/51.png)

其中 $$a$$ 是神经元的输出，其中训练输入为 $$x=1$$，$$y=0$$ 则是目标输出。显式地使用权重和偏差来表达这个，我们有 $$a=\sigma(z)$$，其中 $$z=wx+b$$。使用链式法则来求偏导数就有：

![](images/52.png)

其中我已经将 $$x=1$$ 和 $$y=0$$ 代入了。为了理解这些表达式的行为，让我们仔细看 $$\sigma'(z)$$ 这一项。首先回忆一下 $$\sigma$$ 函数图像：

![](images/53.png)

我们可以从这幅图看出，当神经元的输出接近 $$1$$ 的时候，曲线变得相当平，所以 $$\sigma'(z)$$ 就很小了。方程 (55) 和 (56) 也告诉我们 $$\partial C/\partial w$$ 和 $$\partial C/\partial b$$ 会非常小。这其实就是学习缓慢的原因所在。而且，我们后面也会提到，这种学习速度下降的原因实际上也是更加一般的神经网络学习缓慢的原因，并不仅仅是在这个特例中特有的。

### 引入交叉熵代价函数

那么我们如何解决这个问题呢？研究表明，我们可以通过使用交叉熵代价函数来替换二次代价函数。为了理解什么是交叉熵，我们稍微改变一下之前的简单例子。假设，我们现在要训练一个包含若干输入变量的的神经元，$$x_1,x_2,...$$ 对应的权重为 $$w_1,w_2,...$$ 和偏差，$$b$$：

![](images/54.png)

神经元的输出就是 $$a=\sigma(z)$$，其中 $$z=\sum_jw_jx_j+b$$ 是输入的带权和。我们如下定义交叉熵代价函数：

![](images/55.png)

其中 $$n$$ 是训练数据的总数，对所有的训练数据 $$x$$ 和 对应的目标输出 $$y$$ 进行求和。

公式(57)是否解决学习缓慢的问题并不明显。实际上，甚至将这个定义看做是代价函数也不是显而易见的！在解决学习缓慢前，我们来看看交叉熵为何能够解释成一个代价函数。

将交叉熵看做是代价函数有两点原因。第一，它使非负的，$$C>0$$。可以看出：(a) 公式(57)的和式中所有独立的项都是非负的，因为对数函数的定义域是 $$(0,1)$$；(b) 前面有一个负号。

第二，如果神经元实际的输出接近目标值。假设在这个例子中，$$y=0$$ 而 $$a\approx 0$$。这是我们想到得到的结果。我们看到公式(57)中第一个项就消去了，因为 $$y=0$$，而第二项实际上就是 $$-\ln (1-a)\approx 0$$。反之，$$y=1$$ 而 $$a\approx 1$$。所以在实际输出和目标输出之间的差距越小，最终的交叉熵的值就越低了。

综上所述，交叉熵是非负的，在神经元达到很好的正确率的时候会接近 $$0$$。这些其实就是我们想要的代价函数的特性。其实这些特性也是二次代价函数具备的。所以，交叉熵就是很好的选择了。但是交叉熵代价函数有一个比二次代价函数更好的特性就是它避免了学习速度下降的问题。为了弄清楚这个情况，我们来算算交叉熵函数关于权重的偏导数。我们将 $$a=\sigma(z)$$ 代入到公式 (57) 中应用两次链式法则，得到：

![](images/56.png)

将结果合并一下，简化成：

![](images/57.png)

根据 $$\sigma(z) = 1/(1+e^{-z})$$ 的定义，和一些运算，我们可以得到 $$\sigma'(z) = \sigma(z)(1-\sigma(z))$$。后面在练习中会要求你计算这个，现在可以直接使用这个结果。我们看到 $$\sigma'$$ 和 $$\sigma(z)(1-\sigma(z))$$ 这两项在方程中直接约去了，所以最终形式就是：

![](images/58.png)

这是一个优美的公式。它告诉我们权重学习的速度受到 $$\sigma(z)-y$$，也就是输出中的误差的控制。更大的误差，更快的学习速度。这是我们直觉上期待的结果。特别地，这个代价函数还避免了像在二次代价函数中类似公式中 $$\sigma'(z)$$ 导致的学习缓慢，见公式(55)。当我们使用交叉熵的时候，$$\sigma'(z)$$ 被约掉了，所以我们不再需要关心它是不是变得很小。这种约除就是交叉熵带来的特效。实际上，这也并不是非常奇迹的事情。我们在后面可以看到，交叉熵其实只是满足这种特性的一种选择罢了。

根据类似的方法，我们可以计算出关于偏差的偏导数。我这里不再给出详细的过程，你可以轻易验证得到

![](images/59.png)

### 练习

* 验证 $$\sigma'(z) = \sigma(z)(1-\sigma(z))$$。

让我们重回最原初的例子，来看看换成了交叉熵之后的学习过程。现在仍然按照前面的参数配置来初始化网络，开始权重为 $$0.6$$，而偏差为 $$0.9$$。点击“Run”按钮看看在换成交叉熵之后网络的学习情况：

![1](images/60.png)

![2](images/61.png)


![3](images/62.png)

![4](images/63.png)

![5](images/64.png)

毫不奇怪，在这个例子中，神经元学习得相当出色，跟之前差不多。现在我们再看看之前出问题的那个例子，权重和偏差都初始化为 $$2.0$$：

![1](images/65.png)

![2](images/66.png)

![3](images/67.png)

![4](images/68.png)

![5](images/69.png)

成功了！这次神经元的学习速度相当快，跟我们预期的那样。如果你观测的足够仔细，你可以发现代价函数曲线要比二次代价函数训练前面部分要陡很多。正是交叉熵带来的快速下降的坡度让神经元在处于误差很大的情况下能够逃脱出学习缓慢的困境，这才是我们直觉上所期待的效果。

我们还没有提及关于学习率的讨论。刚开始使用二次代价函数的时候，我们使用了 $$\eta=0.15$$。在新例子中，我们还应该使用同样的学习率么？实际上，根据不同的代价函数，我们不能够直接去套用同样的学习率。这好比苹果和橙子的比较。对于这两种代价函数，我只是通过简单的实验来找到一个能够让我们看清楚变化过程的学习率的值。尽管我不愿意提及，但如果你仍然好奇，这个例子中我使用了 $$\eta=0.005$$。

你可能会反对说，上面学习率的改变使得上面的图失去了意义。谁会在意当学习率的选择是任意挑选的时候神经元学习的速度？！这样的反对其实没有抓住重点。上面的图例不是想讨论学习的绝对速度。而是想研究学习速度的变化情况。特别地，当我们使用二次代价函数时，学习在神经元犯了明显的错误的时候却比学习快接近真实值的时候缓慢；而使用交叉熵学习正是在神经元犯了明显错误的时候速度更快。这些现象并不是因为学习率的改变造成的。

我们已经研究了一个神经元的交叉熵。不过，将其推广到多个神经元的多层神经网络上也是很简单的。特别地，假设 $$y=y_1,y_2,...$$ 是输出神经元上的目标值，而 $$a_1^L,a_2^L,...$$是实际输出值。那么我们定义交叉熵如下

![](images/70.png)

除了这里需要对所有输出神经元进行求和外，这个其实和我们早前的公式(57)一样的。这里不会给出一个推算的过程，但需要说明的时使用公式(63)确实会在很多的神经网络中避免学习的缓慢。如果你感兴趣，你可以尝试一下下面问题的推导。

那么我们应该在什么时候用交叉熵来替换二次代价函数？实际上，如果在输出神经元使用 sigmoid 激活函数时，交叉熵一般都是更好的选择。为什么？考虑一下我们初始化网络的时候通常使用某种随机方法。可能会发生这样的情况，这些初始选择会对某些训练输入误差相当明显——比如说，目标输出是 $$1$$，而实际值是 $$0$$，或者完全反过来。如果我们使用二次代价函数，那么这就会导致学习速度的下降。它并不会完全终止学习的过程，因为这些权重会持续从其他的样本中进行学习，但是显然这不是我们想要的效果。

### 练习

* 一个小问题就是刚接触交叉熵时，很难一下子记住那些表达式对应的角色。又比如说，表达式的正确形式是 $$−[y\ln a+(1−y)\ln (1−a)]$$
 或者 $$−[a\ln y+(1−a)\ln (1−y)]$$。在 $$y=0$$ 或者 $$1$$ 的时候第二个表达式的结果怎样？这个问题会影响第一个表达式么？为什么？
* 在对单个神经元讨论中，我们指出如果对所有的训练数据有 $$\sigma(z)\approx y$$，交叉熵会很小。这个论断其实是和 $$y$$ 只是等于 $$1$$ 或者 $$0$$ 有关。这在分类问题一般是可行的，但是对其他的问题（如回归问题）$$y$$ 可以取 $$0$$  和 $$1$$ 之间的中间值的。证明，交叉熵在 $$\sigma(z)=y$$ 时仍然是最小化的。此时交叉熵的表示是：

![](images/71.png)

而其中 $$−[y\ln y+(1−y)\ln (1−y)]$$ 有时候被称为 [二元熵](http://en.wikipedia.org/wiki/Binary_entropy_function)

### 问题

* **多层多神经元网络** 用上一章的定义符号，证明对二次代价函数，关于输出层的权重的偏导数为

![](images/72.png)

项 $$\sigma'(z_j^L)$$ 会在一个输出神经元困在错误值时导致学习速度的下降。证明对于交叉熵代价函数，针对一个训练样本 $$x$$ 的输出误差 $$\delta^L$$为

![](images/73.png)

使用这个表达式来证明关于输出层的权重的偏导数为

![](images/74.png)

这里 $$\sigma'(z_j^L)$$ 就消失了，所以交叉熵避免了学习缓慢的问题，不仅仅是在一个神经元上，而且在多层多元网络上都起作用。这个分析过程稍作变化对偏差也是一样的。如果你对这个还不确定，那么请仔细研读一下前面的分析。

* **在输出层使用线性神经元时使用二次代价函数** 假设我们有一个多层多神经元网络，最终输出层的神经元都是线性的，输出不再是 sigmoid 函数作用的结果，而是 $$a_j^L = z_j^L$$。证明如果我们使用二次代价函数，那么对单个训练样本 $$x$$ 的输出误差就是

![](images/75.png)

类似于前一个问题，使用这个表达式来证明关于输出层的权重和偏差的偏导数为

![](images/76.png)

这表明如果输出神经元是线性的那么二次代价函数不再会导致学习速度下降的问题。在此情形下，二次代价函数就是一种合适的选择。

### 使用交叉熵来对 MNIST 数字进行分类

交叉熵很容易作为使用梯度下降和反向传播方式进行模型学习的一部分来实现。我们将会在下一章进行对前面的程序 `network.py` 的改进。新的程序写在 `network2.py` 中，不仅使用了交叉熵，还有本章中介绍的其他的技术。现在我们看看新的程序在进行 MNIST 数字分类问题上的表现。如在第一章中那样，我们会使用一个包含 $$30$$ 个隐藏元的网络，而 minibatch 的大小也设置为 $$10$$。我们将学习率设置为 $$\eta=0.5$$ 然后训练 $$30$$ 回合。`network2.py` 的接口和 `network.py` 稍微不同，但是过程还是很清楚的。你可以使用如 `help(network2.Network.SGD)` 这样的命令来检查对应的文档。

```python
>>> import mnist_loader
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper()
>>> import network2
>>> net = network2.Network([784, 30, 10], cost=network2.CrossEntropyCost)
>>> net.large_weight_initializer()
>>> net.SGD(training_data, 30, 10, 0.5, evaluation_data=test_data,
... monitor_evaluation_accuracy=True)
```

注意看下，`net.large_weight_initializer()` 命令使用和第一章介绍的同样的方式来进行权重和偏差的初始化。运行上面的代码我们得到了一个 95.49% 准确度的网络。这跟我们在第一章中使用二次代价函数得到的结果相当接近了，95.42%。

同样也来试试使用 $$100$$ 个隐藏元的交叉熵及其他参数保持不变的情况。在这个情形下，我们获得了 96.82% 的准确度。相比第一章使用二次代价函数的结果 96.59%，这有一定的提升。这看起来是一个小小的变化，但是考虑到误差率已经从 3.41% 下降到 3.18%了。我们已经消除了原误差的 $$1/14$$。这其实是可观的改进。

令人振奋的是交叉熵代价函数给了我们类似的或者更好的结果。然而，这些结果并没有能够确定性地证明交叉熵是更好的选择。原因在于我已经在选择诸如学习率，minibatch 大小等等这样的超参数上做出了一些努力。为了让这些提升更具说服力，我们需要进行对超参数进行深度的优化。当然，这些结果都挺好的，强化了我们早先关于交叉熵优于二次代价的理论论断。

这只是本章后面的内容和整本书剩余内容中的更为一般的模式的一部分。我们将给出一个新的技术，然后进行尝试，随后获得“提升的”结果。当然，看到这些进步是很好的。但是这些提升的解释一般来说都困难重重。在进行了大量工作来优化所有其他的超参数，使得提升真的具有说服力。工作量很大，需要大量的计算能力，我们也不会进行这样耗费的调查。相反，我采用上面进行的那些不正式的测试来达成目标。所以你要记住这样的测试缺少确定性的证明，需要注意那些使得论断失效的信号。

至此，我们已经花了大量篇幅介绍交叉熵。为何对一个只能给出一点点性能提升的技术上花费这么多的精力？后面，我们会看到其他的技术——规范化，会带来更大的提升效果。所以为何要这么细致地讨论交叉熵？部分原因在于交叉熵是一种广泛使用的代价函数，值得深入理解。但是更加重要的原因是神经元的饱和是神经网络中一个关键的问题，整本书都会不断回归到这个问题上。因此我现在深入讨论交叉熵就是因为这是一种开始理解神经元饱和和如何解决这个问题的很好的实验。

### 交叉熵的含义？源自哪里？

我们对于交叉熵的讨论聚焦在代数分析和代码实现。这虽然很有用，但是也留下了一个未能回答的更加宽泛的概念上的问题，如：交叉熵究竟表示什么？存在一些直觉上的思考交叉熵的方法么？我们如何想到这个概念？

让我们从最后一个问题开始回答：什么能够激发我们想到交叉熵？假设我们发现学习速度下降了，并理解其原因是因为在公式(55)(56)中的 $$\sigma'(z)$$ 那一项。在研究了这些公式后，我们可能就会想到选择一个不包含 $$\sigma'(z)$$ 的代价函数。所以，这时候对一个训练样本 $$x$$，代价函数是 $$C=C_x$$ 就满足：

![](images/77.png)

如果我们选择的代价函数满足这些条件，那么同样他们会拥有遇到明显误差时的学习速度越快这样的特性。这也能够解决学习速度下降的问题。实际上，从这些公式开始，现在就给你们看看若是跟随自身的数学嗅觉推导出交叉熵的形式是可能的。我们来推一下，由链式法则，我们有

![](images/78.png)

使用 $$\sigma'(z)=\sigma(z)(1-\sigma(z))=a(1-a)$$，上个等式就变成

![](images/79.png)

对比等式(72)，我们有

![](images/80.png)

对此方程关于 $$a$$ 进行积分，得到 

![](images/81.png)

其中 $$constant$$ 是积分常量。这是一个单独的训练样本 $$x$$ 对代价函数的贡献。为了得到整个的代价函数，我们需要对所有的训练样本进行平均，得到了

![](images/82.png)

而这里的常量就是所有单独的常量的平均。所以我们看到方程(71)和(72)唯一确定了交叉熵的形式，并加上了一个常量的项。这个交叉熵并不是凭空产生的。而是一种我们以自然和简单的方法获得的结果。

那么交叉熵直觉含义又是什么？我们如何看待它？深入解释这一点会将我们带到一个不大愿意讨论的领域。然而，还是值得提一下，有一种源自信息论的解释交叉熵的标准方式。粗略地说，交叉熵是惊奇的度量（measure of surprise）。特别地，我们的神经元想要要计算函数 $$x\rightarrow y=y(x)$$。但是，它用函数$$x\rightarrow a=a(x)$$ 进行了替换。假设我们将 $$a$$ 想象成我们神经元估计 $$y = 1$$ 概率，而 $$1-a$$ 则是 $$y=0$$ 的概率。如果输出我们期望的结果，惊奇就会小一点；反之，惊奇就大一些。当然，我这里没有严格地给出“惊奇”到底意味着什么，所以看起来像在夸夸奇谈。但是实际上，在信息论中有一种准确的方式来定义惊奇究竟是什么。不过，我也不清楚在网络上，哪里有好的，短小的自包含对这个内容的讨论。但如果你要深入了解的话，Wikipedia 包含一个[简短的总结](http://en.wikipedia.org/wiki/Cross_entropy#Motivation)，这会指引你正确地探索这个领域。而更加细节的内容，你们可以阅读[Cover and Thomas](http://books.google.ca/books?id=VWq5GG6ycxMC)的第五章涉及 Kraft 不等式的有关信息论的内容。

### 问题

* 我们已经深入讨论了使用二次代价函数的网络中在输出神经元饱和时候学习缓慢的问题，另一个可能会影响学习的因素就是在方程(61)中的 $$x_j$$ 项。由于此项，当输入 $$x_j$$ 接近 $$0$$ 时，对应的权重 $$x_j$$ 会学习得相当缓慢。解释为何不可以通过改变代价函数来消除 $$x_j$$ 项的影响。

## Softmax

本章，我们大多数情况会使用交叉熵来解决学习缓慢的问题。但是，我希望简要介绍一下基于 softmax 神经元层的解决这个问题的另一种观点。我们不会实际在剩下的章节中使用 softmax 层，所以你如果赶时间，就可以跳到下一个小节了。不过，softmax 仍然有其重要价值，一方面它本身很有趣，另一方面，因为我们会在第六章在对深度神经网络的讨论中使用 softmax 层。

softmax 的想法其实就是为神经网络定义一种新式的输出层。开始时和 sigmoid 层一样的，首先计算带权输入 $$z_j^L = \sum_k w_{jk}^La_k^{L-1}+b_j^L$$。不过，这里我们不会使用 sigmoid 函数来获得输出。而是，会应用一种叫做 softmax 函数在 $$z_j^L$$ 上。根据这个函数，激活值 $$a_j^L$$ 就是

![](images/83.png)

其中，分母是对所有的输出神经元进行求和。

如果你不习惯这个函数，方程(78)可能看起来会比较难理解。因为对于使用这个函数的原因你不清楚。为了更好地理解这个公式，假设我们有一个包含四个输出神经元的神经网络，对应四个带权输入，表示为 $$z_1^L, z_2^L, z_3^L, z_4^L$$。下面的例子可以进行对应权值的调整，并给出对应的激活值的图像。可以通过固定其他值，来看看改变 $$z_4^L$$ 的值会产生什么样的影响：

![1](images/84.png)


![2](images/85.png)


![3](images/86.png)

> 这里给出了三个选择的图像，建议去原网站体验

在我们增加 $$z_4^L$$ 的时候，你可以看到对应激活值 $$a_4^L$$ 的增加，而其他的激活值就在下降。类似地，如果你降低 $$z_4^L$$ 那么 $$a_4^L$$ 就随之下降。实际上，如果你仔细看，你会发现在两种情形下，其他激活值的整个改变恰好填补了 $$a_4^L$$ 的变化的空白。原因很简单，根据定义，输出的激活值加起来正好为 $$1$$，使用公式(78)我们可以证明：

![](images/87.png)

所以，如果 $$a_4^L$$ 增加，那么其他输出激活值肯定会总共下降相同的量，来保证和式为 $$1$$。当然，类似的结论对其他的激活函数也需要满足。

方程(78)同样保证输出激活值都是正数，因为指数函数是正的。将这两点结合起来，我们看到 softmax 层的输出是一些相加为 $$1$$ 正数的集合。换言之，softmax 层的输出可以被看做是一个概率分布。

这样的效果很令人满意。在很多问题中，将这些激活值作为网络对于某个输出正确的概率的估计非常方便。所以，比如在 MNIST 分类问题中，我们可以将 $$a_j^L$$ 解释成网络估计正确数字分类为 $$j$$ 的概率。

对比一下，如果输出层是 sigmoid 层，那么我们肯定不能假设激活值形成了一个概率分布。我不会证明这一点，但是源自 sigmoid 层的激活值是不能够形成一种概率分布的一般形式的。所以使用 sigmoid 输出层，我们没有关于输出的激活值的简单的解释。

### 练习

* 构造例子表明在使用 sigmoid 输出层的网络中输出激活值 $$a_j^L$$ 的和并不会确保为 $$1$$。

我们现在开始体会到 softmax 函数的形式和行为特征了。来回顾一下：在公式(78)中的指数函数确保了所有的输出激活值是正数。然后分母的求和又保证了 softmax 的输出和为 $$1$$。所以这个特定的形式不再像之前那样难以理解了：反而是一种确保输出激活值形成一个概率分布的自然的方式。你可以将其想象成一种重新调节 $$z_j^L$$   的方法，然后将这个结果整合起来构成一个概率分布。

### 练习

* **softmax 的单调性** 证明如果 $$j=k$$ 则 $$\partial a_j^L/\partial z_k^L$$ 为正，否则为负。结果是，增加 $$z_k^L$$ 会提高对应的输出激活值 $$a_k^L$$ 并降低其他所有输出激活值。我们已经在滑条示例中实验性地看到了这一点，这里需要你给出一个严格证明。
* **softmax 的非局部性** sigmoid 层的一个好处是输出 $$a_j^L$$ 是对应带权输入 $$a_j^L = \sigma(z_j^L)$$ 的函数。解释为何对于 softmax 来说，并不是这样的情况：仍和特定的输出激活值 $$a_j^L$$ 依赖所有的带权输入。

### 问题

* **逆转 softmax 层** 假设我们有一个使用 softmax 输出层的神经网络，然后激活值 $$a_j^L$$ 已知。证明对应带权输入的形式为 $$z_j^L = \ln a_j^L + C$$，其中 $$C$$ 是独立于 $$j$$ 的。

**学习缓慢问题**：我们现在已经对 softmax 神经元有了一定的认识。但是我们还没有看到 softmax 会怎么样解决学习缓慢问题。为了理解这点，先定义一个 log-likelihood 代价函数。我们使用 $$x$$ 表示训练输入，$$y$$ 表示对应的目标输出。然后关联这个训练输入样本的 log-likelihood 代价函数就是

![](images/88.png)

所以，如果我们训练的是 MNIST 图像，输入为 $$7$$ 的图像，那么对应的 log-likelihood 代价就是 $$-\ln a_7^L$$。看看这个直觉上的含义，想想当网络表现很好的时候，也就是确认输入为 $$7$$ 的时候。这时，他会估计一个对应的概率 $$a_7^L$$ 跟 $$1$$ 非常接近，所以代价 $$-\ln a_7^L$$ 就会很小。反之，如果网络的表现糟糕时，概率 $$a_7^L$$ 就变得很小，代价 $$-\ln a_7^L$$ 随之增大。所以 log-likelihood 代价函数也是满足我们期待的代价函数的条件的。

那关于学习缓慢问题呢？为了分析它，回想一下学习缓慢的关键就是量 $$\partial C/\partial w_{jk}^L$$ 和 $$\partial C/\partial b_j^L$$ 的变化情况。我不会显式地给出详细的推导——请你们自己去完成这个过程——但是会给出一些关键的步骤：

![](images/89.png)

> 请注意这里的表示上的差异，这里的 $$y$$ 和之前的目标输出值不同，是离散的向量表示，对应位的值为 $$1$$，而其他为 $$0$$。
这些方程其实和我们前面对交叉熵得到的类似。就拿方程(82) 和 (67) 比较。尽管后者我对整个训练样本进行了平均，不过形式还是一致的。而且，正如前面的分析，这些表达式确保我们不会遇到学习缓慢的问题。实际上，将 softmax 输出层和 log-likelihood 组合对照 sigmoid 输出层和交叉熵的组合类比着看是非常有用的。

有了这样的相似性，你会使用哪一种呢？实际上，在很多应用场景中，这两种方式的效果都不错。本章剩下的内容，我们会使用 sigmoid 输出层和交叉熵的组合。后面，在第六章中，我们有时候会使用 softmax 输出层和 log-likelihood 的则和。切换的原因就是为了让我们的网络和某些在具有影响力的学术论文中的形式更为相似。作为一种更加通用的视角，softmax 加上 log-likelihood 的组合更加适用于那些需要将输出激活值解释为概率的场景。当然这不总是合理的，但是在诸如 MNIST 这种有着不重叠的分类问题上确实很有用。

### 问题

* 推导方程(81) 和 (82)
* **softmax 这个名称从何处来？** 假设我们改变一下 softmax 函数，使得输出激活值定义如下

![](images/90.png)

其中 $$c$$ 是正的常量。注意 $$c=1$$ 对应标准的 softmax 函数。但是如果我们使用不同的 $$c$$ 得到不同的函数，其实最终的量的结果却和原来的 softmax 差不多。特别地，证明输出激活值也会形成一个概率分布。假设我们允许 $$c$$ 足够大，比如说 $$c\rightarrow \infty$$。那么输出激活值 $$a_j^L$$ 的极限值是什么？在解决了这个问题后，你应该能够理解 $$c=1$$ 对应的函数是一个最大化函数的 softened 版本。这就是 softmax 的来源。
> 这让我联想到 EM 算法，对 k-Means 算法的一种推广。

* **softmax 和 log-likelihood 的反向传播** 上一章，我们推到了使用 sigmoid 层的反向传播算法。为了应用在 softmax 层的网络上，我们需要搞清楚最后一层上误差的表示 $$\delta_j^L \equiv \partial C/\partial z_j^L$$。证明形式如下：

![](images/91.png)

使用这个表达式，我们可以应用反向传播在采用了 softmax 输出层和 log-likelihood 的网络上。

## 过匹配和规范化
---
诺贝尔奖得主美籍意大利裔物理学家恩里科·费米曾被问到他对一个同僚提出的尝试解决一个重要的未解决物理难题的数学模型。模型和实验非常匹配，但是费米却对其产生了怀疑。他问模型中需要设置的自由参数有多少个。答案是“4”。费米回答道：“我记得我的朋友约翰·冯·诺依曼过去常说，有四个参数，我可以模拟一头大象，而有五个参数，我还能让他卷鼻子。”

这里，其实是说拥有大量的自由参数的模型能够描述特别神奇的现象。即使这样的模型能够很好的拟合已有的数据，但并不表示是一个好模型。因为这可能只是因为模型中足够的自由度使得它可以描述几乎所有给定大小的数据集，不需要对现象的本质有创新的认知。所以发生这种情形时，模型对已有的数据会表现的很好，但是对新的数据很难泛化。对一个模型真正的测验就是它对没有见过的场景的预测能力。

费米和冯·诺依曼对有四个参数的模型就开始怀疑了。我们用来对 MNIST 数字分类的 $$30$$ 个隐藏神经元神经网络拥有将近  $$24,000$$ 个参数！当然很多。我们有 $$100$$ 个隐藏元的网络拥有将近 $$80,000$$ 个参数，而目前最先进的深度神经网络包含百万级或者十亿级的参数。我们应当信赖这些结果么？

让我们将问题暴露出来，通过构造一个网络泛化能力很差的例子。我们的网络有 $$30$$ 个隐藏神经元，共 $$23,860$$ 个参数。但是我们不会使用所有 $$50,000$$ 幅训练图像。相反，我们只使用前 $$1000$$ 幅图像。使用这个受限的集合，会让泛化的问题突显。按照同样的方式，使用交叉熵代价函数，学习率设置为 $$\eta=0.5$$ 而 minibatch 大小设置为 $$10$$。不过这里我们训练回合设置为 $$400$$，比前面的要多很多，因为我们只用了少量的训练样本。我们现在使用 `network2` 来研究代价函数改变的情况：

```python
>>> import mnist_loader 
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper()
>>> import network2 
>>> net = network2.Network([784, 30, 10], cost=network2.CrossEntropyCost) 
>>> net.large_weight_initializer()
>>> net.SGD(training_data[:1000], 400, 10, 0.5, evaluation_data=test_data,
... monitor_evaluation_accuracy=True, monitor_training_cost=True)
```

使用上面的结果，我们可以画出代价函数变化的情况：


![](images/92.png)

这看起来令人振奋，因为代价函数有一个光滑的下降，跟我们预期一致。注意，我只是展示了 $$200$$ 到 $$399$$ 回合的情况。这给出了很好的近距离理解训练后期的情况，这也是出现有趣现象的地方。

让我们看看分类准确度在测试集上的表现：

![](images/93.png)

这里我还是聚焦到了后面的过程。 在前 $$200$$ 回合（图中没有显示）准确度提升到了 82%。然后学习逐渐变缓。最终，在 $$280$$ 回合左右分类准确度就停止了增长。后面的回合，仅仅看到了在 $$280$$ 回合准确度周围随机的震荡。将这幅图和前面的图进行对比，和训练数据相关的代价函数持续平滑下降。如果我们只看哪个代价，我们会发现模型的表现变得“更好”。但是测试准确度展示了提升只是一种假象。就像费米不大喜欢的那个模型一样，我们的网络在 $$280$$ 回合后就不在能够繁华到测试数据上。所以这种学习不大有用。也可以说网络在 $$280$$ 后就过匹配（或者过度训练）了。

你可能想知道这里的问题是不是由于我们看的是训练数据的代价，而对比的却是测试数据上的分类准确度导致的。换言之，可能我们这里在进行苹果和橙子的对比。如果我们比较训练数据上的代价和测试数据上的代价，会发生什么，我们是在比较类似的度量么？或者可能我们可以比较在两个数据集上的分类准确度啊？实际上，不管我们使用什么度量的方式尽管，细节会变化，但本质上都是一样的。
让我们来看看测试数据集上的代价变化情况：

![](images/94.png)

我们可以看到测试集上的代价在 $$15$$ 回合前一直在提升，随后越来越差，尽管训练数据机上的代价表现是越来越好的。这其实是另一种模型过匹配的标志。尽管，这里带来了关于我们应当将 $$15$$ 还是 $$280$$ 回合当作是过匹配占主导的时间点的困扰。从一个实践角度，我们真的关心的是提升测试数据集上的分类准确度，而测试集合上的代价不过是分类准确度的一个反应。所以更加合理的选择就是将 $$280$$ 看成是过匹配开始占统治地位的时间点。

另一个过匹配的信号在训练数据上的分类准确度上也能看出来：

![](images/95.png)

准确度一直在提升接近 100%。也就是说，我们的网络能够正确地对所有 $$1000$$ 幅图像进行分类！而在同时，我们的测试准确度仅仅能够达到 82.27%。所以我们的网络实际上在学习训练数据集的特例，而不是能够一般地进行识别。我们的网络几乎是在单纯记忆训练集合，而没有对数字本质进行理解能够泛化到测试数据集上。

过匹配是神经网络的一个主要问题。这在现代网络中特别正常，因为网络权重和偏差数量巨大。为了高效地训练，我们需要一种检测过匹配是不是发生的技术，这样我们不会过度训练。并且我们也想要找到一些技术来降低过匹配的影响。

检测过匹配的明显方法是使用上面的方法——跟踪测试数据集合上的准确度随训练变化情况。如果我们看到测试数据上的准确度不再提升，那么我们就停止训练。当然，严格地说，这其实并非是过匹配的一个必要现象，因为测试集和训练集上的准确度可能会同时停止提升。当然，采用这样的方式是可以阻止过匹配的。

实际上，我们会使用这种方式变形来试验。记得之前我们载入 MNIST 数据的时候有：

```python 
>>> import mnist_loader 
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper()
```

到现在我们一直在使用 `training_data` 和 `test_data`，没有用过 `validation_data`。`validation_data` 中包含了 $$10,000$$ 幅数字图像，这些图像和训练数据集中的 $$50,000$$ 幅图像以及测试数据集中的 $$10,000$$ 幅都不相同。我们会使用 `validation_data` 来防止过匹配。我们会使用和上面应用在 `test_data` 的策略。我们每个回合都计算在 `validation_data` 上的分类准确度。一旦分类准确度已经饱和，就停止训练。这个策略被称为 **提前停止**（Early stopping）。当然，实际应用中，我们不会立即知道什么时候准确度会饱和。相反，我们会一直训练知道我们确信准确度已经饱和。
> 这里需要一些判定标准来确定什么时候停止。在我前面的图中，将 $$280$$ 回合看成是饱和的地方。可能这有点太悲观了。因为神经网络有时候会训练过程中处在一个平原期，然后又开始提升。如果在 $$400$$ 回合后，性能又开始提升（也许只是一些少量提升），那我也不会诧异。所以，在**提前停止**中采取一点激进的策略也是可以的。

为何要使用 `validation_data` 来替代 `test_data` 防止过匹配问题？实际上，这是一个更为一般的策略的一部分，这个一般的策略就是使用 `validation_data` 来衡量不同的超参数（如训练回合，学习率，最好的网络架构等等）的选择的效果。我们使用这样方法来找到超参数的合适值。因此，尽管到现在我并没有提及这点，但其实本书前面已经稍微介绍了一些超参数选择的方法。

当然，这对于我们前面关于 `validation_data` 取代 `test_data` 来防止过匹配的原因仍旧没有回答。实际上，有一个更加一般的问题，就是为何用`validation_data` 取代 `test_data` 来设置更好的超参数？为了理解这点，想想当设置超参数时，我们想要尝试许多不同的超参数选择。如果我们设置超参数是基于 `test_data` 的话，可能最终我们就会得到过匹配于 `test_data` 的超参数。也就是说，我们可能会找到那些 符合 `test_data` 特点的超参数，但是网络的性能并不能够泛化到其他数据集合上。我们借助 `validation_data` 来克服这个问题。然后一旦获得了想要的超参数，最终我们就使用 `test_data` 进行准确度测量。这给了我们在 `test_data` 上结果是一个网络泛化能力真正的度量方式的信心。换言之，你可以将验证集看成是一种特殊的训练数据集能够帮助我们学习好的超参数。这种寻找好的超参数的观点有时候被称为 **hold out** 方法，因为 `validation_data` 是从训练集中保留出来的一部分。

在实际应用中，甚至在衡量了测试集上的性能后，我们可能也会改变想法并去尝试另外的方法——也许是一种不同的网络架构——这将会引入寻找新的超参数的的过程。如果我们这样做，难道不会产生过匹配于 `test_data` 的困境么？我们是不是需要一种潜在无限大的数据集的回归，这样才能够确信模型能够泛化？去除这样的疑惑其实是一个深刻而困难的问题。但是对实际应用的目标，我们不会担心太多。相反，我们会继续采用基于 `training_data, validation_data, and test_data` 的基本 hold out 方法。

我们已经研究了在使用 $$1,000$$ 幅训练图像时的过匹配问题。那么如果我们使用所有的训练数据会发生什么？我们会保留所有其他的参数都一样（$$30$$ 个隐藏元，学习率 $$0.5$$，mini-batch 规模为 $$10$$），但是训练回合为 $$30$$ 次。下图展示了分类准确度在训练和测试集上的变化情况。注意我们使用的测试数据，而不是验证集合，为了让结果看起来和前面的图更方便比较。

![](images/96.png)

如你所见，测试集和训练集上的准确度相比我们使用 $$1,000$$ 个训练数据时相差更小。特别地，在训练数据上的最佳的分类准确度 97.86% 只比测试集上的 95.33% 准确度高一点点。而之前的例子中，这个差距是 17.73%！过匹配仍然发生了，但是已经减轻了不少。我们的网络从训练数据上更好地泛化到了测试数据上。一般来说，最好的降低过匹配的方式之一就是增加训练样本的量。有了足够的训练数据，就算是一个规模非常大的网络也不大容易过匹配。不幸的是，训练数据其实是很难或者很昂贵的资源，所以这不是一种太切实际的选择。

### 规范化

增加训练样本的数量是一种减轻过匹配的方法。还有其他的一下方法能够减轻过匹配的程度么？一种可行的方式就是降低网络的规模。然而，大的网络拥有一种比小网络更强的潜力，所以这里存在一种应用冗余性的选项。

幸运的是，还有其他的技术能够缓解过匹配，即使我们只有一个固定的网络和固定的训练集合。这种技术就是**规范化**。本节，我会给出一种最为常用的规范化手段——有时候被称为权重下降（weight decay）或者 **L2** 规范化。**L2** 规范化的想法是增加一个额外的项到代价函数上，这个项叫做 **规范化** 项。下面是规范化交叉熵：

![](images/97.png)

其中第一个项就是常规的交叉熵的表达式。第二个现在加入到就是所有权重的平方的和。然后使用一个因子 $$\lambda/2n$$ 进行量化调整，其中 $$\lambda > 0$$ 可以成为 **规范化参数**，而 $$n$$ 就是训练集合的大小。我们会在后面讨论 $$\lambda$$ 的选择策略。需要注意的是，规范化项里面并不包含偏差。这点我们后面也会在讲述。

当然，对其他的代价函数也可以进行规范化，例如二次代价函数。类似的规范化的形式如下：

![](images/98.png)

两者都可以写成这样：

![](images/99.png)

其中 $$C_0$$ 是原始的代价函数。

直觉地看，规范化的效果是让网络倾向于学习小一点的权重，其他的东西都一样的。大的权重只有能够给出代价函数第一项足够的提升时才被允许。换言之，规范化可以当做一种寻找小的权重和最小化原始的代价函数之间的折中。这两部分之前相对的重要性就由 $$\lambda$$ 的值来控制了：$$\lambda$$ 越小，就偏向于最小化原始代价函数，反之，倾向于小的权重。

现在，对于这样的折中为何能够减轻过匹配还不是很清楚！但是，实际表现表明了这点。我们会在下一节来回答这个问题。但现在，我们来看看一个规范化的确减轻过匹配的例子。

为了构造这个例子，我们首先需要弄清楚如何将随机梯度下降算法应用在一个规范化的神经网络上。特别地，我们需要知道如何计算偏导数 $$\partial C/\partial w$$ 和  $$\partial C/\partial b$$。对公式(87)进行求偏导数得：

![](images/100.png)

$$\partial C_0/\partial w$$ 和  $$\partial C_0/\partial b$$ 可以通过反向传播进行计算，和上一章中的那样。所以我们看到其实计算规范化的代价函数的梯度是很简单的：仅仅需要反向传播，然后加上 $$\frac{\lambda}{n} w$$ 得到所有权重的偏导数。而偏差的偏导数就不要变化，所以梯度下降学习规则不会发生变化：

![](images/101.png)

权重的学习规则就变成：

![](images/102.png)

这其实和通常的梯度下降学习规则相同欧诺个，除了乘了 $$1-\frac{\eta\lambda}{n}$$ 因子。这里就是**权重下降**的来源。粗看，这样会导致权重会不断下降到 $$0$$。但是实际不是这样的，因为如果在原始代价函数中造成下降的话其他的项可能会让权重增加。

好的，这就是梯度下降工作的原理。那么随机梯度下降呢？正如在没有规范化的随机梯度下降中，我们可以通过平均 minibatch 中 $$m$$ 个训练样本来估计 $$\partial C_0/\partial w$$。因此，为了随机梯度下降的规范化学习规则就变成（参考 方程(20)）

![](images/103.png)

其中后面一项是对 minibatch 中的训练样本 $$x$$ 进行求和，而 $$C_x$$ 是对每个训练样本的（无规范化的）代价。这其实和之前通常的随机梯度下降的规则是一样的，除了有一个权重下降的因子 $$1-\frac{\eta\lambda}{n}$$。最后，为了完备性，我给出偏差的规范化的学习规则。这当然是和我们之前的非规范化的情形一致了（参考公式(32)）

![](images/104.png)

这里也是对minibatch 中的训练样本 $$x$$ 进行求和的。

让我们看看规范化给网络带来的性能提升吧。这里还会使用有 $$30$$ 个隐藏神经元、minibatch 为 $$10$$，学习率为 $$0.5$$，使用交叉熵的神经网络。然而，这次我们会使用规范化参数为 $$\lambda = 0.1$$。注意在代码中，我们使用的变量名字为 `lmbda`，这是因为在 Python 中 `lambda` 是关键字，尤其特定的作用。我也会使用 `test_data`，而不是 `validation_data`。**不过严格地讲，我们应当使用 `validation_data`的，因为前面已经讲过了。**这里我这样做，是因为这会让结果和非规范化的结果对比起来效果更加直接。你可以轻松地调整为 `validation_data`，你会发现有相似的结果。

```python
>>> import mnist_loader 
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper() 
>>> import network2 
>>> net = network2.Network([784, 30, 10], cost=network2.CrossEntropyCost)
>>> net.large_weight_initializer()
>>> net.SGD(training_data[:1000], 400, 10, 0.5,
... evaluation_data=test_data, lmbda = 0.1,
... monitor_evaluation_cost=True, monitor_evaluation_accuracy=True,
... monitor_training_cost=True, monitor_training_accuracy=True)
```

训练集上的代价函数持续下降，和前面无规范化的情况一样的规律：

![](images/105.png)

但是这里测试集上的准确度是随着回合次数持续增加的：

![](images/106.png)

显然，规范化的使用能够解决过匹配的问题。而且，准确度相当搞了，最高处达到了 87.1%，相较于之前的 82.27%。因此，我们几乎可以确信持续训练会有更加好的结果。实验起来，规范化让网络具有更好的泛化能力，显著地减轻了过匹配的效果。

如果我们换成全部的训练数据进行训练呢？当然，我们之前已经看到过匹配在大规模的数据上其实不是那么明显了。那规范化能不能起到相应的作用呢？保持超参数和之前一样。不过我们这里需要改变规范化参数。原因在于训练数据的大小已经从 $$n=1,000$$ 改成了 $$n=50,000$$，这个会改变权重下降因子 $$1-\frac{\eta\lambda}{n}$$。如果我们持续使用 $$\lambda = 0.1$$ 就会产生很小的权重下降，因此就将规范化的效果降低很多。我们通过将 $$\lambda = 5.0$$ 来补偿这种下降。

好了，来训练网络，重新初始化权重：

```python
>>> net.large_weight_initializer()
>>> net.SGD(training_data, 30, 10, 0.5,
... evaluation_data=test_data, lmbda = 5.0,
... monitor_evaluation_accuracy=True, monitor_training_accuracy=True)
```

我们得到：

![](images/107.png)

这个结果很不错。第一，我们在测试集上的分类准确度在使用规范化后有了提升，从 95.49% 到 96.49%。这是个很大的进步。第二，我们可以看到在训练数据和测试数据上的结果之间的差距也更小了。这仍然是一个大的差距，不过我们已经显著得到了本质上的降低过匹配的进步。

最后，我们看看在使用 $$100$$ 个隐藏元和规范化参数为 $$\lambda = 5.0$$ 相应的测试分类准确度。我不会给出详细分析，就为了有趣，来看看我们使用一些技巧（交叉熵函数和 $$L2$$ 规范化）能够达到多高的准确度。

```python
>>> net = network2.Network([784, 100, 10], cost=network2.CrossEntropyCost)
>>> net.large_weight_initializer()
>>> net.SGD(training_data, 30, 10, 0.5, lmbda=5.0,
... evaluation_data=validation_data,
... monitor_evaluation_accuracy=True)
```

最终在验证集上的准确度达到了 97.92%。这是比 $$30$$ 个隐藏元的较大飞跃。实际上，稍微改变一点，$$60$$ 回合 $$\eta=0.1$$ 和 $$\lambda = 5.0$$。我们就突破了 98%，达到了 98.04% 的分类准确度。对于 $$152$$ 行代码这个效果还真不错！

我们讨论了作为一种减轻过匹配和提高分类准确度的方式的规范化技术。实际上，这不是仅有的好处。实践表明，在使用不同的（随机）权重初始化进行多次 MNIST 网络训练的时候，我发现无规范化的网络会偶然被限制住，明显困在了代价函数的局部最优值处。结果就是不同的运行会给出相差很大的结果。对比看来，规范化的网络能够提供更容易复制的结果。

为何会这样子？从经验上看，如果代价函数是无规范化的，那么权重向量的长度可能会增长，而其他的东西都保持一样。随着时间的推移，这个会导致权重向量变得非常大。所以会使得权重向困在差不多方向上，因为由于梯度下降的改变当长度很大的时候仅仅会在那个方向发生微小的变化。我相信这个现象让学习算法更难有效地探索权重空间，最终导致很难找到代价函数的最优值。

### 为何规范化可以帮助减轻过匹配

我们已经看到了规范化在实践中能够减少过匹配了。这是令人振奋的，不过，这背后的原因还不得而知！通常的说法是：小的权重在某种程度上，意味着更低的复杂性，也就给出了一种更简单却更强大的数据解释，因此应该优先选择。这虽然很简短，不过暗藏了一些可能看起来会令人困惑的因素。让我们将这个解释细化，认真地研究一下。现在给一个简单的数据集，我们为其建立模型：

![](images/108.png)

这里我们其实在研究某种真实的现象，$$x$$ 和 $$y$$ 表示真实的数据。我们的目标是训练一个模型来预测 $$y$$ 关于 $$x$$ 的函数。我们可以使用神经网络来构建这个模型，但是我们先来个简单的：用一个多项式来拟合数据。这样做的原因其实是多项式相比神经网络能够让事情变得更加清楚。一旦我们理解了多项式的场景，对于神经网络可以如法炮制。现在，图中有十个点，我们就可以找到唯一的 $$9$$ 阶多项式 $$y=a_0x^9 + a_1x^8 + ... + a_9$$ 来完全拟合数据。下面是多项式的图像：
> I won't show the coefficients explicitly, although they are easy to find using a routine such as Numpy's polyfit. You can view the exact form of the polynomial in the [source code for the  graph](http://neuralnetworksanddeeplearning.com/js/polynomial_model.js) if you're curious. It's the function p(x) defined starting on line 14 of the program which produces the graph.

![](images/109.png)

这给出了一个完美的拟合。但是我们同样也能够使用线性模型 $$y=2x$$ 得到一个好的拟合效果：

![](images/110.png)

哪个是更好的模型？哪个更可能是真的？还有哪个模型更可能泛化到其他的拥有同样现象的样本上？

这些都是很难回答的问题。实际上，我们如果没有关于现象背后的信息的话，并不能确定给出上面任何一个问题的答案。但是让我们考虑两种可能的情况：（1）$$9$$ 阶多项式实际上是完全描述了真实情况的模型，最终它能够很好地泛化；（2）正确的模型是 $$y=2x$$，但是存在着由于测量误差导致的额外的噪声，使得模型不能够准确拟合。

先验假设无法说出哪个是正确的（或者，如果还有其他的情况出现）。逻辑上讲，这些都可能出现。并且这不是易见的差异。在给出的数据上，两个模型的表现其实是差不多的。但是假设我们想要预测对应于某个超过了图中所有的 $$x$$ 的 $$y$$ 的值，在两个模型给出的结果之间肯定有一个极大的差距，因为 $$9$$ 阶多项式模型肯定会被 $$x^9$$ 主导，而线性模型只是线性的增长。

在科学中，一种观点是我们除非不得已应该追随更简单的解释。当我们找到一个简单模型似乎能够解释很多数据样本的时候，我们都会激动地认为发现了规律！总之，这看起来简单的解决仅仅会是偶然出现的不大可能。我们怀疑模型必须表达出某些关于现象的内在的真理。如上面的例子，线性模型加噪声肯定比多项式更加可能。所以如果简单性是偶然出现的话就很令人诧异。因此我们会认为线性模型加噪声表达除了一些潜在的真理。从这个角度看，多项式模型仅仅是学习到了局部噪声的影响效果。所以尽管多是对于这些特定的数据点表现得很好。模型最终会在未知数据上的泛化上出现问题，所以噪声线性模型具有更强大的预测能力。

让我们从这个观点来看神经网络。假设神经网络大多数有很小的权重，这最可能出现在规范化的网络中。更小的权重意味着网络的行为不会因为我们随便改变了一个输入而改变太大。这会让规范化网络学习局部噪声的影响更加困难。将它看做是一种让单个的证据不会影响网络输出太多的方式。相对的，规范化网络学习去对整个训练集中经常出现的证据进行反应。对比看，大权重的网络可能会因为输入的微小改变而产生比较大的行为改变。所以一个无规范化的网络可以使用大的权重来学习包含训练数据中的噪声的大量信息的复杂模型。简言之，规范化网络受限于根据训练数据中常见的模式来构造相对简单的模型，而能够抵抗训练数据中的噪声的特性影响。我们的想法就是这可以让我们的网络对看到的现象进行真实的学习，并能够根据已经学到的知识更好地进行泛化。

所以，倾向于更简单的解释的想法其实会让我们觉得紧张。人们有时候将这个想法称为“奥卡姆剃刀原则”，然后就会热情地将其当成某种科学原理来应用这个法则。但是，这就不是一个一般的科学原理。也没有任何先验的逻辑原因来说明简单的解释就比更为负责的解释要好。实际上，有时候更加复杂的解释其实是正确的。

让我介绍两个说明复杂正确的例子。在 $$1940$$ 年代，物理学家 Marcel Schein 发布了一个发现新粒子的声明。而他工作的公司，GE，非常欢喜，就广泛地推广这个发现。但是物理学及 Hans Bethe 就有怀疑。Bethe 访问了 Schein，看着那些展示 Schein 的新粒子的轨迹的盘子。但是在每个 plate 上，Bethe 都发现了某个说明数据需要被去除的问题。最后 Schein 展示给 Bethe 一个看起来很好的 plate。Bethe 说这可能就是一个统计上的侥幸。Schein 说，“使得，但是这个可能就是统计学，甚至是根据你自己的公式，也就是 $$1/5$$ 的概率。” Bethe 说：“但我们已经看过了这 $$5$$ 个plate 了”。最终，Schein 说：“但是在我的plate中，每个好的plate，每个好的场景，你使用了不同的理论（说它们是新的粒子）进行解释，而我只有一种假设来解释所有的 plate。” Bethe 回答说，“在你和我的解释之间的唯一差别就是你的是错的，而我所有的观点是正确的。你单一的解释是错误的，我的多重解释所有都是正确的。”后续的工作证实了，Bethe 的想法是正确的而 Schein 粒子不再正确。
> **注意**：这一段翻译得很不好，请参考原文

第二个例子，在 $$1859$$ 年，天文学家 Urbain Le Verrier 观察到水星并没有按照牛顿万有引力给出的轨迹进行运转。与牛顿力学只有很小的偏差，那时候一些解释就是牛顿力学需要一些微小的改动了。在 $$1916$$ 年爱因斯坦证明偏差用他的广义相对论可以解释得更好，这是一种和牛顿重力体系相差很大的理论，基于更复杂的数学。尽管引入了更多的复杂性，现如今爱因斯坦的解释其实是正确的，而牛顿力学即使加入一些调整，仍旧是错误的。这部分因为我们知道爱因斯坦的理论不仅仅解释了这个问题，还有很多其他牛顿力学无法解释的问题也能够完美解释。另外，令人印象深刻的是，爱因斯坦的理论准确地给出了一些牛顿力学没能够预测到的显现。但是这些令人印象深刻的现象其实在先前的时代是观测不到的。如果一个人仅仅通过简单性作为判断合理模型的基础，那么一些牛顿力学的改进理论可能会看起来更加合理一些。

从这些故事中可以读出三点。第一，确定两种解释中哪个“更加简单”其实是一件相当微妙的工作。第二，即使我们可以做出这样一个判断，简单性也是一个使用时需要相当小心的指导！第三，对模型真正的测试不是简单性，而是它在新场景中对新的活动中的预测能力。

所以，我们应当时时记住这一点，规范化的神经网络常常能够比非规范化的泛化能力更强，这只是一种实验事实（empirical fact）。所以，本书剩下的内容，我们也会频繁地使用规范化技术。我已经在上面讲过了为何现在还没有一个人能够发展出一整套具有说服力的关于规范化可以帮助网络泛化的理论解释。实际上，研究者们不断地在写自己尝试不同的规范化方法，然后看看哪种表现更好，尝试理解为何不同的观点表现的更好。所以你可以将规范化看做某种任意整合的技术。尽管其效果不错，但我们并没有一套完整的关于所发生情况的理解，仅仅是一些不完备的启发式规则或者经验。

这里也有更深的问题，这个问题也是有关科学的关键问题——我们如何泛化。规范化能够给我们一种计算上的魔力帮助神经网络更好地泛化，但是并不会带来原理上理解的指导，甚至不会告诉我们什么样的观点才是最好的。
> 这个问题要追溯到[归纳问题](http://en.wikipedia.org/wiki/Problem_of_induction)，最先由苏格兰哲学家大卫 休谟在 ["An Enquiry Concerning Human Understanding"](http://www.gutenberg.org/ebooks/9662)(1748) 中提出。在现代机器学习领域中归纳问题被 David Wolpert 和 William Macready 描述成[无免费午餐定理](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?tp=&arnumber=585893)。

这实在是令人困扰，因为在日常生活中，我们人类在泛化上表现很好。给一个儿童几幅大象的图片，他就能快速地学会认识其他的大象。当然，他们偶尔也会搞错，很可能将一只犀牛误认为大象，但是一般说来，这个过程会相当准确。所以我们有个系统——人的大脑——拥有超大量的自由变量。在受到仅仅少量的训练图像后，系统学会了在其他图像的推广。某种程度上，我们的大脑的规范化做得特别好！怎么做的？现在还不得而知。我期望若干年后，我们能够发展出更加强大的技术来规范化神经网络，最终这些技术会让神经网络甚至在小的训练集上也能够学到强大的泛化能力。

实际上，我们的网络已经比我们预先期望的要好一些了。拥有 $$100$$ 个隐藏元的网络会有接近 $$80,000$$ 个参数。我们的训练集仅仅有 $$50,000$$ 幅图像。这好像是用一个 $$80,000$$ 阶的多项式来拟合 $$50,000$$ 个数据点。我们的网络肯定会过匹配得很严重。但是，这样的网络实际上却泛化得很好。为什么？这一点并没有很好滴理解。这里有个猜想：梯度下降学习的动态有一种自规范化的效应。这真是一个意料之外的巧合，但也带来了对于这种现象本质无知的不安。不过，我们还是会在后面依照这种实践的观点来应用规范化技术的。
神经网络也是由于这点表现才更好一些。

现在我们回到前面留下来的一个细节：L2 规范化没有限制偏差，以此作为本节的结论。当然了，对规范化的过程稍作调整就可以对偏差进行规范了。实践看来，做出这样的调整并不会对结果改变太多，所以，在某种程度上，对不对偏差进行规范化其实就是一种习惯了。然而，需要注意的是，有一个大的偏差并不会像大的权重那样会让神经元对输入太过敏感。所以我们不需要对大的偏差所带来的学习训练数据的噪声太过担心。同时，允许大的偏差能够让网络更加灵活——因为，大的偏差让神经元更加容易饱和，这有时候是我们所要达到的效果。所以，我们通常不会对偏差进行规范化。

### 规范化的其他技术

除了 L2 外还有很多规范化技术。实际上，正是由于数量众多，我这里也不回将所有的都列举出来。在本节，我简要地给出三种减轻过匹配的其他的方法：L1 规范化、dropout 和人工增加训练样本。我们不会像上面介绍得那么深入。其实，目的只是想让读者熟悉这些主要的思想，然后来体会一下规范化技术的多样性。

**L1 规范化**：这个方法其实是在代价函数上加上一个权重绝对值的和：

![](images/111.png)

直觉上看，这和 L2 规范化相似，惩罚大的权重，倾向于让网络的权重变小。当然，L1 规范化和 L2 规范化并不相同，所以我们不应该期望 L1 规范化是进行同样的行为。让我们来看看试着理解使用 L1 规范化和 L2 规范化所不同的地方。

首先，我们会研究一下代价函数的偏导数。对(95)求导我们有：

![](images/112.png)

其中 $$sgn(w)$$ 就是 $$w$$ 的正负号。使用这个表达式，我们可以轻易地对反向传播进行修改从而使用基于 L1 规范化的随机梯度下降进行学习。对 L1 规范化的网络进行更新的规则就是

![](images/113.png)

其中和往常一样，我们可以用 minibatch 的均值来估计 $$\partial C_0/\partial w$$。对比公式(93)的 L2 规范化，

![](images/114.png)

在两种情形下，规范化的效果就是缩小权重。这和我们想要让权重不会太大的直觉目标相符。在 L1 规范化中，权重按照一个接近 $$0$$ 的常量进行缩小。在 L2 规范化中，权重同按照一个和 $$w$$ 成比例的量进行缩小的。所以，当一个特定的权重绝对值 $$|w|$$很大时，L1 规范化缩小得远比 L2 规范化要小得多。而一个特定的权重绝对值 $$|w|$$很小时，L1 规范化权值要比 L2 规范化缩小得更大。最终的结果就是：L1 规范化倾向于聚集网络的权重在相对少量的高重要度连接上，而其他权重就会被驱使向 $$0$$ 接近。

我在上面的讨论中其实忽略了一个问题——在 $$w=0$$ 的时候，偏导数 $$\partial C/\partial w$$ 未定义。原因在于函数 $$|w|$$ 在 $$w=0$$ 时有个直角，事实上，导数是不存在的。不过也没有关系。我们下面要做的就是应用无规范化的通常的梯度下降的规则在 $$w=0$$ 处。这应该不会有什么问题，直觉上看，规范化的效果就是缩小权重，显然，不能对一个已经是 $$0$$ 的权重进行缩小了。更准确地说，我们将会使用方程(96)(97)并约定 $$sgn(0)=0$$。这样就给出了一种紧致的规则来进行采用 L1 规范化的随机梯度下降学习。

**Dropout **：Dropout 是一种相当激进的技术。和 L1、L2 规范化不同，dropout 并不依赖对代价函数的变更。而是，在 dropout 中，我们改变了网络本身。让我在给出为何工作的原理之前描述一下 dropout 基本的工作机制和所得到的结果。

假设我们尝试训练一个网络：

![](images/115.png)

特别地，假设我们有一个训练数据 $$x$$ 和 对应的目标输出 $$y$$。通常我们会通过在网络中前向传播 $$x$$ ，然后进行反向传播来确定对梯度的共现。使用 dropout，这个过程就改了。我们会从随机（临时）地删除网络中的一半的神经元开始，让输入层和输出层的神经元保持不变。在此之后，我们会得到最终的网络。注意那些被 dropout 的神经元，即那些临时性删除的神经元，用虚圈表示在途中：

![](images/116.png)

我们前向传播输入，通过修改后的网络，然后反向传播结果，同样通过这个修改后的网络。在 minibatch 的若干样本上进行这些步骤后，我们对那些权重和偏差进行更新。然后重复这个过程，首先重置 dropout 的神经元，然后选择新的随机隐藏元的子集进行删除，估计对一个不同的minibatch的梯度，然后更新权重和偏差。

通过不断地重复，我们的网络会学到一个权重和偏差的集合。当然，这些权重和偏差也是在一般的隐藏元被丢弃的情形下学到的。当我们实际运行整个网络时，是指两倍的隐藏元将会被激活。为了补偿这个，我们将从隐藏元出去的权重减半了。

这个 dropout 过程可能看起来奇怪和ad hoc。为什么我们期待这样的方法能够进行规范化呢？为了解释所发生的事，我希望你停下来想一下没有 dropout 的训练方式。特别地，想象一下我们训练几个不同的神经网络，使用的同一个训练数据。当然，网络可能不是从同一初始状态开始的，最终的结果也会有一些差异。出现这种情况时，我们可以使用一些平均或者投票的方式来确定接受哪个输出。例如，如果我们训练了五个网络，其中三个被分类当做是 $$3$$，那很可能它就是 $$3$$。另外两个可能就犯了错误。这种平均的方式通常是一种强大（尽管代价昂贵）的方式来减轻过匹配。原因在于不同的网络可能会以不同的方式过匹配，平均法可能会帮助我们消除那样的过匹配。

那么这和 dropout 有什么关系呢？启发式地看，当我们丢掉不同的神经元集合时，有点像我们在训练不同的神经网络。所以，dropout 过程就如同大量不同网络的效果的平均那样。不同的网络以不同的方式过匹配了，所以，dropout 的网络会减轻过匹配。

一个相关的启发式解释在早期使用这项技术的论文中曾经给出：“因为神经元不能依赖其他神经元特定的存在，这个技术其实减少了复杂的互适应的神经元。所以，强制要学习那些在神经元的不同随机子集中更加健壮的特征。”换言之，如果我们就爱那个神经网络看做一个进行预测的模型的话，我们就可以将 dropout 看做是一种确保模型对于证据丢失健壮的方式。这样看来，dropout 和 L1、L2 规范化也是有相似之处的，这也倾向于更小的权重，最后让网络对丢失个体连接的场景更加健壮。

当然，真正衡量 dropout 的方式在提升神经网络性能上应用得相当成功。原始论文介绍了用来解决很多不同问题的技术。对我们来说，特别感兴趣的是他们应用 dropout 在 MNIST 数字分类上，使用了一个和我们之前介绍的那种初级的前向神经网络。这篇文章关注到最好的结果是在测试集上去得到 98.4% 的准确度。他们使用dropout 和 L2 规范化的组合将其提高到了 98.7%。类似重要的结果在其他不同的任务上也取得了一定的成效。dropout 已经在过匹配问题尤其突出的训练大规模深度网络中。

**人工扩展训练数据**：我们前面看到了 MNIST 分类准确度在我们使用 $$1,000$$ 幅训练图像时候下降到了 $$80$$ 年代的准确度。这种情况并不奇怪，因为更少的训练数据意味着我们的网络所接触到较少的人类手写的数字中的变化。让我们训练 $$30$$ 个隐藏元的网络，使用不同的训练数据集，来看看性能的变化情况。我们使用 minibatch 大小为 $$10$$，学习率是 $$\eta=0.5$$，规范化参数是 $$\lambda=5.0$$，交叉熵代价函数。我们在全部训练数据集合上训练 30 个回合，然后会随着训练数据量的下降而成比例变化回合数。为了确保权重下降因子在训练数据集上相同，我们会在全部训练集上使用规范化参数为 $$\lambda = 5.0$$，然后在使用更小的训练集的时候成比例地下降 $$\lambda$$ 值。
> This and the next two graph are produced with the program[more_data.py](https://github.com/mnielsen/neural-networks-and-deep-learning/blob/master/fig/more_data.py).

![](images/117.png)

如你所见，分类准确度在使用更多的训练数据时提升了很大。根据这个趋势的话，提升会随着更多的数据而不断增加。当然，在训练的后期我们看到学习过程已经进入了饱和状态。然而，如果我们使用对数作为横坐标的话，可以重画此图如下：

![](images/118.png)

这看起来到了后面结束的地方，增加仍旧明显。这表明如果我们使用大量更多的训练数据——不妨设百万或者十亿级的手写样本——那么，我们可能会得到更好的性能，即使是用这样的简单网络。

获取更多的训练样本其实是很重要的想法。不幸的是，这个方法代价很大，在实践中常常是很难达到的。不过，还有一种方法能够获得类似的效果，那就是进行人工的样本扩展。假设我们使用一个 $$5$$ 的训练样本，

![](images/119.png)

将其进行旋转，比如说 $$15$$°：

![](images/120.png)

这还是会被设别为同样的数字的。但是在像素层级这和任何一幅在 MNIST 训练数据中的图像都不相同。所以将这样的样本加入到训练数据中是很可能帮助我们学习有关手写数字更多知识的方法。而且，显然，我们不会就只对这幅图进行人工的改造。我们可以在所有的 MNIST 训练样本上通过和多小的旋转扩展训练数据，然后使用扩展后的训练数据来提升我们网络的性能。

这个想法非常强大并且已经被广发应用了。让我们看看一些在 MNIST 上使用了类似的方法进行研究成果。其中一种他们考虑的网络结构其实和我们已经使用过的类似——一个拥有 800 个隐藏元的前驱神经网络，使用了交叉熵代价函数。在标准的 MNIST 训练数据上运行这个网络，得到了 98.4% 的分类准确度，其中使用了不只是旋转还包括转换和扭曲。通过在这个扩展后的数据集上的训练，他们提升到了 98.9% 的准确度。然后还在“弹性扭曲（elastic distortion）”的数据上进行了实验，这是一种特殊的为了模仿手部肌肉的随机抖动的图像扭曲方法。通过使用弹性扭曲扩展的数据，他们最终达到了 99.3% 的分类准确度。他们通过展示训练数据的所有类型的变体来扩展了网络的经验。
>[Best Practices for Convolutional Neural Networks Applied to Visual Document Analysis](http://dx.doi.org/10.1109/ICDAR.2003.1227801), by Patrice Simard, Dave Steinkraus, and John Platt (2003).

这个想法的变体也可以用在提升手写数字识别之外不同学习任务上的性能。一般就是通过应用反映真实世界变化的操作来扩展训练数据。找到这些方法其实并不困难。例如，你要构建一个神经网络来进行语音识别。我们人类甚至可以在有背景噪声的情况下识别语音。所以你可以通过增加背景噪声来扩展你的训练数据。我们同样能够对其进行加速和减速来获得相应的扩展数据。所以这是另外的一些扩展训练数据的方法。这些技术并不总是有用——例如，其实与其在数据中加入噪声，倒不如先对数据进行噪声的清理，这样可能更加有效。当然，记住可以进行数据的扩展，寻求应用的机会还是相当有价值的一件事。

### 练习

* 正如上面讨论的那样，一种扩展 MNIST 训练数据的方式是用一些微小的旋转。如果我们允许过大的旋转，则会出现什么状况呢？

**大数据的旁白和对分类准确度的影响**：让我们看看神经网络准确度随着训练集大小变化的情况：

![](images/121.png)

假设，我们使用别的什么方法来进行分类。例如，我们使用 SVM。正如第一章介绍的那样，不要担心你熟不熟悉 SVM，我们不进行深入的讨论。下面是 SVM 模型的准确度随着训练数据集的大小变化的情况：

![](images/122.png)

可能第一件让你吃惊的是神经网络在每个训练规模下性能都超过了 SVM。这很好，尽管你对细节和原理可能不太了解，因为我们只是直接从 scikit-learn 中直接调用了这个方法，而对神经网络已经深入讲解了很多。更加微妙和有趣的现象其实是如果我们训练 SVM 使用 $$50,000$$ 幅图像，实际上 SVM 已经能够超过我们使用 $$5,000$$ 幅图像的准确度。换言之，更多的训练数据可以补偿不同的机器学习算法的差距。

还有更加有趣的现象也出现了。假设我们试着用两种机器学习算法去解决问题，算法 $$A$$ 和算法 $$B$$。有时候出现，算法 $$A$$ 在一个训练集合上超过 算法 $$B$$，却在另一个训练集上弱于算法 $$B$$。上面我们并没有看到这个情况——因为这要求两幅图有交叉的点——这里并没有。对“算法 A 是不是要比算法 $$B$$ 好？”正确的反应应该是“你在使用什么训练集合？”

在进行开发时或者在读研究论文时，这都是需要记住的事情。很多论文聚焦在寻找新的技术来给出标准数据集上更好的性能。“我们的超赞的技术在标准测试集 $$Y$$ 上给出了百分之 $$X$$ 的性能提升。”这是通常的研究声明。这样的声明通常比较有趣，不过也必须被理解为仅仅在特定的训练数据机上的应用效果。那些给出基准数据集的人们会拥有更大的研究经费支持，这样能够获得更好的训练数据。所以，很可能他们由于超赞的技术的性能提升其实在更大的数据集合上就丧失了。**换言之，人们标榜的提升可能就是历史的偶然。**所以需要记住的特别是在实际应用中，我们想要的是更好的算法和更好的训练数据。寻找更好的算法很重，不过需要确保你在此过程中，没有放弃对更多更好的数据的追求。

### 问题

* **研究问题**：我们的机器学习算法在非常大的数据集上如何进行？对任何给定的算法，其实去定义一个随着训练数据规模变化的渐近的性能是一种很自然的尝试。一种简单粗暴的方法就是简单地进行上面图中的趋势分析，然后将图像推进到无穷大。而对此想法的反驳是曲线本身会给出不同的渐近性能。你能够找到拟合某些特定类别曲线的理论上的验证方法吗？如果可以，比较不同的机器学习算法的渐近性能。

**总结**：我们现在已经介绍完了过匹配和规范化。当然，我们重回这个问题。正如我们前面讲过的那样，尤其是计算机越来越强大，我们有训练更大的网络的能力时。过匹配是神经网络中一个主要的问题。我们有迫切的愿望来开发出强大的规范化技术来减轻过匹配，所以，这也是当前研究的极其热门的方向之一。
