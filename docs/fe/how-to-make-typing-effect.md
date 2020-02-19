# 如何用CSS实现打字的效果
如果你看过我的首页，就会发现我的首页介绍是一段打字效果实现的，那么如何实现呢？接下来来探讨一下

## STEP-1 Animation 动画 {docsify-ignore}
以下列div为例
```
<div class="typing">This is a article share you how to make typing effect.</div>
```
实现的核心思路是：**让容器的宽度不断地增大**
根据这个想法，我们可以如此实现：
```
@keyframes typing{
    from{
        width: 0;
    }
}
.typing{
    width: 54em;
    animation: typing 10s;
}
```
加上之后会发现，在动画刚开始的时候，字体会因为宽度不够被挤压到下方，因此我们加一个 ```overflow:hidden;``` 试试，发现仍然没有效果，这是因为我们用了一个 div，默认没有设置高度，所以它的 height 默认是 auto，也就是在动态适应内容，既然如此，我们可以给他加一个```white-space:nowrap```，表示不允许换行，加上之后似乎效果好了很多，但是如果我们把动画时间适当延长会发现一个问题：每一次出现的字母并不是完整地出现的，而是有可能会出来一半，然后再出来另外一半，这样给人看上去的效果就不是很逼真。不过神奇的 animation 还有另外一个属性：```steps（number)```，它表示你要让动画分成多少个步骤来完成，如果没有在 keyframes 指定每一步的内容，那么就会均匀地将时间分摊给每一个 step，举个🌰，我们这里动画时间是10s，我加上以下属性：
```animation: typing 10s steps(20)```
表示 10s 会被分成 20 个步骤，每个步骤占 0.5s。借助这个属性，我们可以根据我们的字数来设置：
```animation: typing 10s steps(54)```
我们发现虽然时间被分割了，但是好像并没有达到我们的效果，我们企图让他没 10/54 秒渲染一次，但是长度并没有达到我们的效果。

## STEP-2 font 字体 {docsify-ignore}
经过观察发现，在示例中，每个字母的宽度都是不一样的，这也就导致了，在计算动画的 width 偏移量时，会出现移动不均匀的情况。因此，我们需要将每个字母的宽度设置成一样，刚好 css 里面有一类字体叫做等宽字体，也就是每一个字符的宽度都是一样的，这类字体有例如：

```Consolas, Monaco, monospace```

我们加上看看效果：

``` font-family:Consolas,Monaco,monospace; ```

似乎还没是没有达到我们想要的效果，原因在哪里呢？

哈哈哈，其实原因在于我们用的宽度单位，纳尼？我们不是已经用了em做单位了嘛？

其实em只是继承父节点的字体宽度大小，而父节点的字体并不一定是等宽字体，因此我们需要设置一下当前节点，在等宽字体下的宽度。而恰好css也有一个比较不常用的单位：ch，它就表示字符0的宽度，而字符0是一个神奇的字符，不论在任何字体下，它的宽度都是固定的。因此，我们可以使用 ch 来代替 em 来作为节点的宽度单位。

```width: 54ch;```

看上去一切很完美！

## STEP-3 border 光标 {docsify-ignore}
等等，我们好像忘了最关键的打字光标。
这个实现就比较简单了，我们可以直接用容器的右边框来作为它的光标。
```border-right: 1px solid;```
这里还差一个问题，我们知道，在打字暂停或者结束时，光标是会闪烁的，因此我们给它加一个动画效果
```css
@keyframes flashing{
    50%{
        border-color: transparent;
    }
}

.typing{
    animation: typing 10s steps(54),flashing 1s 10s steps(1) infinite;
}
```

看上去效果好多了，这里我们做了10s的延时，体现出来正在打字时，光标不会闪烁的效果，等到打字结束时才开始闪烁。

## STEP-4 Javascript 自适应文本字数 {docsify-ignore}
在通常的场景下，我们文案字数是经常改变的，没有办法在css里面指定这个宽度，如此情况我们该如何实现呢。
这里只能通过Javascript来实现了，我们先去掉样式里面写死的 width 以及 steps ，然后用JS来计算字符个数，并且根据字符个数来得到对应的值：
```javascript
var typingArr = Array.prototype.slice.call(document.querySelectorAll('.typing'));

typingArr.forEach(function (el) {
    var len = el.textContent.length;
    var style = el.style;
    style.width = len + 'ch';
    style.animationTimingFunction = 'steps('+len+'),steps(1)';
});
```

大功告成！看上去有模有样，完整的代码如下：

HTML:
```html
<div class="typing">This is a article share you how to make typing effect.</div>
```

CSS:
```css
html{
  background: #000;
  color: green;
}
@keyframes typing{
    from{
        width: 0;
    }
}
@keyframes flashing{
  50%{
    border-color: transparent;
  }
}
.typing{
    font-family: Consolas,Monaco,monospace;
    white-space:nowrap;
    overflow:hidden;
    border-right: 1px solid;
    animation: typing 10s,
               flashing 1s 10s infinite;
}
```

JavaScript:
```javascript
var typingArr = Array.prototype.slice.call(document.querySelectorAll('.typing'));

typingArr.forEach(function (el) {
    var len = el.textContent.length;
    var style = el.style;
    style.width = len + 'ch';
    style.animationTimingFunction = 'steps('+len+'),steps(1)';
});
```

效果如下
<iframe height="464" style="width: 100%;" scrolling="no" title="oORrEx" src="https://codepen.io/april0421/embed/oORrEx?height=464&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/april0421/pen/oORrEx'>oORrEx</a> by april
  (<a href='https://codepen.io/april0421'>@april0421</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>