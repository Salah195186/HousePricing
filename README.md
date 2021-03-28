
<!DOCTYPE html>
<html>

  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Hacker's guide to Neural Networks</title>
    <meta name="viewport" content="width=device-width">
    <meta name="description" content="Musings of a Computer Scientist.">
    <link rel="canonical" href="http://karpathy.github.io/neuralnets/">
    <link href="/feed.xml" type="application/atom+xml" rel="alternate" title="Andrej Karpathy blog posts" />

    <!-- Custom CSS -->
    <link rel="stylesheet" href="/css/main.css">

    <!-- Google Analytics -->
    <script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
      ga('create', 'UA-3698471-23', 'auto');
      ga('send', 'pageview');
    </script>

</head>


    <body>

    <header class="site-header">

  <div class="wrap">

    <div style="float:left; margin-top:10px; margin-right:10px;">
    <a href="/feed.xml">
      <img src="/assets/rssicon.svg" width="40">
    </a>
    </div>

    <a class="site-title" href="/">Andrej Karpathy blog</a>
    
    <nav class="site-nav">
      <a href="#" class="menu-icon">
        <svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
           viewBox="0 0 18 15" enable-background="new 0 0 18 15" xml:space="preserve">
          <path fill="#505050" d="M18,1.484c0,0.82-0.665,1.484-1.484,1.484H1.484C0.665,2.969,0,2.304,0,1.484l0,0C0,0.665,0.665,0,1.484,0
            h15.031C17.335,0,18,0.665,18,1.484L18,1.484z"/>
          <path fill="#505050" d="M18,7.516C18,8.335,17.335,9,16.516,9H1.484C0.665,9,0,8.335,0,7.516l0,0c0-0.82,0.665-1.484,1.484-1.484
            h15.031C17.335,6.031,18,6.696,18,7.516L18,7.516z"/>
          <path fill="#505050" d="M18,13.516C18,14.335,17.335,15,16.516,15H1.484C0.665,15,0,14.335,0,13.516l0,0
            c0-0.82,0.665-1.484,1.484-1.484h15.031C17.335,12.031,18,12.696,18,13.516L18,13.516z"/>
        </svg>
      </a>
      <div class="trigger">
        
          <a class="page-link" href="/about/">About</a>
        
          
        
          
        
          
        
          
        
      </div>
    </nav>
  </div>

</header>


    <div class="page-content">
      <div class="wrap">
      <div class="post">

  <header class="post-header">
    <h1>Hacker's guide to Neural Networks</h1>
  </header>

  <article class="post-content">
  <p><strong>Note: this is now a very old tutorial that I’m leaving up, but I don’t believe should be referenced or used. Better materials include CS231n course lectures, slides, and notes, or the Deep Learning book</strong>.</p>

<p>Hi there, I’m a <a href="http://cs.stanford.edu/people/karpathy/">CS PhD student at Stanford</a>. I’ve worked on Deep Learning for a few years as part of my research and among several of my related pet projects is <a href="http://convnetjs.com">ConvNetJS</a> - a Javascript library for training Neural Networks. Javascript allows one to nicely visualize what’s going on and to play around with the various hyperparameter settings, but I still regularly hear from people who ask for a more thorough treatment of the topic. This article (which I plan to slowly expand out to lengths of a few book chapters) is my humble attempt. It’s on web instead of PDF because all books should be, and eventually it will hopefully include animations/demos etc.</p>

<p>My personal experience with Neural Networks is that everything became much clearer when I started ignoring full-page, dense derivations of backpropagation equations and just started writing code. Thus, this tutorial will contain <strong>very little math</strong> (I don’t believe it is necessary and it can sometimes even obfuscate simple concepts). Since my background is in Computer Science and Physics, I will instead develop the topic from what I refer to as <strong>hackers’s perspective</strong>. My exposition will center around code and physical intuitions instead of mathematical derivations. Basically, I will strive to present the algorithms in a way that I wish I had come across when I was starting out.</p>

<blockquote>
  <p>“…everything became much clearer when I started writing code.”</p>
</blockquote>

<p>You might be eager to jump right in and learn about Neural Networks, backpropagation, how they can be applied to datasets in practice, etc. But before we get there, I’d like us to first forget about all that. Let’s take a step back and understand what is really going on at the core. Lets first talk about real-valued circuits.</p>

<p><em>Update note</em>: I suspended my work on this guide a while ago and redirected a lot of my energy to teaching CS231n (Convolutional Neural Networks) class at Stanford. The notes are on <a href="http://cs231n.github.io">cs231.github.io</a> and the course slides can be found <a href="http://cs231n.stanford.edu/syllabus.html">here</a>. These materials are highly related to material here, but more comprehensive and sometimes more polished.</p>

<h2 id="chapter-1-real-valued-circuits">Chapter 1: Real-valued Circuits</h2>

<p>In my opinion, the best way to think of Neural Networks is as real-valued circuits, where real values (instead of boolean values <code class="language-plaintext highlighter-rouge">{0,1}</code>) “flow” along edges and interact in gates. However, instead of gates such as <code class="language-plaintext highlighter-rouge">AND</code>, <code class="language-plaintext highlighter-rouge">OR</code>, <code class="language-plaintext highlighter-rouge">NOT</code>, etc, we have binary gates such as <code class="language-plaintext highlighter-rouge">*</code> (multiply), <code class="language-plaintext highlighter-rouge">+</code> (add), <code class="language-plaintext highlighter-rouge">max</code> or unary gates such as <code class="language-plaintext highlighter-rouge">exp</code>, etc. Unlike ordinary boolean circuits, however, we will eventually also have <strong>gradients</strong> flowing on the same edges of the circuit, but in the opposite direction. But we’re getting ahead of ourselves. Let’s focus and start out simple.</p>

<h3 id="base-case-single-gate-in-the-circuit">Base Case: Single Gate in the Circuit</h3>
<p>Lets first consider a single, simple circuit with one gate. Here’s an example:</p>

<div class="svgdiv">
<svg width="400" height="150">
  <rect x="130" y="20" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="90" y1="45" x2="130" y2="45" stroke="black" stroke-width="1" />
  <line x1="90" y1="95" x2="130" y2="95" stroke="black" stroke-width="1" />
  <text x="70" y="50" fill="black" text-anchor="middle" font-size="20px">x</text>
  <text x="70" y="100" fill="black" text-anchor="middle" font-size="20px">y</text>

  <text x="180" y="90" fill="black" text-anchor="middle" font-size="40px">*</text>
  <line x1="230" y1="70" x2="280" y2="70" stroke="black" stroke-width="1" />
</svg>
</div>

<p>The circuit takes two real-valued inputs <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> and computes <code class="language-plaintext highlighter-rouge">x * y</code> with the <code class="language-plaintext highlighter-rouge">*</code> gate. Javascript version of this would very simply look something like this:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">forwardMultiplyGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">x</span> <span class="o">*</span> <span class="nx">y</span><span class="p">;</span>
<span class="p">};</span>
<span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span> <span class="c1">// returns -6. Exciting.</span>
</code></pre></div></div>

<p>And in math form we can think of this gate as implementing the real-valued function:</p>

\[f(x,y) = x y\]

<p>As with this example, all of our gates will take one or two inputs and produce a <strong>single</strong> output value.</p>

<h4 id="the-goal">The Goal</h4>

<p>The problem we are interested in studying looks as follows:</p>

<ol>
  <li>We provide a given circuit some specific input values (e.g. <code class="language-plaintext highlighter-rouge">x = -2</code>, <code class="language-plaintext highlighter-rouge">y = 3</code>)</li>
  <li>The circuit computes an output value (e.g. <code class="language-plaintext highlighter-rouge">-6</code>)</li>
  <li>The core question then becomes: <em>How should one tweak the input slightly to increase the output?</em></li>
</ol>

<p>In this case, in what direction should we change <code class="language-plaintext highlighter-rouge">x,y</code> to get a number larger than <code class="language-plaintext highlighter-rouge">-6</code>? Note that, for example, <code class="language-plaintext highlighter-rouge">x = -1.99</code> and <code class="language-plaintext highlighter-rouge">y = 2.99</code> gives <code class="language-plaintext highlighter-rouge">x * y = -5.95</code>, which is higher than <code class="language-plaintext highlighter-rouge">-6.0</code>. Don’t get confused by this: <code class="language-plaintext highlighter-rouge">-5.95</code> is better (higher) than <code class="language-plaintext highlighter-rouge">-6.0</code>. It’s an improvement of <code class="language-plaintext highlighter-rouge">0.05</code>, even though the <em>magnitude</em> of <code class="language-plaintext highlighter-rouge">-5.95</code> (the distance from zero) happens to be lower.</p>

<h4 id="strategy-1-random-local-search">Strategy #1: Random Local Search</h4>

<p>Okay. So wait, we have a circuit, we have some inputs and we just want to tweak them slightly to increase the output value? Why is this hard? We can easily “forward” the circuit to compute the output for any given <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>. So isn’t this trivial? Why don’t we tweak <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> randomly and keep track of the tweak that works best:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// circuit with single gate for now</span>
<span class="kd">var</span> <span class="nx">forwardMultiplyGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="nx">x</span> <span class="o">*</span> <span class="nx">y</span><span class="p">;</span> <span class="p">};</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">3</span><span class="p">;</span> <span class="c1">// some input values</span>

<span class="c1">// try changing x,y randomly small amounts and keep track of what works best</span>
<span class="kd">var</span> <span class="nx">tweak_amount</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">best_out</span> <span class="o">=</span> <span class="o">-</span><span class="kc">Infinity</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">best_x</span> <span class="o">=</span> <span class="nx">x</span><span class="p">,</span> <span class="nx">best_y</span> <span class="o">=</span> <span class="nx">y</span><span class="p">;</span>
<span class="k">for</span><span class="p">(</span><span class="kd">var</span> <span class="nx">k</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">k</span> <span class="o">&lt;</span> <span class="mi">100</span><span class="p">;</span> <span class="nx">k</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">var</span> <span class="nx">x_try</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">+</span> <span class="nx">tweak_amount</span> <span class="o">*</span> <span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()</span> <span class="o">*</span> <span class="mi">2</span> <span class="o">-</span> <span class="mi">1</span><span class="p">);</span> <span class="c1">// tweak x a bit</span>
  <span class="kd">var</span> <span class="nx">y_try</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">+</span> <span class="nx">tweak_amount</span> <span class="o">*</span> <span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()</span> <span class="o">*</span> <span class="mi">2</span> <span class="o">-</span> <span class="mi">1</span><span class="p">);</span> <span class="c1">// tweak y a bit</span>
  <span class="kd">var</span> <span class="nx">out</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x_try</span><span class="p">,</span> <span class="nx">y_try</span><span class="p">);</span>
  <span class="k">if</span><span class="p">(</span><span class="nx">out</span> <span class="o">&gt;</span> <span class="nx">best_out</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// best improvement yet! Keep track of the x and y</span>
    <span class="nx">best_out</span> <span class="o">=</span> <span class="nx">out</span><span class="p">;</span> 
    <span class="nx">best_x</span> <span class="o">=</span> <span class="nx">x_try</span><span class="p">,</span> <span class="nx">best_y</span> <span class="o">=</span> <span class="nx">y_try</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>

<p>When I run this, I get <code class="language-plaintext highlighter-rouge">best_x = -1.9928</code>, <code class="language-plaintext highlighter-rouge">best_y = 2.9901</code>, and <code class="language-plaintext highlighter-rouge">best_out = -5.9588</code>. Again, <code class="language-plaintext highlighter-rouge">-5.9588</code> is higher than <code class="language-plaintext highlighter-rouge">-6.0</code>. So, we’re done, right? Not quite: This is a perfectly fine strategy for tiny problems with a few gates if you can afford the compute time, but it won’t do if we want to eventually consider huge circuits with millions of inputs. It turns out that we can do much better.</p>

<h4 id="stategy-2-numerical-gradient">Stategy #2: Numerical Gradient</h4>

<p>Here’s a better way. Remember again that in our setup we are given a circuit (e.g. our circuit with a single <code class="language-plaintext highlighter-rouge">*</code> gate) and some particular input (e.g. <code class="language-plaintext highlighter-rouge">x = -2, y = 3</code>). The gate computes the output (<code class="language-plaintext highlighter-rouge">-6</code>) and now we’d like to tweak <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> to make the output higher.</p>

<p>A nice intuition for what we’re about to do is as follows: Imagine taking the output value that comes out from the circuit and tugging on it in the positive direction. This positive tension will in turn translate through the gate and induce forces on the inputs <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>. Forces that tell us how <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> should change to increase the output value.</p>

<p>What might those forces look like in our specific example? Thinking through it, we can intuit that the force on <code class="language-plaintext highlighter-rouge">x</code> should also be positive, because making <code class="language-plaintext highlighter-rouge">x</code> slightly larger improves the circuit’s output. For example, increasing <code class="language-plaintext highlighter-rouge">x</code> from <code class="language-plaintext highlighter-rouge">x = -2</code> to <code class="language-plaintext highlighter-rouge">x = -1</code> would give us output <code class="language-plaintext highlighter-rouge">-3</code> - much larger than <code class="language-plaintext highlighter-rouge">-6</code>. On the other hand, we’d expect a negative force induced on <code class="language-plaintext highlighter-rouge">y</code> that pushes it to become lower (since a lower <code class="language-plaintext highlighter-rouge">y</code>, such as <code class="language-plaintext highlighter-rouge">y = 2</code>, down from the original <code class="language-plaintext highlighter-rouge">y = 3</code> would make output higher: <code class="language-plaintext highlighter-rouge">2 x -2 = -4</code>, again, larger than <code class="language-plaintext highlighter-rouge">-6</code>). That’s the intuition to keep in mind, anyway. As we go through this, it will turn out that forces I’m describing will in fact turn out to be the <strong>derivative</strong> of the output value with respect to its inputs (<code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>). You may have heard this term before.</p>

<blockquote>
  <p>The derivative can be thought of as a force on each input as we pull on the output to become higher.</p>
</blockquote>

<p>So how do we exactly evaluate this force (derivative)? It turns out that there is a very simple procedure for this. We will work backwards: Instead of pulling on the circuit’s output, we’ll iterate over every input one by one, increase it very slightly and look at what happens to the output value. The amount the output changes in response is the derivative. Enough intuitions for now. Lets look at the mathematical definition. We can write down the derivative for our function with respect to the inputs. For example, the derivative with respect to <code class="language-plaintext highlighter-rouge">x</code> can be computed as:</p>

<div>
$$
\frac{\partial f(x,y)}{\partial x} = \frac{f(x+h,y) - f(x,y)}{h}
$$
</div>

<p>Where \( h \) is small - it’s the tweak amount. Also, if you’re not very familiar with calculus it is important to note that in the left-hand side of the equation above, the horizontal line does <em>not</em> indicate division. The entire symbol \( \frac{\partial f(x,y)}{\partial x} \) is a single thing: the derivative of the function \( f(x,y) \) with respect to \( x \). The horizontal line on the right <em>is</em> division. I know it’s confusing but it’s standard notation. Anyway, I hope it doesn’t look too scary because it isn’t: The circuit was giving some initial output \( f(x,y) \), and then we changed one of the inputs by a tiny amount \(h \) and read the new output \( f(x+h, y) \). Subtracting those two quantities tells us the change, and the division by \(h \) just normalizes this change by the (arbitrary) tweak amount we used. In other words it’s expressing exactly what I described above and translates directly to this code:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">3</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">out</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// -6</span>
<span class="kd">var</span> <span class="nx">h</span> <span class="o">=</span> <span class="mf">0.0001</span><span class="p">;</span>

<span class="c1">// compute derivative with respect to x</span>
<span class="kd">var</span> <span class="nx">xph</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">+</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// -1.9999</span>
<span class="kd">var</span> <span class="nx">out2</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">xph</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// -5.9997</span>
<span class="kd">var</span> <span class="nx">x_derivative</span> <span class="o">=</span> <span class="p">(</span><span class="nx">out2</span> <span class="o">-</span> <span class="nx">out</span><span class="p">)</span> <span class="o">/</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// 3.0</span>

<span class="c1">// compute derivative with respect to y</span>
<span class="kd">var</span> <span class="nx">yph</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">+</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// 3.0001</span>
<span class="kd">var</span> <span class="nx">out3</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">yph</span><span class="p">);</span> <span class="c1">// -6.0002</span>
<span class="kd">var</span> <span class="nx">y_derivative</span> <span class="o">=</span> <span class="p">(</span><span class="nx">out3</span> <span class="o">-</span> <span class="nx">out</span><span class="p">)</span> <span class="o">/</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// -2.0</span>
</code></pre></div></div>

<p>Lets walk through <code class="language-plaintext highlighter-rouge">x</code> for example. We turned the knob from <code class="language-plaintext highlighter-rouge">x</code> to <code class="language-plaintext highlighter-rouge">x + h</code> and the circuit responded by giving a higher value (note again that yes, <code class="language-plaintext highlighter-rouge">-5.9997</code> is <em>higher</em> than <code class="language-plaintext highlighter-rouge">-6</code>: <code class="language-plaintext highlighter-rouge">-5.9997 &gt; -6</code>). The division by <code class="language-plaintext highlighter-rouge">h</code> is there to normalize the circuit’s response by the (arbitrary) value of <code class="language-plaintext highlighter-rouge">h</code> we chose to use here. Technically, you want the value of <code class="language-plaintext highlighter-rouge">h</code> to be infinitesimal (the precise mathematical definition of the gradient is defined as the limit of the expression as <code class="language-plaintext highlighter-rouge">h</code> goes to zero), but in practice <code class="language-plaintext highlighter-rouge">h=0.00001</code> or so works fine in most cases to get a good approximation. Now, we see that the derivative w.r.t. <code class="language-plaintext highlighter-rouge">x</code> is <code class="language-plaintext highlighter-rouge">+3</code>. I’m making the positive sign explicit, because it indicates that the circuit is tugging on x to become higher. The actual value, <code class="language-plaintext highlighter-rouge">3</code> can be interpreted as the <em>force</em> of that tug.</p>

<blockquote>
  <p>The derivative with respect to some input can be computed by tweaking that input by a small amount and observing the change on the output value.</p>
</blockquote>

<p>By the way, we usually  talk about the <em>derivative</em> with respect to a single input, or about a <strong>gradient</strong> with respect to all the inputs. The gradient is just made up of the derivatives of all the inputs concatenated in a vector (i.e. a list). Crucially, notice that if we let the inputs respond to the tug by following the gradient a tiny amount (i.e. we just add the derivative on top of every input), we can see that the value increases, as expected:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">out</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// before: -6</span>
<span class="nx">x</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">+</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">x_derivative</span><span class="p">;</span> <span class="c1">// x becomes -1.97</span>
<span class="nx">y</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">+</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">y_derivative</span><span class="p">;</span> <span class="c1">// y becomes 2.98</span>
<span class="kd">var</span> <span class="nx">out_new</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// -5.87! exciting.</span>
</code></pre></div></div>

<p>As expected, we changed the inputs by the gradient and the circuit now gives a slightly higher value (<code class="language-plaintext highlighter-rouge">-5.87 &gt; -6.0</code>). That was much simpler than trying random changes to <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>, right? A fact to appreciate here is that if you take calculus you can prove that the gradient is, in fact, the direction of the steepest increase of the function. There is no need to monkey around trying out random pertubations as done in Strategy #1. Evaluating the gradient requires just three evaluations of the forward pass of our circuit instead of hundreds, and gives the best tug you can hope for (locally) if you are interested in increasing the value of the output.</p>

<p><strong>Bigger step is not always better.</strong> Let me clarify on this point a bit. It is important to note that in this very simple example, using a bigger <code class="language-plaintext highlighter-rouge">step_size</code> than 0.01  will always work better. For example, <code class="language-plaintext highlighter-rouge">step_size = 1.0</code> gives output <code class="language-plaintext highlighter-rouge">-1</code> (higer, better!), and indeed infinite step size would give infinitely good results. The crucial thing to realize is that once our circuits get much more complex (e.g. entire neural networks), the function from inputs to the output value will be more chaotic and wiggly. The gradient guarantees that if you have a very small (indeed, infinitesimally small) step size, then you will definitely get a higher number when you follow its direction, and for that infinitesimally small step size there is no other direction that would have worked better. But if you use a bigger step size (e.g. <code class="language-plaintext highlighter-rouge">step_size = 0.01</code>) all bets are off. The reason we can get away with a larger step size than infinitesimally small is that our functions are usually relatively smooth. But really, we’re crossing our fingers and hoping for the best.</p>

<p><strong>Hill-climbing analogy.</strong> One analogy I’ve heard before is that the output value of our circut is like the height of a hill, and we are blindfolded and trying to climb upwards. We can sense the steepness of the hill at our feet (the gradient), so when we shuffle our feet a bit we will go upwards. But if we took a big, overconfident step, we could have stepped right into a hole.</p>

<p>Great, I hope I’ve convinced you that the numerical gradient is indeed a very useful thing to evaluate, and that it is cheap. But. It turns out that we can do <em>even</em> better.</p>

<h4 id="strategy-3-analytic-gradient">Strategy #3: Analytic Gradient</h4>

<p>In the previous section we evaluated the gradient by probing the circuit’s output value, independently for every input. This procedure gives you what we call a <strong>numerical gradient</strong>. This approach, however, is <em>still</em> expensive because we need to compute the circuit’s output as we tweak every input value independently a small amount. So the complexity of evaluating the gradient is linear in number of inputs. But in practice we will have hundreds, thousands or (for neural networks) even tens to hundreds of millions of inputs, and the circuits aren’t just one multiply gate but huge expressions that can be expensive to compute. We need something better.</p>

<p>Luckily, there is an easier and <em>much</em> faster way to compute the gradient: we can use calculus to derive a direct expression for it that will be as simple to evaluate as the circuit’s output value. We call this an <strong>analytic gradient</strong> and there will be no need for tweaking anything. You may have seen other people who teach Neural Networks derive the gradient in huge and, frankly, scary and confusing mathematical equations (if you’re not well-versed in maths). But it’s unnecessary. I’ve written plenty of Neural Nets code and I rarely have to do mathematical derivation longer than two lines, and 95% of the time it can be done without writing anything at all. That is because we will only ever derive the gradient for very small and simple expressions (think of it as the <strong>base case</strong>) and then I will show you how we can compose these very simply with <strong>chain rule</strong> to evaluate the full gradient (think inductive/recursive case).</p>

<blockquote>
  <p>The analytic derivative requires no tweaking of the inputs. It can be derived using mathematics (calculus).</p>
</blockquote>

<p>If you remember your product rules, power rules, quotient rules, etc. (see e.g. <a href="http://www.mathsisfun.com/calculus/derivatives-rules.html">derivative rules</a> or <a href="http://en.wikipedia.org/wiki/Differentiation_rules">wiki page</a>), it’s very easy to write down the derivitative with respect to both <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> for a small expression such as <code class="language-plaintext highlighter-rouge">x * y</code>. But suppose you don’t remember your calculus rules. We can go back to the definition. For example, here’s the expression for the derivative w.r.t <code class="language-plaintext highlighter-rouge">x</code>:</p>

<div>
$$
\frac{\partial f(x,y)}{\partial x} = \frac{f(x+h,y) - f(x,y)}{h}
$$
</div>

<p>(Technically I’m not writing the limit as <code class="language-plaintext highlighter-rouge">h</code> goes to zero, forgive me math people). Okay and lets plug in our function ( \( f(x,y) = x y \) ) into the expression. Ready for the hardest piece of math of this entire article? Here we go:</p>

<div>
$$
\frac{\partial f(x,y)}{\partial x} = \frac{f(x+h,y) - f(x,y)}{h}
= \frac{(x+h)y - xy}{h}
= \frac{xy + hy - xy}{h}
= \frac{hy}{h}
= y
$$
</div>

<p>That’s interesting. The derivative with respect to <code class="language-plaintext highlighter-rouge">x</code> is just equal to <code class="language-plaintext highlighter-rouge">y</code>. Did you notice the coincidence in the previous section? We tweaked <code class="language-plaintext highlighter-rouge">x</code> to <code class="language-plaintext highlighter-rouge">x+h</code> and calculated <code class="language-plaintext highlighter-rouge">x_derivative = 3.0</code>, which exactly happens to be the value of <code class="language-plaintext highlighter-rouge">y</code> in that example. It turns out that wasn’t a coincidence at all because that’s just what the analytic gradient tells us the <code class="language-plaintext highlighter-rouge">x</code> derivative should be for <code class="language-plaintext highlighter-rouge">f(x,y) = x * y</code>. The derivative with respect to <code class="language-plaintext highlighter-rouge">y</code>, by the way, turns out to be <code class="language-plaintext highlighter-rouge">x</code>, unsurprisingly by symmetry. So there is no need for any tweaking! We invoked powerful mathematics and can now transform our derivative calculation into the following code:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">3</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">out</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// before: -6</span>
<span class="kd">var</span> <span class="nx">x_gradient</span> <span class="o">=</span> <span class="nx">y</span><span class="p">;</span> <span class="c1">// by our complex mathematical derivation above</span>
<span class="kd">var</span> <span class="nx">y_gradient</span> <span class="o">=</span> <span class="nx">x</span><span class="p">;</span>

<span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
<span class="nx">x</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">x_gradient</span><span class="p">;</span> <span class="c1">// -1.97</span>
<span class="nx">y</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">y_gradient</span><span class="p">;</span> <span class="c1">// 2.98</span>
<span class="kd">var</span> <span class="nx">out_new</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// -5.87. Higher output! Nice.</span>
</code></pre></div></div>

<p>To compute the gradient we went from forwarding the circuit hundreds of times (Strategy #1) to forwarding it only on order of number of times twice the number of inputs (Strategy #2), to forwarding it a single time! And it gets EVEN better, since the more expensive strategies (#1 and #2) only give an approximation of the gradient, while #3 (the fastest one by far) gives you the <em>exact</em> gradient. No approximations. The only downside is that you should be comfortable with some calculus 101.</p>

<p>Lets recap what we have learned:</p>

<ul>
  <li>INPUT: We are given a circuit, some inputs and compute an output value.</li>
  <li>OUTPUT: We are then interested finding small changes to each input (independently) that would make the output higher.</li>
  <li>Strategy #1: One silly way is to <strong>randomly search</strong> for small pertubations of the inputs and keep track of what gives the highest increase in output.</li>
  <li>Strategy #2: We saw we can do much better by computing the gradient. Regardless of how complicated the circuit is, the <strong>numerical gradient</strong> is very simple (but relatively expensive) to compute. We compute it by <em>probing</em> the circuit’s output value as we tweak the inputs one at a time.</li>
  <li>Strategy #3: In the end, we saw that we can be even more clever and analytically derive a direct expression to get the <strong>analytic gradient</strong>. It is identical to the numerical gradient, it is fastest by far, and there is no need for any tweaking.</li>
</ul>

<p>In practice by the way (and we will get to this once again later), all Neural Network libraries always compute the analytic gradient, but the correctness of the implementation is verified by comparing it to the numerical gradient. That’s because the numerical gradient is very easy to evaluate (but can be a bit expensive to compute), while the analytic gradient can contain bugs at times, but is usually extremely efficient to compute. As we will see, evaluating the gradient (i.e. while doing <em>backprop</em>, or <em>backward pass</em>) will turn out to cost about as much as evaluating the <em>forward pass</em>.</p>

<h3 id="recursive-case-circuits-with-multiple-gates">Recursive Case: Circuits with Multiple Gates</h3>

<p>But hold on, you say: <em>“The analytic gradient was trivial to derive for your super-simple expression. This is useless. What do I do when the expressions are much larger? Don’t the equations get huge and complex very fast?”</em>. Good question. Yes the expressions get much more complex. No, this doesn’t make it much harder. As we will see, every gate will be hanging out by itself, completely unaware of any details of the huge and complex circuit that it could be part of. It will only worry about its inputs and it will compute its local derivatives as seen in the previous section, except now there will be a single extra multiplication it will have to do.</p>

<blockquote>
  <p>A single extra multiplication will turn a single (useless gate) into a cog in the complex machine that is an entire neural network.</p>
</blockquote>

<p>I should stop hyping it up now. I hope I’ve piqued your interest! Lets drill down into details and get two gates involved with this next example:</p>

<div class="svgdiv">
<svg width="500" height="150">
  <rect x="130" y="20" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="90" y1="45" x2="130" y2="45" stroke="black" stroke-width="1" />
  <line x1="90" y1="95" x2="130" y2="95" stroke="black" stroke-width="1" />
  <text x="70" y="50" fill="black" text-anchor="middle" font-size="20px">x</text>
  <text x="70" y="100" fill="black" text-anchor="middle" font-size="20px">y</text>
  <text x="70" y="150" fill="black" text-anchor="middle" font-size="20px">z</text>

  <text x="180" y="85" fill="black" text-anchor="middle" font-size="40px">+</text>
  <text x="270" y="60" fill="black" text-anchor="middle" font-size="20px">q</text>

  <line x1="230" y1="70" x2="320" y2="70" stroke="black" stroke-width="1" />

  <line x1="90" y1="145" x2="300" y2="145" stroke="black" stroke-width="1" />
  <line x1="300" y1="145" x2="300" y2="100" stroke="black" stroke-width="1" />
  <line x1="300" y1="100" x2="320" y2="100" stroke="black" stroke-width="1" />

  <rect x="320" y="32" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="420" y1="82" x2="450" y2="82" stroke="black" stroke-width="1" />

  <text x="370" y="105" fill="black" text-anchor="middle" font-size="40px">*</text>
  <text x="460" y="88" fill="black" text-anchor="middle" font-size="20px">f</text>
</svg>
</div>

<p>The expression we are computing now is \( f(x,y,z) = (x + y) z \). Lets structure the code as follows to make the gates explicit as functions:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">forwardMultiplyGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="p">{</span> 
  <span class="k">return</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">b</span><span class="p">;</span>
<span class="p">};</span>
<span class="kd">var</span> <span class="nx">forwardAddGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="p">{</span> 
  <span class="k">return</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">;</span>
<span class="p">};</span>
<span class="kd">var</span> <span class="nx">forwardCircuit</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">z</span><span class="p">)</span> <span class="p">{</span> 
  <span class="kd">var</span> <span class="nx">q</span> <span class="o">=</span> <span class="nx">forwardAddGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span>
  <span class="kd">var</span> <span class="nx">f</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">q</span><span class="p">,</span> <span class="nx">z</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">f</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">5</span><span class="p">,</span> <span class="nx">z</span> <span class="o">=</span> <span class="o">-</span><span class="mi">4</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">f</span> <span class="o">=</span> <span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="nx">z</span><span class="p">);</span> <span class="c1">// output is -12</span>
</code></pre></div></div>

<p>In the above, I am using <code class="language-plaintext highlighter-rouge">a</code> and <code class="language-plaintext highlighter-rouge">b</code> as the local variables in the gate functions so that we don’t get these confused with our circuit inputs <code class="language-plaintext highlighter-rouge">x,y,z</code>. As before, we are interested in finding the derivatives with respect to the three inputs <code class="language-plaintext highlighter-rouge">x,y,z</code>. But how do we compute it now that there are multiple gates involved? First, lets pretend that the <code class="language-plaintext highlighter-rouge">+</code> gate is not there and that we only have two variables in the circuit: <code class="language-plaintext highlighter-rouge">q,z</code> and a single <code class="language-plaintext highlighter-rouge">*</code> gate. Note that the <code class="language-plaintext highlighter-rouge">q</code> is is output of the <code class="language-plaintext highlighter-rouge">+</code> gate. If we don’t worry about <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> but only about <code class="language-plaintext highlighter-rouge">q</code> and <code class="language-plaintext highlighter-rouge">z</code>, then we are back to having only a single gate, and as far as that single <code class="language-plaintext highlighter-rouge">*</code> gate is concerned, we know what the (analytic) derivates are from previous section. We can write them down (except here we’re replacing <code class="language-plaintext highlighter-rouge">x,y</code> with <code class="language-plaintext highlighter-rouge">q,z</code>):</p>

\[f(q,z) = q z \hspace{0.5in} \implies \hspace{0.5in} \frac{\partial f(q,z)}{\partial q} = z, \hspace{1in} \frac{\partial f(q,z)}{\partial z} = q\]

<p>Simple enough: these are the expressions for the gradient with respect to <code class="language-plaintext highlighter-rouge">q</code> and <code class="language-plaintext highlighter-rouge">z</code>. But wait, we don’t want gradient with respect to <code class="language-plaintext highlighter-rouge">q</code>, but with respect to the inputs: <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>. Luckily, <code class="language-plaintext highlighter-rouge">q</code> is computed as a function of <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> (by addition in our example). We can write down the gradient for the addition gate as well, it’s even simpler:</p>

\[q(x,y) = x + y \hspace{0.5in} \implies \hspace{0.5in} \frac{\partial q(x,y)}{\partial x} = 1, \hspace{1in} \frac{\partial q(x,y)}{\partial y} = 1\]

<p>That’s right, the derivatives are just 1, regardless of the actual values of <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>. If you think about it, this makes sense because to make the output of a single addition gate higher, we expect a positive tug on both <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>, regardless of their values.</p>

<h4 id="backpropagation">Backpropagation</h4>

<p>We are finally ready to invoke the <strong>Chain Rule</strong>: We know how to compute the gradient of <code class="language-plaintext highlighter-rouge">q</code> with respect to <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> (that’s a single gate case with <code class="language-plaintext highlighter-rouge">+</code> as the gate). And we know how to compute the gradient of our final output with respect to <code class="language-plaintext highlighter-rouge">q</code>. The chain rule tells us how to combine these to get the gradient of the final output with respect to <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code>, which is what we’re ultimately interested in. Best of all, the chain rule very simply states that the right thing to do is to simply multiply the gradients together to chain them. For example, the final derivative for <code class="language-plaintext highlighter-rouge">x</code> will be:</p>

\[\frac{\partial f(q,z)}{\partial x} = \frac{\partial q(x,y)}{\partial x} \frac{\partial f(q,z)}{\partial q}\]

<p>There are many symbols there so maybe this is confusing again, but it’s really just two numbers being multiplied together. Here is the code:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// initial conditions</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">5</span><span class="p">,</span> <span class="nx">z</span> <span class="o">=</span> <span class="o">-</span><span class="mi">4</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">q</span> <span class="o">=</span> <span class="nx">forwardAddGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// q is 3</span>
<span class="kd">var</span> <span class="nx">f</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">q</span><span class="p">,</span> <span class="nx">z</span><span class="p">);</span> <span class="c1">// output is -12</span>

<span class="c1">// gradient of the MULTIPLY gate with respect to its inputs</span>
<span class="c1">// wrt is short for "with respect to"</span>
<span class="kd">var</span> <span class="nx">derivative_f_wrt_z</span> <span class="o">=</span> <span class="nx">q</span><span class="p">;</span> <span class="c1">// 3</span>
<span class="kd">var</span> <span class="nx">derivative_f_wrt_q</span> <span class="o">=</span> <span class="nx">z</span><span class="p">;</span> <span class="c1">// -4</span>

<span class="c1">// derivative of the ADD gate with respect to its inputs</span>
<span class="kd">var</span> <span class="nx">derivative_q_wrt_x</span> <span class="o">=</span> <span class="mf">1.0</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">derivative_q_wrt_y</span> <span class="o">=</span> <span class="mf">1.0</span><span class="p">;</span>

<span class="c1">// chain rule</span>
<span class="kd">var</span> <span class="nx">derivative_f_wrt_x</span> <span class="o">=</span> <span class="nx">derivative_q_wrt_x</span> <span class="o">*</span> <span class="nx">derivative_f_wrt_q</span><span class="p">;</span> <span class="c1">// -4</span>
<span class="kd">var</span> <span class="nx">derivative_f_wrt_y</span> <span class="o">=</span> <span class="nx">derivative_q_wrt_y</span> <span class="o">*</span> <span class="nx">derivative_f_wrt_q</span><span class="p">;</span> <span class="c1">// -4</span>
</code></pre></div></div>

<p>That’s it. We computed the gradient (the forces) and now we can let our inputs respond to it by a bit. Lets add the gradients on top of the inputs. The output value of the circuit better increase, up from -12!</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// final gradient, from above: [-4, -4, 3]</span>
<span class="kd">var</span> <span class="nx">gradient_f_wrt_xyz</span> <span class="o">=</span> <span class="p">[</span><span class="nx">derivative_f_wrt_x</span><span class="p">,</span> <span class="nx">derivative_f_wrt_y</span><span class="p">,</span> <span class="nx">derivative_f_wrt_z</span><span class="p">]</span>

<span class="c1">// let the inputs respond to the force/tug:</span>
<span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
<span class="nx">x</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">+</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">derivative_f_wrt_x</span><span class="p">;</span> <span class="c1">// -2.04</span>
<span class="nx">y</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">+</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">derivative_f_wrt_y</span><span class="p">;</span> <span class="c1">// 4.96</span>
<span class="nx">z</span> <span class="o">=</span> <span class="nx">z</span> <span class="o">+</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">derivative_f_wrt_z</span><span class="p">;</span> <span class="c1">// -3.97</span>

<span class="c1">// Our circuit now better give higher output:</span>
<span class="kd">var</span> <span class="nx">q</span> <span class="o">=</span> <span class="nx">forwardAddGate</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// q becomes 2.92</span>
<span class="kd">var</span> <span class="nx">f</span> <span class="o">=</span> <span class="nx">forwardMultiplyGate</span><span class="p">(</span><span class="nx">q</span><span class="p">,</span> <span class="nx">z</span><span class="p">);</span> <span class="c1">// output is -11.59, up from -12! Nice!</span>

</code></pre></div></div>

<p>Looks like that worked! Lets now try to interpret intuitively what just happened. The circuit wants to output higher values. The last gate saw inputs <code class="language-plaintext highlighter-rouge">q = 3, z = -4</code> and computed output <code class="language-plaintext highlighter-rouge">-12</code>. “Pulling” upwards on this output value induced a force on both <code class="language-plaintext highlighter-rouge">q</code> and <code class="language-plaintext highlighter-rouge">z</code>: To increase the output value, the circuit “wants” <code class="language-plaintext highlighter-rouge">z</code> to increase, as can be seen by the positive value of the derivative(<code class="language-plaintext highlighter-rouge">derivative_f_wrt_z = +3</code>). Again, the size of this derivative can be interpreted as the magnitude of the force. On the other hand, <code class="language-plaintext highlighter-rouge">q</code> felt a stronger and downward force, since <code class="language-plaintext highlighter-rouge">derivative_f_wrt_q = -4</code>. In other words the circuit wants <code class="language-plaintext highlighter-rouge">q</code> to decrease, with a force of <code class="language-plaintext highlighter-rouge">4</code>.</p>

<p>Now we get to the second, <code class="language-plaintext highlighter-rouge">+</code> gate which outputs <code class="language-plaintext highlighter-rouge">q</code>. By default, the <code class="language-plaintext highlighter-rouge">+</code> gate computes its derivatives which tells us how to change <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> to make <code class="language-plaintext highlighter-rouge">q</code> higher. BUT! Here is the <strong>crucial point</strong>: the gradient on <code class="language-plaintext highlighter-rouge">q</code> was computed as negative (<code class="language-plaintext highlighter-rouge">derivative_f_wrt_q = -4</code>), so the circuit wants <code class="language-plaintext highlighter-rouge">q</code> to <em>decrease</em>, and with a force of <code class="language-plaintext highlighter-rouge">4</code>! So if the <code class="language-plaintext highlighter-rouge">+</code> gate wants to contribute to making the final output value larger, it needs to listen to the gradient signal coming from the top. In this particular case, it needs to apply tugs on <code class="language-plaintext highlighter-rouge">x,y</code> opposite of what it would normally apply, and with a force of <code class="language-plaintext highlighter-rouge">4</code>, so to speak. The multiplication by <code class="language-plaintext highlighter-rouge">-4</code> seen in the chain rule achieves exactly this: instead of applying a positive force of <code class="language-plaintext highlighter-rouge">+1</code> on both <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> (the local derivative), the full circuit’s gradient on both <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> becomes <code class="language-plaintext highlighter-rouge">1 x -4 = -4</code>. This makes sense: the circuit wants both <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> to get smaller because this will make <code class="language-plaintext highlighter-rouge">q</code> smaller, which in turn will make <code class="language-plaintext highlighter-rouge">f</code> larger.</p>

<blockquote>
  <p>If this makes sense, you understand backpropagation.</p>
</blockquote>

<p>Lets <strong>recap</strong> once again what we learned:</p>

<ul>
  <li>
    <p>In the previous chapter we saw that in the case of a single gate (or a single expression), we can derive the analytic gradient using simple calculus. We interpreted the gradient as a force, or a tug on the inputs that pulls them in a direction which would make this gate’s output higher.</p>
  </li>
  <li>
    <p>In case of multiple gates everything stays pretty much the same way: every gate is hanging out by itself completely unaware of the circuit it is embedded in. Some inputs come in and the gate computes its output and the derivate with respect to the inputs. The <em>only</em> difference now is that suddenly, something can pull on this gate from above. That’s the gradient of the final circuit output value with respect to the ouput this gate computed. It is the circuit asking the gate to output higher or lower numbers, and with some force. The gate simply takes this force and multiplies it to all the forces it computed for its inputs before (chain rule). This has the desired effect:</p>
  </li>
</ul>

<ol>
  <li>If a gate experiences a strong positive pull from above, it will also pull harder on its own inputs, scaled by the force it is experiencing from above</li>
  <li>And if it experiences a negative tug, this means that circuit wants its value to decrease not increase, so it will flip the force of the pull on its inputs to make its own output value smaller.</li>
</ol>

<blockquote>
  <p>A nice picture to have in mind is that as we pull on the circuit’s output value at the end, this induces pulls downward through the entire circuit, all the way down to the inputs.</p>
</blockquote>

<p>Isn’t it beautiful? The only difference between the case of a single gate and multiple interacting gates that compute arbitrarily complex expressions is this additional multipy operation that now happens in each gate.</p>

<h4 id="patterns-in-the-backward-flow">Patterns in the “backward” flow</h4>

<p>Lets look again at our example circuit with the numbers filled in. The first circuit shows the raw values, and the second circuit shows the gradients that flow back to the inputs as discussed. Notice that the gradient always starts off with <code class="language-plaintext highlighter-rouge">+1</code> at the end to start off the chain. This is the (default) pull on the circuit to have its value increased.</p>

<div class="svgdiv">
<svg width="600" height="350">

  <text x="550" y="90" fill="black" text-anchor="middle" font-size="16px">(Values)</text>
  <rect x="130" y="20" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="90" y1="45" x2="130" y2="45" stroke="black" stroke-width="1" />
  <line x1="90" y1="95" x2="130" y2="95" stroke="black" stroke-width="1" />
  <text x="70" y="50" fill="black" text-anchor="middle" font-size="20px">-2</text>
  <text x="70" y="100" fill="black" text-anchor="middle" font-size="20px">5</text>
  <text x="70" y="150" fill="black" text-anchor="middle" font-size="20px">-4</text>

  <text x="180" y="85" fill="black" text-anchor="middle" font-size="40px">+</text>
  <text x="270" y="60" fill="black" text-anchor="middle" font-size="20px">3</text>

  <line x1="230" y1="70" x2="320" y2="70" stroke="black" stroke-width="1" />

  <line x1="90" y1="145" x2="300" y2="145" stroke="black" stroke-width="1" />
  <line x1="300" y1="145" x2="300" y2="100" stroke="black" stroke-width="1" />
  <line x1="300" y1="100" x2="320" y2="100" stroke="black" stroke-width="1" />

  <rect x="320" y="32" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="420" y1="82" x2="450" y2="82" stroke="black" stroke-width="1" />

  <text x="370" y="105" fill="black" text-anchor="middle" font-size="40px">*</text>
  <text x="460" y="88" fill="black" text-anchor="middle" font-size="20px">-12</text>

  <text x="550" y="290" fill="black" text-anchor="middle" font-size="16px">(Gradients)</text>
  <rect x="130" y="220" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="90" y1="245" x2="130" y2="245" stroke="black" stroke-width="1" />
  <line x1="90" y1="295" x2="130" y2="295" stroke="black" stroke-width="1" />
  <text x="70" y="250" fill="black" text-anchor="middle" font-size="20px">-4</text>
  <text x="70" y="300" fill="black" text-anchor="middle" font-size="20px">-4</text>
  <text x="70" y="350" fill="black" text-anchor="middle" font-size="20px">3</text>

  <text x="180" y="285" fill="black" text-anchor="middle" font-size="40px">+</text>
  <text x="270" y="260" fill="black" text-anchor="middle" font-size="20px">-4</text>

  <line x1="230" y1="270" x2="320" y2="270" stroke="black" stroke-width="1" />

  <line x1="90" y1="345" x2="300" y2="345" stroke="black" stroke-width="1" />
  <line x1="300" y1="345" x2="300" y2="300" stroke="black" stroke-width="1" />
  <line x1="300" y1="300" x2="320" y2="300" stroke="black" stroke-width="1" />

  <rect x="320" y="232" width="100" height="100" stroke="black" stroke-width="1" fill="white" />
  <line x1="420" y1="282" x2="450" y2="282" stroke="black" stroke-width="1" />

  <text x="370" y="305" fill="black" text-anchor="middle" font-size="40px">*</text>
  <text x="460" y="288" fill="black" text-anchor="middle" font-size="20px">1</text>

</svg>
</div>

<p>After a while you start to notice patterns in how the gradients flow backward in the circuits. For example, the <code class="language-plaintext highlighter-rouge">+</code> gate always takes the gradient on top and simply passes it on to all of its inputs (notice the example with -4 simply passed on to both of the inputs of <code class="language-plaintext highlighter-rouge">+</code> gate). This is because its own derivative for the inputs is just <code class="language-plaintext highlighter-rouge">+1</code>, regardless of what the actual values of the inputs are, so in the chain rule, the gradient from above is just multiplied by 1 and stays the same. Similar intuitions apply to, for example, a <code class="language-plaintext highlighter-rouge">max(x,y)</code> gate. Since the gradient of <code class="language-plaintext highlighter-rouge">max(x,y)</code> with respect to its input is <code class="language-plaintext highlighter-rouge">+1</code> for whichever one of <code class="language-plaintext highlighter-rouge">x</code>, <code class="language-plaintext highlighter-rouge">y</code> is larger and <code class="language-plaintext highlighter-rouge">0</code> for the other, this gate is during backprop effectively just a gradient “switch”: it will take the gradient from above and “route” it to the input that had a higher value during the forward pass.</p>

<p><strong>Numerical Gradient Check.</strong> Before we finish with this section, lets just make sure that the (analytic) gradient we computed by backprop above is correct as a sanity check. Remember that we can do this simply by computing the numerical gradient and making sure that we get <code class="language-plaintext highlighter-rouge">[-4, -4, 3]</code> for <code class="language-plaintext highlighter-rouge">x,y,z</code>. Here’s the code:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// initial conditions</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">5</span><span class="p">,</span> <span class="nx">z</span> <span class="o">=</span> <span class="o">-</span><span class="mi">4</span><span class="p">;</span>

<span class="c1">// numerical gradient check</span>
<span class="kd">var</span> <span class="nx">h</span> <span class="o">=</span> <span class="mf">0.0001</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x_derivative</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="o">+</span><span class="nx">h</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">z</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">z</span><span class="p">))</span> <span class="o">/</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// -4</span>
<span class="kd">var</span> <span class="nx">y_derivative</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="o">+</span><span class="nx">h</span><span class="p">,</span><span class="nx">z</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">z</span><span class="p">))</span> <span class="o">/</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// -4</span>
<span class="kd">var</span> <span class="nx">z_derivative</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">z</span><span class="o">+</span><span class="nx">h</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuit</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">z</span><span class="p">))</span> <span class="o">/</span> <span class="nx">h</span><span class="p">;</span> <span class="c1">// 3</span>
</code></pre></div></div>

<p>and we get <code class="language-plaintext highlighter-rouge">[-4, -4, 3]</code>, as computed with backprop. phew! :)</p>

<h3 id="example-single-neuron">Example: Single Neuron</h3>

<p>In the previous section you hopefully got the basic intuition behind backpropagation. Lets now look at an even more complicated and borderline practical example. We will consider a 2-dimensional neuron that computes the following function:</p>

\[f(x,y,a,b,c) = \sigma(ax + by + c)\]

<p>In this expression, \( \sigma \) is the <em>sigmoid</em> function. Its best thought of as a “squashing function”, because it takes the input and squashes it to be between zero and one: Very negative values are squashed towards zero and positive values get squashed towards one. For example, we have <code class="language-plaintext highlighter-rouge">sig(-5) = 0.006, sig(0) = 0.5, sig(5) = 0.993</code>. Sigmoid function is defined as:</p>

\[\sigma(x) = \frac{1}{1 + e^{-x}}\]

<p>The gradient with respect to its single input, as you can check on Wikipedia or derive yourself if you know some calculus is given by this expression:</p>

\[\frac{\partial \sigma(x)}{\partial x} = \sigma(x) (1 - \sigma(x))\]

<p>For example, if the input to the sigmoid gate is <code class="language-plaintext highlighter-rouge">x = 3</code>, the gate will compute output <code class="language-plaintext highlighter-rouge">f = 1.0 / (1.0 + Math.exp(-x)) = 0.95</code>, and then the (local) gradient on its input will simply be <code class="language-plaintext highlighter-rouge">dx = (0.95) * (1 - 0.95) = 0.0475</code>.</p>

<p>That’s all we need to use this gate: we know how to take an input and <em>forward</em> it through the sigmoid gate, and we also have the expression for the gradient with respect to its input, so we can also <em>backprop</em> through it. Another thing to note is that technically, the sigmoid function is made up of an entire series of gates in a line that compute more <em>atomic</em> functions: an exponentiation gate, an addition gate and a division gate. Treating it so would work perfectly fine but for this example I chose to collapse all of these gates into a single gate that just computes sigmoid in one shot, because the gradient expression turns out to be simple.</p>

<p>Lets take this opportunity to carefully structure the associated code in a nice and modular way. First, I’d like you to note that every <strong>wire</strong> in our diagrams has two numbers associated with it:</p>

<ol>
  <li>the value it carries during the forward pass</li>
  <li>the gradient (i.e the <em>pull</em>) that flows back through it in the backward pass</li>
</ol>

<p>Lets create a simple <code class="language-plaintext highlighter-rouge">Unit</code> structure that will store these two values on every wire. Our gates will now operate over <code class="language-plaintext highlighter-rouge">Unit</code>s: they will take them as inputs and create them as outputs.</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// every Unit corresponds to a wire in the diagrams</span>
<span class="kd">var</span> <span class="nx">Unit</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">value</span><span class="p">,</span> <span class="nx">grad</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// value computed in the forward pass</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="nx">value</span><span class="p">;</span> 
  <span class="c1">// the derivative of circuit output w.r.t this unit, computed in backward pass</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">grad</span> <span class="o">=</span> <span class="nx">grad</span><span class="p">;</span> 
<span class="p">}</span>
</code></pre></div></div>

<p>In addition to Units we also need 3 gates: <code class="language-plaintext highlighter-rouge">+</code>, <code class="language-plaintext highlighter-rouge">*</code> and <code class="language-plaintext highlighter-rouge">sig</code> (sigmoid). Lets start out by implementing a multiply gate. I’m using Javascript here which has a funny way of simulating classes using functions. If you’re not a Javascript - familiar person, all that’s going on here is that I’m defining a class that has certain properties (accessed with use of <code class="language-plaintext highlighter-rouge">this</code> keyword), and some methods (which in Javascript are placed into the function’s <em>prototype</em>). Just think about these as class methods. Also keep in mind that the way we will use these eventually is that we will first <code class="language-plaintext highlighter-rouge">forward</code> all the gates one by one, and then <code class="language-plaintext highlighter-rouge">backward</code> all the gates in reverse order. Here is the implementation:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="kd">var</span> <span class="nx">multiplyGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(){</span> <span class="p">};</span>
<span class="nx">multiplyGate</span><span class="p">.</span><span class="nx">prototype</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">forward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">u0</span><span class="p">,</span> <span class="nx">u1</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// store pointers to input Units u0 and u1 and output unit utop</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u0</span> <span class="o">=</span> <span class="nx">u0</span><span class="p">;</span> 
    <span class="k">this</span><span class="p">.</span><span class="nx">u1</span> <span class="o">=</span> <span class="nx">u1</span><span class="p">;</span> 
    <span class="k">this</span><span class="p">.</span><span class="nx">utop</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="nx">u0</span><span class="p">.</span><span class="nx">value</span> <span class="o">*</span> <span class="nx">u1</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">;</span>
  <span class="p">},</span>
  <span class="na">backward</span><span class="p">:</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="c1">// take the gradient in output unit and chain it with the</span>
    <span class="c1">// local gradients, which we derived for multiply gate before</span>
    <span class="c1">// then write those gradients to those Units.</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u0</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">u1</span><span class="p">.</span><span class="nx">value</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u1</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="k">this</span><span class="p">.</span><span class="nx">u0</span><span class="p">.</span><span class="nx">value</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>

<p>The multiply gate takes two units that each hold a value and creates a unit that stores its output. The gradient is initialized to zero. Then notice that in the <code class="language-plaintext highlighter-rouge">backward</code> function call we get the gradient from the output unit we produced during the forward pass (which will by now hopefully have its gradient filled in) and multiply it with the local gradient for this gate (chain rule!). This gate computes multiplication (<code class="language-plaintext highlighter-rouge">u0.value * u1.value</code>) during forward pass, so recall that the gradient w.r.t <code class="language-plaintext highlighter-rouge">u0</code> is <code class="language-plaintext highlighter-rouge">u1.value</code> and w.r.t <code class="language-plaintext highlighter-rouge">u1</code> is <code class="language-plaintext highlighter-rouge">u0.value</code>. Also note that we are using <code class="language-plaintext highlighter-rouge">+=</code> to add onto the gradient in the <code class="language-plaintext highlighter-rouge">backward</code> function. This will allow us to possibly use the output of one gate multiple times (think of it as a wire branching out), since it turns out that the gradients from these different branches just add up when computing the final gradient with respect to the circuit output. The other two gates are defined analogously:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">addGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(){</span> <span class="p">};</span>
<span class="nx">addGate</span><span class="p">.</span><span class="nx">prototype</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">forward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">u0</span><span class="p">,</span> <span class="nx">u1</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u0</span> <span class="o">=</span> <span class="nx">u0</span><span class="p">;</span> 
    <span class="k">this</span><span class="p">.</span><span class="nx">u1</span> <span class="o">=</span> <span class="nx">u1</span><span class="p">;</span> <span class="c1">// store pointers to input units</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">utop</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="nx">u0</span><span class="p">.</span><span class="nx">value</span> <span class="o">+</span> <span class="nx">u1</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">;</span>
  <span class="p">},</span>
  <span class="na">backward</span><span class="p">:</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="c1">// add gate. derivative wrt both inputs is 1</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u0</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="mi">1</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u1</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="mi">1</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">sigmoidGate</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span> 
  <span class="c1">// helper function</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">sig</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="mi">1</span> <span class="o">/</span> <span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">exp</span><span class="p">(</span><span class="o">-</span><span class="nx">x</span><span class="p">));</span> <span class="p">};</span>
<span class="p">};</span>
<span class="nx">sigmoidGate</span><span class="p">.</span><span class="nx">prototype</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">forward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">u0</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u0</span> <span class="o">=</span> <span class="nx">u0</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">utop</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">sig</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">u0</span><span class="p">.</span><span class="nx">value</span><span class="p">),</span> <span class="mf">0.0</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">;</span>
  <span class="p">},</span>
  <span class="na">backward</span><span class="p">:</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="kd">var</span> <span class="nx">s</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">sig</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">u0</span><span class="p">.</span><span class="nx">value</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">u0</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="p">(</span><span class="nx">s</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="nx">s</span><span class="p">))</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">utop</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>

<p>Note that, again, the <code class="language-plaintext highlighter-rouge">backward</code> function in all cases just computes the local derivative with respect to its input and then multiplies on the gradient from the unit above (i.e. chain rule). To fully specify everything lets finally write out the forward and backward flow for our 2-dimensional neuron with some example values:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// create input units</span>
<span class="kd">var</span> <span class="nx">a</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="mf">1.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">b</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="mf">2.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">c</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="o">-</span><span class="mf">3.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="o">-</span><span class="mf">1.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">y</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="mf">3.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>

<span class="c1">// create the gates</span>
<span class="kd">var</span> <span class="nx">mulg0</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">multiplyGate</span><span class="p">();</span>
<span class="kd">var</span> <span class="nx">mulg1</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">multiplyGate</span><span class="p">();</span>
<span class="kd">var</span> <span class="nx">addg0</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">addGate</span><span class="p">();</span>
<span class="kd">var</span> <span class="nx">addg1</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">addGate</span><span class="p">();</span>
<span class="kd">var</span> <span class="nx">sg0</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">sigmoidGate</span><span class="p">();</span>

<span class="c1">// do the forward pass</span>
<span class="kd">var</span> <span class="nx">forwardNeuron</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  <span class="nx">ax</span> <span class="o">=</span> <span class="nx">mulg0</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">x</span><span class="p">);</span> <span class="c1">// a*x = -1</span>
  <span class="nx">by</span> <span class="o">=</span> <span class="nx">mulg1</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">b</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// b*y = 6</span>
  <span class="nx">axpby</span> <span class="o">=</span> <span class="nx">addg0</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">ax</span><span class="p">,</span> <span class="nx">by</span><span class="p">);</span> <span class="c1">// a*x + b*y = 5</span>
  <span class="nx">axpbypc</span> <span class="o">=</span> <span class="nx">addg1</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">axpby</span><span class="p">,</span> <span class="nx">c</span><span class="p">);</span> <span class="c1">// a*x + b*y + c = 2</span>
  <span class="nx">s</span> <span class="o">=</span> <span class="nx">sg0</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">axpbypc</span><span class="p">);</span> <span class="c1">// sig(a*x + b*y + c) = 0.8808</span>
<span class="p">};</span>
<span class="nx">forwardNeuron</span><span class="p">();</span>

<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">circuit output: </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">s</span><span class="p">.</span><span class="nx">value</span><span class="p">);</span> <span class="c1">// prints 0.8808</span>
</code></pre></div></div>

<p>And now lets compute the gradient: Simply iterate in reverse order and call the <code class="language-plaintext highlighter-rouge">backward</code> function! Remember that we stored the pointers to the units when we did the forward pass, so every gate has access to its inputs and also the output unit it previously produced.</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nx">s</span><span class="p">.</span><span class="nx">grad</span> <span class="o">=</span> <span class="mf">1.0</span><span class="p">;</span>
<span class="nx">sg0</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// writes gradient into axpbypc</span>
<span class="nx">addg1</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// writes gradients into axpby and c</span>
<span class="nx">addg0</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// writes gradients into ax and by</span>
<span class="nx">mulg1</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// writes gradients into b and y</span>
<span class="nx">mulg0</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// writes gradients into a and x</span>
</code></pre></div></div>

<p>Note that the first line sets the gradient at the output (very last unit) to be <code class="language-plaintext highlighter-rouge">1.0</code> to start off the gradient chain. This can be interpreted as tugging on the last gate with a force of <code class="language-plaintext highlighter-rouge">+1</code>. In other words, we are pulling on the entire circuit to induce the forces that will increase the output value. If we did not set this to 1, all gradients would be computed as zero due to the multiplications in the chain rule. Finally, lets make the inputs respond to the computed gradients and check that the function increased:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
<span class="nx">a</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">a</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span> <span class="c1">// a.grad is -0.105</span>
<span class="nx">b</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">b</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span> <span class="c1">// b.grad is 0.315</span>
<span class="nx">c</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">c</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span> <span class="c1">// c.grad is 0.105</span>
<span class="nx">x</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">x</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span> <span class="c1">// x.grad is 0.105</span>
<span class="nx">y</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">y</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span> <span class="c1">// y.grad is 0.210</span>

<span class="nx">forwardNeuron</span><span class="p">();</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">circuit output after one backprop: </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">s</span><span class="p">.</span><span class="nx">value</span><span class="p">);</span> <span class="c1">// prints 0.8825</span>
</code></pre></div></div>

<p>Success! <code class="language-plaintext highlighter-rouge">0.8825</code> is higher than the previous value, <code class="language-plaintext highlighter-rouge">0.8808</code>. Finally, lets verify that we implemented the backpropagation correctly by checking the numerical gradient:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">forwardCircuitFast</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">)</span> <span class="p">{</span> 
  <span class="k">return</span> <span class="mi">1</span><span class="o">/</span><span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">exp</span><span class="p">(</span> <span class="o">-</span> <span class="p">(</span><span class="nx">a</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c</span><span class="p">)));</span> 
<span class="p">};</span>
<span class="kd">var</span> <span class="nx">a</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="nx">b</span> <span class="o">=</span> <span class="mi">2</span><span class="p">,</span> <span class="nx">c</span> <span class="o">=</span> <span class="o">-</span><span class="mi">3</span><span class="p">,</span> <span class="nx">x</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="nx">y</span> <span class="o">=</span> <span class="mi">3</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">h</span> <span class="o">=</span> <span class="mf">0.0001</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">a_grad</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="o">+</span><span class="nx">h</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">))</span><span class="o">/</span><span class="nx">h</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">b_grad</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="o">+</span><span class="nx">h</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">))</span><span class="o">/</span><span class="nx">h</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">c_grad</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="o">+</span><span class="nx">h</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">))</span><span class="o">/</span><span class="nx">h</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x_grad</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="o">+</span><span class="nx">h</span><span class="p">,</span><span class="nx">y</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">))</span><span class="o">/</span><span class="nx">h</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">y_grad</span> <span class="o">=</span> <span class="p">(</span><span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="o">+</span><span class="nx">h</span><span class="p">)</span> <span class="o">-</span> <span class="nx">forwardCircuitFast</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">,</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">))</span><span class="o">/</span><span class="nx">h</span><span class="p">;</span>
</code></pre></div></div>

<p>Indeed, these all give the same values as the backpropagated gradients <code class="language-plaintext highlighter-rouge">[-0.105, 0.315, 0.105, 0.105, 0.210]</code>. Nice!</p>

<p>I hope it is clear that even though we only looked at an example of a single neuron, the code I gave above generalizes in a very straight-forward way to compute gradients of arbitrary expressions (including very deep expressions #foreshadowing). All you have to do is write small gates that compute local, simple derivatives w.r.t their inputs, wire it up in a graph, do a forward pass to compute the output value and then a backward pass that chains the gradients all the way to the input.</p>

<h3 id="becoming-a-backprop-ninja">Becoming a Backprop Ninja</h3>

<p>Over time you will become much more efficient in writing the backward pass, even for complicated circuits and all at once. Lets practice backprop a bit with a few examples. In what follows, lets not worry about Unit, Circuit classes because they obfuscate things a bit, and lets just use variables such as <code class="language-plaintext highlighter-rouge">a,b,c,x</code>, and refer to their gradients as <code class="language-plaintext highlighter-rouge">da,db,dc,dx</code> respectively. Again, we think of the variables as the “forward flow” and their gradients as “backward flow” along every wire. Our first example was the <code class="language-plaintext highlighter-rouge">*</code> gate:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">b</span><span class="p">;</span>
<span class="c1">// and given gradient on x (dx), we saw that in backprop we would compute:</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="nx">b</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
</code></pre></div></div>

<p>In the code above, I’m assuming that the variable <code class="language-plaintext highlighter-rouge">dx</code> is given, coming from somewhere above us in the circuit while we’re doing backprop (or it is +1 by default otherwise). I’m writing it out because I want to explicitly show how the gradients get chained together. Note from the equations that the <code class="language-plaintext highlighter-rouge">*</code> gate acts as a <em>switcher</em> during backward pass, for lack of better word. It remembers what its inputs were, and the gradients on each one will be the value of the other during the forward pass. And then of course we have to multiply with the gradient from above, which is the chain rule. Here’s the <code class="language-plaintext highlighter-rouge">+</code> gate in this condensed form:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">;</span>
<span class="c1">// -&gt;</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
</code></pre></div></div>

<p>Where <code class="language-plaintext highlighter-rouge">1.0</code> is the local gradient, and the multiplication is our chain rule. What about adding three numbers?:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// lets compute x = a + b + c in two steps:</span>
<span class="kd">var</span> <span class="nx">q</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">;</span> <span class="c1">// gate 1</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">q</span> <span class="o">+</span> <span class="nx">c</span><span class="p">;</span> <span class="c1">// gate 2</span>

<span class="c1">// backward pass:</span>
<span class="nx">dc</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> <span class="c1">// backprop gate 2</span>
<span class="nx">dq</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> 
<span class="nx">da</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span> <span class="c1">// backprop gate 1</span>
<span class="nx">db</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span>
</code></pre></div></div>

<p>You can see what’s happening, right? If you remember the backward flow diagram, the <code class="language-plaintext highlighter-rouge">+</code> gate simply takes the gradient on top and routes it equally to all of its inputs (because its local gradient is always simply <code class="language-plaintext highlighter-rouge">1.0</code> for all its inputs, regardless of their actual values). So we can do it much faster:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span> <span class="o">+</span> <span class="nx">c</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> <span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> <span class="kd">var</span> <span class="nx">dc</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
</code></pre></div></div>

<p>Okay, how about combining gates?:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">b</span> <span class="o">+</span> <span class="nx">c</span><span class="p">;</span>
<span class="c1">// given dx, backprop in-one-sweep would be =&gt;</span>
<span class="nx">da</span> <span class="o">=</span> <span class="nx">b</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
<span class="nx">db</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
<span class="nx">dc</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
</code></pre></div></div>

<p>If you don’t see how the above happened, introduce a temporary variable <code class="language-plaintext highlighter-rouge">q = a * b</code> and then compute <code class="language-plaintext highlighter-rouge">x = q + c</code> to convince yourself. And here is our neuron, lets do it in two steps:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// lets do our neuron in two steps:</span>
<span class="kd">var</span> <span class="nx">q</span> <span class="o">=</span> <span class="nx">a</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">f</span> <span class="o">=</span> <span class="nx">sig</span><span class="p">(</span><span class="nx">q</span><span class="p">);</span> <span class="c1">// sig is the sigmoid function</span>
<span class="c1">// and now backward pass, we are given df, and:</span>
<span class="kd">var</span> <span class="nx">df</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dq</span> <span class="o">=</span> <span class="p">(</span><span class="nx">f</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="nx">f</span><span class="p">))</span> <span class="o">*</span> <span class="nx">df</span><span class="p">;</span>
<span class="c1">// and now we chain it to the inputs</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dx</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dy</span> <span class="o">=</span> <span class="nx">b</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dc</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dq</span><span class="p">;</span>
</code></pre></div></div>

<p>I hope this is starting to make a little more sense. Now how about this:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">a</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="c1">//???</span>
</code></pre></div></div>

<p>You can think of this as value <code class="language-plaintext highlighter-rouge">a</code> flowing to the <code class="language-plaintext highlighter-rouge">*</code> gate, but the wire gets split and becomes both inputs. This is actually simple because the backward flow of gradients always adds up. In other words nothing changes:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> <span class="c1">// gradient into a from first branch</span>
<span class="nx">da</span> <span class="o">+=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> <span class="c1">// and add on the gradient from the second branch</span>

<span class="c1">// short form instead is:</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="mi">2</span> <span class="o">*</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
</code></pre></div></div>

<p>In fact, if you know your power rule from calculus you would also know that if you have \( f(a) = a^2 \) then \( \frac{\partial f(a)}{\partial a} = 2a \), which is exactly what we get if we think of it as wire splitting up and being two inputs to a gate.</p>

<p>Lets do another one:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">a</span><span class="o">*</span><span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="o">*</span><span class="nx">b</span> <span class="o">+</span> <span class="nx">c</span><span class="o">*</span><span class="nx">c</span><span class="p">;</span>
<span class="c1">// we get:</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="mi">2</span><span class="o">*</span><span class="nx">a</span><span class="o">*</span><span class="nx">dx</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="mi">2</span><span class="o">*</span><span class="nx">b</span><span class="o">*</span><span class="nx">dx</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dc</span> <span class="o">=</span> <span class="mi">2</span><span class="o">*</span><span class="nx">c</span><span class="o">*</span><span class="nx">dx</span><span class="p">;</span>
</code></pre></div></div>

<p>Okay now lets start to get more complex:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">pow</span><span class="p">(((</span><span class="nx">a</span> <span class="o">*</span> <span class="nx">b</span> <span class="o">+</span> <span class="nx">c</span><span class="p">)</span> <span class="o">*</span> <span class="nx">d</span><span class="p">),</span> <span class="mi">2</span><span class="p">);</span> <span class="c1">// pow(x,2) squares the input JS</span>
</code></pre></div></div>

<p>When more complex cases like this come up in practice, I like to split the expression into manageable chunks which are almost always composed of simpler expressions and then I chain them together with chain rule:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x1</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">b</span> <span class="o">+</span> <span class="nx">c</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x2</span> <span class="o">=</span> <span class="nx">x1</span> <span class="o">*</span> <span class="nx">d</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">x2</span> <span class="o">*</span> <span class="nx">x2</span><span class="p">;</span> <span class="c1">// this is identical to the above expression for x</span>
<span class="c1">// and now in backprop we go backwards:</span>
<span class="kd">var</span> <span class="nx">dx2</span> <span class="o">=</span> <span class="mi">2</span> <span class="o">*</span> <span class="nx">x2</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span> <span class="c1">// backprop into x2</span>
<span class="kd">var</span> <span class="nx">dd</span> <span class="o">=</span> <span class="nx">x1</span> <span class="o">*</span> <span class="nx">dx2</span><span class="p">;</span> <span class="c1">// backprop into d</span>
<span class="kd">var</span> <span class="nx">dx1</span> <span class="o">=</span> <span class="nx">d</span> <span class="o">*</span> <span class="nx">dx2</span><span class="p">;</span> <span class="c1">// backprop into x1</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="nx">b</span> <span class="o">*</span> <span class="nx">dx1</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">dx1</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dc</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx1</span><span class="p">;</span> <span class="c1">// done!</span>
</code></pre></div></div>

<p>That wasn’t too difficult! Those are the backprop equations for the entire expression, and we’ve done them piece by piece and backpropped to all the variables. Notice again how for every variable during forward pass we have an equivalent variable during backward pass that contains its gradient with respect to the circuit’s final output. Here are a few more useful functions and their local gradients that are useful in practice:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="mf">1.0</span><span class="o">/</span><span class="nx">a</span><span class="p">;</span> <span class="c1">// division</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="o">-</span><span class="mf">1.0</span><span class="o">/</span><span class="p">(</span><span class="nx">a</span><span class="o">*</span><span class="nx">a</span><span class="p">);</span>
</code></pre></div></div>

<p>Here’s what division might look like in practice then:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="p">(</span><span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">)</span><span class="o">/</span><span class="p">(</span><span class="nx">c</span> <span class="o">+</span> <span class="nx">d</span><span class="p">);</span>
<span class="c1">// lets decompose it in steps:</span>
<span class="kd">var</span> <span class="nx">x1</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x2</span> <span class="o">=</span> <span class="nx">c</span> <span class="o">+</span> <span class="nx">d</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x3</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">/</span> <span class="nx">x2</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">x1</span> <span class="o">*</span> <span class="nx">x3</span><span class="p">;</span> <span class="c1">// equivalent to above</span>
<span class="c1">// and now backprop, again in reverse order:</span>
<span class="kd">var</span> <span class="nx">dx1</span> <span class="o">=</span> <span class="nx">x3</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dx3</span> <span class="o">=</span> <span class="nx">x1</span> <span class="o">*</span> <span class="nx">dx</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dx2</span> <span class="o">=</span> <span class="p">(</span><span class="o">-</span><span class="mf">1.0</span><span class="o">/</span><span class="p">(</span><span class="nx">x2</span><span class="o">*</span><span class="nx">x2</span><span class="p">))</span> <span class="o">*</span> <span class="nx">dx3</span><span class="p">;</span> <span class="c1">// local gradient as shown above, and chain rule</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx1</span><span class="p">;</span> <span class="c1">// and finally into the original variables</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx1</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dc</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx2</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">dd</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx2</span><span class="p">;</span>
</code></pre></div></div>

<p>Hopefully you see that we are breaking down expressions, doing the forward pass, and then for every variable (such as <code class="language-plaintext highlighter-rouge">a</code>) we derive its gradient <code class="language-plaintext highlighter-rouge">da</code> as we go backwards, one by one, applying the simple local gradients and chaining them with gradients from above. Here’s another one:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">===</span> <span class="nx">x</span> <span class="p">?</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span> <span class="p">:</span> <span class="mf">0.0</span><span class="p">;</span>
<span class="kd">var</span> <span class="nx">db</span> <span class="o">=</span> <span class="nx">b</span> <span class="o">===</span> <span class="nx">x</span> <span class="p">?</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span> <span class="p">:</span> <span class="mf">0.0</span><span class="p">;</span>
</code></pre></div></div>

<p>Okay this is making a very simple thing hard to read. The <code class="language-plaintext highlighter-rouge">max</code> function passes on the value of the input that was largest and ignores the other ones. In the backward pass then, the max gate will simply take the gradient on top and route it to the input that actually flowed through it during the forward pass. The gate acts as a simple switch based on which input had the highest value during forward pass. The other inputs will have zero gradient. That’s what the <code class="language-plaintext highlighter-rouge">===</code> is about, since we are testing for which input was the actual max and only routing the gradient to it.</p>

<p>Finally, lets look at the Rectified Linear Unit non-linearity (or ReLU), which you may have heard of. It is used in Neural Networks in place of the sigmoid function. It is simply thresholding at zero:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span>
<span class="c1">// backprop through this gate will then be:</span>
<span class="kd">var</span> <span class="nx">da</span> <span class="o">=</span> <span class="nx">a</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dx</span> <span class="p">:</span> <span class="mf">0.0</span><span class="p">;</span>
</code></pre></div></div>

<p>In other words this gate simply passes the value through if it’s larger than 0, or it stops the flow and sets it to zero. In the backward pass, the gate will pass on the gradient from the top if it was activated during the forawrd pass, or if the original input was below zero, it will stop the gradient flow.</p>

<p>I will stop at this point. I hope you got some intuition about how you can compute entire expressions (which are made up of many gates along the way) and how you can compute backprop for every one of them.</p>

<p>Everything we’ve done in this chapter comes down to this: We saw that we can feed some input through arbitrarily complex real-valued circuit, tug at the end of the circuit with some force, and backpropagation distributes that tug through the entire circuit all the way back to the inputs. If the inputs respond slightly along the final direction of their tug, the circuit will “give” a bit along the original pull direction. Maybe this is not immediately obvious, but this machinery is a powerful <em>hammer</em> for Machine Learning.</p>

<blockquote>
  <p>“Maybe this is not immediately obvious, but this machinery is a powerful <em>hammer</em> for Machine Learning.”</p>
</blockquote>

<p>Lets now put this machinery to good use.</p>

<h2 id="chapter-2-machine-learning">Chapter 2: Machine Learning</h2>

<p>In the last chapter we were concerned with real-valued circuits that computed possibly complex expressions of their inputs (the forward pass), and also we could compute the gradients of these expressions on the original inputs (backward pass). In this chapter we will see how useful this extremely simple mechanism is in Machine Learning.</p>

<h3 id="binary-classification">Binary Classification</h3>

<p>As we did before, lets start out simple. The simplest, common and yet very practical problem in Machine Learning is <strong>binary classification</strong>. A lot of very interesting and important problems can be reduced to it. The setup is as follows: We are given a dataset of <code class="language-plaintext highlighter-rouge">N</code> vectors and every one of them is labeled with a <code class="language-plaintext highlighter-rouge">+1</code> or a <code class="language-plaintext highlighter-rouge">-1</code>. For example, in two dimensions our dataset could look as simple as:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>vector -&gt; label
---------------
[1.2, 0.7] -&gt; +1
[-0.3, 0.5] -&gt; -1
[-3, -1] -&gt; +1
[0.1, 1.0] -&gt; -1
[3.0, 1.1] -&gt; -1
[2.1, -3] -&gt; +1
</code></pre></div></div>

<p>Here, we have <code class="language-plaintext highlighter-rouge">N = 6</code> <strong>datapoints</strong>, where every datapoint has two <strong>features</strong> (<code class="language-plaintext highlighter-rouge">D = 2</code>). Three of the datapoints have <strong>label</strong> <code class="language-plaintext highlighter-rouge">+1</code> and the other three label <code class="language-plaintext highlighter-rouge">-1</code>. This is a silly toy example, but in practice a +1/-1 dataset could be very useful things indeed: For example spam/no spam emails, where the vectors somehow measure various features of the content of the email, such as the number of times certain enhancement drugs are mentioned.</p>

<p><strong>Goal</strong>. Our goal in binary classification is to learn a function that takes a 2-dimensional vector and predicts the label. This function is usually parameterized by a certain set of parameters, and we will want to tune the parameters of the function so that its outputs are consistent with the labeling in the provided dataset. In the end we can discard the dataset and use the learned parameters to predict labels for previously unseen vectors.</p>

<h4 id="training-protocol">Training protocol</h4>

<p>We will eventually build up to entire neural networks and complex expressions, but lets start out simple and train a linear classifier very similar to the single neuron we saw at the end of Chapter 1. The only difference is that we’ll get rid of the sigmoid because it makes things unnecessarily complicated (I only used it as an example in Chapter 1 because sigmoid neurons are historically popular but modern Neural Networks rarely, if ever, use sigmoid non-linearities). Anyway, lets use a simple linear function:</p>

\[f(x, y) = ax + by + c\]

<p>In this expression we think of <code class="language-plaintext highlighter-rouge">x</code> and <code class="language-plaintext highlighter-rouge">y</code> as the inputs (the 2D vectors) and <code class="language-plaintext highlighter-rouge">a,b,c</code> as the parameters of the function that we will want to learn. For example, if <code class="language-plaintext highlighter-rouge">a = 1, b = -2, c = -1</code>, then the function will take the first datapoint (<code class="language-plaintext highlighter-rouge">[1.2, 0.7]</code>) and output <code class="language-plaintext highlighter-rouge">1 * 1.2 + (-2) * 0.7 + (-1) = -1.2</code>. Here is how the training will work:</p>

<ol>
  <li>We select a random datapoint and feed it through the circuit</li>
  <li>We will interpret the output of the circuit as a confidence that the datapoint has class <code class="language-plaintext highlighter-rouge">+1</code>. (i.e. very high values = circuit is very certain datapoint has class <code class="language-plaintext highlighter-rouge">+1</code> and very low values = circuit is certain this datapoint has class <code class="language-plaintext highlighter-rouge">-1</code>.)</li>
  <li>We will measure how well the prediction aligns with the provided labels. Intuitively, for example, if a positive example scores very low, we will want to tug in the positive direction on the circuit, demanding that it should output higher value for this datapoint. Note that this is the case for the the first datapoint: it is labeled as <code class="language-plaintext highlighter-rouge">+1</code> but our predictor unction only assigns it value <code class="language-plaintext highlighter-rouge">-1.2</code>. We will therefore tug on the circuit in positive direction; We want the value to be higher.</li>
  <li>The circuit will take the tug and backpropagate it to compute tugs on the inputs <code class="language-plaintext highlighter-rouge">a,b,c,x,y</code></li>
  <li>Since we think of <code class="language-plaintext highlighter-rouge">x,y</code> as (fixed) datapoints, we will ignore the pull on <code class="language-plaintext highlighter-rouge">x,y</code>. If you’re a fan of my physical analogies, think of these inputs as pegs, fixed in the ground.</li>
  <li>On the other hand, we will take the parameters <code class="language-plaintext highlighter-rouge">a,b,c</code> and make them respond to their tug (i.e. we’ll perform what we call a <strong>parameter update</strong>). This, of course, will make it so that the circuit will output a slightly higher score on this particular datapoint in the future.</li>
  <li>Iterate! Go back to step 1.</li>
</ol>

<p>The training scheme I described above, is commonly referred as <strong>Stochastic Gradient Descent</strong>. The interesting part I’d like to reiterate is that <code class="language-plaintext highlighter-rouge">a,b,c,x,y</code> are all made up of the same <em>stuff</em> as far as the circuit is concerned: They are inputs to the circuit and the circuit will tug on all of them in some direction. It doesn’t know the difference between parameters and datapoints. However, after the backward pass is complete we ignore all tugs on the datapoints (<code class="language-plaintext highlighter-rouge">x,y</code>) and keep swapping them in and out as we iterate over examples in the dataset. On the other hand, we keep the parameters (<code class="language-plaintext highlighter-rouge">a,b,c</code>) around and keep tugging on them every time we sample a datapoint. Over time, the pulls on these parameters will tune these values in such a way that the function outputs high scores for positive examples and low scores for negative examples.</p>

<h4 id="learning-a-support-vector-machine">Learning a Support Vector Machine</h4>

<p>As a concrete example, lets learn a <strong>Support Vector Machine</strong>. The SVM is a very popular linear classifier; Its functional form is exactly as I’ve described in previous section, \( f(x,y) = ax + by + c\). At this point, if you’ve seen an explanation of SVMs you’re probably expecting me to define the SVM loss function and plunge into an explanation of slack variables, geometrical intuitions of large margins, kernels, duality, etc. But here, I’d like to take a different approach. Instead of definining loss functions, I would like to base the explanation on the <em>force specification</em> (I just made this term up by the way) of a Support Vector Machine, which I personally find much more intuitive. As we will see, talking about the force specification and the loss function are identical ways of seeing the same problem. Anyway, here it is:</p>

<p><strong>Support Vector Machine “Force Specification”:</strong></p>

<ul>
  <li>If we feed a positive datapoint through the SVM circuit and the output value is less than 1, pull on the circuit with force <code class="language-plaintext highlighter-rouge">+1</code>. This is a positive example so we want the score to be higher for it.</li>
  <li>Conversely, if we feed a negative datapoint through the SVM and the output is greater than -1, then the circuit is giving this datapoint dangerously high score: Pull on the circuit downwards with force <code class="language-plaintext highlighter-rouge">-1</code>.</li>
  <li>In addition to the pulls above, always add a small amount of pull on the parameters <code class="language-plaintext highlighter-rouge">a,b</code> (notice, not on <code class="language-plaintext highlighter-rouge">c</code>!) that pulls them towards zero. You can think of both <code class="language-plaintext highlighter-rouge">a,b</code> as being attached to a physical spring that is attached at zero. Just as with a physical spring, this will make the pull proprotional to the value of each of <code class="language-plaintext highlighter-rouge">a,b</code> (Hooke’s law in physics, anyone?). For example, if <code class="language-plaintext highlighter-rouge">a</code> becomes very high it will experience a strong pull of magnitude <code class="language-plaintext highlighter-rouge">|a|</code> back towards zero. This pull is something we call <strong>regularization</strong>, and it ensures that neither of our parameters <code class="language-plaintext highlighter-rouge">a</code> or <code class="language-plaintext highlighter-rouge">b</code> gets disproportionally large. This would be undesirable because both <code class="language-plaintext highlighter-rouge">a,b</code> get multiplied to the input features <code class="language-plaintext highlighter-rouge">x,y</code> (remember the equation is <code class="language-plaintext highlighter-rouge">a*x + b*y + c</code>), so if either of them is too high, our classifier would be overly sensitive to these features. This isn’t a nice property because features can often be noisy in practice, so we want our classifier to change relatively smoothly if they wiggle around.</li>
</ul>

<p>Lets quickly go through a small but concrete example. Suppose we start out with a random parameter setting, say, <code class="language-plaintext highlighter-rouge">a = 1, b = -2, c = -1</code>. Then:</p>

<ul>
  <li>If we feed the point <code class="language-plaintext highlighter-rouge">[1.2, 0.7]</code>, the SVM will compute score <code class="language-plaintext highlighter-rouge">1 * 1.2 + (-2) * 0.7 - 1 = -1.2</code>. This point is labeled as <code class="language-plaintext highlighter-rouge">+1</code> in the training data, so we want the score to be higher than 1. The gradient on top of the circuit will thus be positive: <code class="language-plaintext highlighter-rouge">+1</code>, which will backpropagate to <code class="language-plaintext highlighter-rouge">a,b,c</code>. Additionally, there will also be a regularization pull on <code class="language-plaintext highlighter-rouge">a</code> of <code class="language-plaintext highlighter-rouge">-1</code> (to make it smaller) and regularization pull on <code class="language-plaintext highlighter-rouge">b</code> of <code class="language-plaintext highlighter-rouge">+2</code> to make it larger, toward zero.</li>
  <li>Suppose instead that we fed the datapoint <code class="language-plaintext highlighter-rouge">[-0.3, 0.5]</code> to the SVM. It computes <code class="language-plaintext highlighter-rouge">1 * (-0.3) + (-2) * 0.5 - 1 = -2.3</code>. The label for this point is <code class="language-plaintext highlighter-rouge">-1</code>, and since <code class="language-plaintext highlighter-rouge">-2.3</code> is smaller than <code class="language-plaintext highlighter-rouge">-1</code>, we see that according to our force specification the SVM should be happy: The computed score is very negative, consistent with the negative label of this example. There will be no pull at the end of the circuit (i.e it’s zero), since there no changes are necessary. However, there will <em>still</em> be the regularization pull on <code class="language-plaintext highlighter-rouge">a</code> of <code class="language-plaintext highlighter-rouge">-1</code> and on <code class="language-plaintext highlighter-rouge">b</code> of <code class="language-plaintext highlighter-rouge">+2</code>.</li>
</ul>

<p>Okay there’s been too much text. Lets write the SVM code and take advantage of the circuit machinery we have from Chapter 1:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// A circuit: it takes 5 Units (x,y,a,b,c) and outputs a single Unit</span>
<span class="c1">// It can also compute the gradient w.r.t. its inputs</span>
<span class="kd">var</span> <span class="nx">Circuit</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  <span class="c1">// create some gates</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">mulg0</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">multiplyGate</span><span class="p">();</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">mulg1</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">multiplyGate</span><span class="p">();</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">addg0</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">addGate</span><span class="p">();</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">addg1</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">addGate</span><span class="p">();</span>
<span class="p">};</span>
<span class="nx">Circuit</span><span class="p">.</span><span class="nx">prototype</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">forward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">,</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">,</span><span class="nx">c</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">ax</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">mulg0</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">x</span><span class="p">);</span> <span class="c1">// a*x</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">by</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">mulg1</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">b</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// b*y</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">axpby</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">addg0</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ax</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">by</span><span class="p">);</span> <span class="c1">// a*x + b*y</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">axpbypc</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">addg1</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">axpby</span><span class="p">,</span> <span class="nx">c</span><span class="p">);</span> <span class="c1">// a*x + b*y + c</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">axpbypc</span><span class="p">;</span>
  <span class="p">},</span>
  <span class="na">backward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">gradient_top</span><span class="p">)</span> <span class="p">{</span> <span class="c1">// takes pull from above</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">axpbypc</span><span class="p">.</span><span class="nx">grad</span> <span class="o">=</span> <span class="nx">gradient_top</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">addg1</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// sets gradient in axpby and c</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">addg0</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// sets gradient in ax and by</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">mulg1</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// sets gradient in b and y</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">mulg0</span><span class="p">.</span><span class="nx">backward</span><span class="p">();</span> <span class="c1">// sets gradient in a and x</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>

<p>That’s a circuit that simply computes <code class="language-plaintext highlighter-rouge">a*x + b*y + c</code> and can also compute the gradient. It uses the gates code we developed in Chapter 1. Now lets write the SVM, which doesn’t care about the actual circuit. It is only concerned with the values that come out of it, and it pulls on the circuit.</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// SVM class</span>
<span class="kd">var</span> <span class="nx">SVM</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  
  <span class="c1">// random initial parameter values</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">a</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="mf">1.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span> 
  <span class="k">this</span><span class="p">.</span><span class="nx">b</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="o">-</span><span class="mf">2.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>
  <span class="k">this</span><span class="p">.</span><span class="nx">c</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="o">-</span><span class="mf">1.0</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">);</span>

  <span class="k">this</span><span class="p">.</span><span class="nx">circuit</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Circuit</span><span class="p">();</span>
<span class="p">};</span>
<span class="nx">SVM</span><span class="p">.</span><span class="nx">prototype</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">forward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">)</span> <span class="p">{</span> <span class="c1">// assume x and y are Units</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">unit_out</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">circuit</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">a</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">b</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">c</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">unit_out</span><span class="p">;</span>
  <span class="p">},</span>
  <span class="na">backward</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">label</span><span class="p">)</span> <span class="p">{</span> <span class="c1">// label is +1 or -1</span>

    <span class="c1">// reset pulls on a,b,c</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">a</span><span class="p">.</span><span class="nx">grad</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span> 
    <span class="k">this</span><span class="p">.</span><span class="nx">b</span><span class="p">.</span><span class="nx">grad</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span> 
    <span class="k">this</span><span class="p">.</span><span class="nx">c</span><span class="p">.</span><span class="nx">grad</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>

    <span class="c1">// compute the pull based on what the circuit output was</span>
    <span class="kd">var</span> <span class="nx">pull</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>
    <span class="k">if</span><span class="p">(</span><span class="nx">label</span> <span class="o">===</span> <span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="k">this</span><span class="p">.</span><span class="nx">unit_out</span><span class="p">.</span><span class="nx">value</span> <span class="o">&lt;</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span> 
      <span class="nx">pull</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span> <span class="c1">// the score was too low: pull up</span>
    <span class="p">}</span>
    <span class="k">if</span><span class="p">(</span><span class="nx">label</span> <span class="o">===</span> <span class="o">-</span><span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="k">this</span><span class="p">.</span><span class="nx">unit_out</span><span class="p">.</span><span class="nx">value</span> <span class="o">&gt;</span> <span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">pull</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="p">;</span> <span class="c1">// the score was too high for a positive example, pull down</span>
    <span class="p">}</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">circuit</span><span class="p">.</span><span class="nx">backward</span><span class="p">(</span><span class="nx">pull</span><span class="p">);</span> <span class="c1">// writes gradient into x,y,a,b,c</span>
    
    <span class="c1">// add regularization pull for parameters: towards zero and proportional to value</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">a</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="o">-</span><span class="k">this</span><span class="p">.</span><span class="nx">a</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">b</span><span class="p">.</span><span class="nx">grad</span> <span class="o">+=</span> <span class="o">-</span><span class="k">this</span><span class="p">.</span><span class="nx">b</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
  <span class="p">},</span>
  <span class="na">learnFrom</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="nx">label</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">);</span> <span class="c1">// forward pass (set .value in all Units)</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">backward</span><span class="p">(</span><span class="nx">label</span><span class="p">);</span> <span class="c1">// backward pass (set .grad in all Units)</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">parameterUpdate</span><span class="p">();</span> <span class="c1">// parameters respond to tug</span>
  <span class="p">},</span>
  <span class="na">parameterUpdate</span><span class="p">:</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">a</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">a</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">b</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">b</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">c</span><span class="p">.</span><span class="nx">value</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="k">this</span><span class="p">.</span><span class="nx">c</span><span class="p">.</span><span class="nx">grad</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre></div></div>

<p>Now lets train the SVM with Stochastic Gradient Descent:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">data</span> <span class="o">=</span> <span class="p">[];</span> <span class="kd">var</span> <span class="nx">labels</span> <span class="o">=</span> <span class="p">[];</span>
<span class="nx">data</span><span class="p">.</span><span class="nx">push</span><span class="p">([</span><span class="mf">1.2</span><span class="p">,</span> <span class="mf">0.7</span><span class="p">]);</span> <span class="nx">labels</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
<span class="nx">data</span><span class="p">.</span><span class="nx">push</span><span class="p">([</span><span class="o">-</span><span class="mf">0.3</span><span class="p">,</span> <span class="o">-</span><span class="mf">0.5</span><span class="p">]);</span> <span class="nx">labels</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>
<span class="nx">data</span><span class="p">.</span><span class="nx">push</span><span class="p">([</span><span class="mf">3.0</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">]);</span> <span class="nx">labels</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
<span class="nx">data</span><span class="p">.</span><span class="nx">push</span><span class="p">([</span><span class="o">-</span><span class="mf">0.1</span><span class="p">,</span> <span class="o">-</span><span class="mf">1.0</span><span class="p">]);</span> <span class="nx">labels</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>
<span class="nx">data</span><span class="p">.</span><span class="nx">push</span><span class="p">([</span><span class="o">-</span><span class="mf">1.0</span><span class="p">,</span> <span class="mf">1.1</span><span class="p">]);</span> <span class="nx">labels</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>
<span class="nx">data</span><span class="p">.</span><span class="nx">push</span><span class="p">([</span><span class="mf">2.1</span><span class="p">,</span> <span class="o">-</span><span class="mi">3</span><span class="p">]);</span> <span class="nx">labels</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="mi">1</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">svm</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">SVM</span><span class="p">();</span>

<span class="c1">// a function that computes the classification accuracy</span>
<span class="kd">var</span> <span class="nx">evalTrainingAccuracy</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">var</span> <span class="nx">num_correct</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
  <span class="k">for</span><span class="p">(</span><span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="mf">0.0</span><span class="p">);</span>
    <span class="kd">var</span> <span class="nx">y</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">],</span> <span class="mf">0.0</span><span class="p">);</span>
    <span class="kd">var</span> <span class="nx">true_label</span> <span class="o">=</span> <span class="nx">labels</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>

    <span class="c1">// see if the prediction matches the provided label</span>
    <span class="kd">var</span> <span class="nx">predicted_label</span> <span class="o">=</span> <span class="nx">svm</span><span class="p">.</span><span class="nx">forward</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">).</span><span class="nx">value</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="mi">1</span> <span class="p">:</span> <span class="o">-</span><span class="mi">1</span><span class="p">;</span>
    <span class="k">if</span><span class="p">(</span><span class="nx">predicted_label</span> <span class="o">===</span> <span class="nx">true_label</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">num_correct</span><span class="o">++</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="nx">num_correct</span> <span class="o">/</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
<span class="p">};</span>

<span class="c1">// the learning loop</span>
<span class="k">for</span><span class="p">(</span><span class="kd">var</span> <span class="nx">iter</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">iter</span> <span class="o">&lt;</span> <span class="mi">400</span><span class="p">;</span> <span class="nx">iter</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// pick a random data point</span>
  <span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()</span> <span class="o">*</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
  <span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="mf">0.0</span><span class="p">);</span>
  <span class="kd">var</span> <span class="nx">y</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Unit</span><span class="p">(</span><span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">],</span> <span class="mf">0.0</span><span class="p">);</span>
  <span class="kd">var</span> <span class="nx">label</span> <span class="o">=</span> <span class="nx">labels</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>
  <span class="nx">svm</span><span class="p">.</span><span class="nx">learnFrom</span><span class="p">(</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="nx">label</span><span class="p">);</span>

  <span class="k">if</span><span class="p">(</span><span class="nx">iter</span> <span class="o">%</span> <span class="mi">25</span> <span class="o">==</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span> <span class="c1">// every 10 iterations... </span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">training accuracy at iter </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">iter</span> <span class="o">+</span> <span class="dl">'</span><span class="s1">: </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">evalTrainingAccuracy</span><span class="p">());</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>
<p>This code prints the following output:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>training accuracy at iteration 0: 0.3333333333333333
training accuracy at iteration 25: 0.3333333333333333
training accuracy at iteration 50: 0.5
training accuracy at iteration 75: 0.5
training accuracy at iteration 100: 0.3333333333333333
training accuracy at iteration 125: 0.5
training accuracy at iteration 150: 0.5
training accuracy at iteration 175: 0.5
training accuracy at iteration 200: 0.5
training accuracy at iteration 225: 0.6666666666666666
training accuracy at iteration 250: 0.6666666666666666
training accuracy at iteration 275: 0.8333333333333334
training accuracy at iteration 300: 1
training accuracy at iteration 325: 1
training accuracy at iteration 350: 1
training accuracy at iteration 375: 1 
</code></pre></div></div>

<p>We see that initially our classifier only had 33% training accuracy, but by the end all training examples are correctly classifier as the parameters <code class="language-plaintext highlighter-rouge">a,b,c</code> adjusted their values according to the pulls we exerted. We just trained an SVM! But please don’t use this code anywhere in production :) We will see how we can make things much more efficient once we understand what is going on at the core.</p>

<p><strong>Number of iterations needed</strong>. With this example data, with this example initialization, and with the setting of step size we used, it took about 300 iterations to train the SVM. In practice, this could be many more or many less depending on how hard or large the problem is, how you’re initializating, normalizing your data, what step size you’re using, and so on. This is just a toy demonstration, but later we will go over all the best practices for actually training these classifiers in practice. For example, it will turn out that the setting of the step size is very imporant and tricky. Small step size will make your model slow to train. Large step size will train faster, but if it is too large, it will make your classifier chaotically jump around and not converge to a good final result. We will eventually use witheld validation data to properly tune it to be just in the sweet spot for your particular data.</p>

<p>One thing I’d like you to appreciate is that the circuit can be arbitrary expression, not just the linear prediction function we used in this example. For example, it can be an entire neural network.</p>

<p>By the way, I intentionally structured the code in a modular way, but we could have trained an SVM with a much simpler code. Here is really what all of these classes and computations boil down to:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">a</span> <span class="o">=</span> <span class="mi">1</span><span class="p">,</span> <span class="nx">b</span> <span class="o">=</span> <span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="nx">c</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="p">;</span> <span class="c1">// initial parameters</span>
<span class="k">for</span><span class="p">(</span><span class="kd">var</span> <span class="nx">iter</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">iter</span> <span class="o">&lt;</span> <span class="mi">400</span><span class="p">;</span> <span class="nx">iter</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// pick a random data point</span>
  <span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()</span> <span class="o">*</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
  <span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">];</span>
  <span class="kd">var</span> <span class="nx">y</span> <span class="o">=</span> <span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
  <span class="kd">var</span> <span class="nx">label</span> <span class="o">=</span> <span class="nx">labels</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>

  <span class="c1">// compute pull</span>
  <span class="kd">var</span> <span class="nx">score</span> <span class="o">=</span> <span class="nx">a</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">pull</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>
  <span class="k">if</span><span class="p">(</span><span class="nx">label</span> <span class="o">===</span> <span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="nx">score</span> <span class="o">&lt;</span> <span class="mi">1</span><span class="p">)</span> <span class="nx">pull</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span>
  <span class="k">if</span><span class="p">(</span><span class="nx">label</span> <span class="o">===</span> <span class="o">-</span><span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="nx">score</span> <span class="o">&gt;</span> <span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="nx">pull</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="p">;</span>

  <span class="c1">// compute gradient and update parameters</span>
  <span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
  <span class="nx">a</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="p">(</span><span class="nx">x</span> <span class="o">*</span> <span class="nx">pull</span> <span class="o">-</span> <span class="nx">a</span><span class="p">);</span> <span class="c1">// -a is from the regularization</span>
  <span class="nx">b</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="p">(</span><span class="nx">y</span> <span class="o">*</span> <span class="nx">pull</span> <span class="o">-</span> <span class="nx">b</span><span class="p">);</span> <span class="c1">// -b is from the regularization</span>
  <span class="nx">c</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">*</span> <span class="nx">pull</span><span class="p">);</span>
<span class="p">}</span>
</code></pre></div></div>

<p>this code gives an identical result. Perhaps by now you can glance at the code and see how these equations came about.</p>

<p><strong>Variable pull?</strong> A quick note to make at this point: You may have noticed that the pull is always 1,0, or -1. You could imagine doing other things, for example making this pull proportional to how bad the mistake was. This leads to a variation on the SVM that some people refer to as <em>squared hinge loss</em> SVM, for reasons that will later become clear. Depending on various features of your dataset, that may work better or worse. For example, if you have very bad outliers in your data, e.g. a negative data point that gets a score <code class="language-plaintext highlighter-rouge">+100</code>, its influence will be relatively minor on our classifier because we will only pull with force of <code class="language-plaintext highlighter-rouge">-1</code> regardless of how bad the mistake was. In practice we refer to this property of a classifier as <strong>robustness</strong> to outliers.</p>

<p>Lets <strong>recap</strong>. We introduced the <strong>binary classification</strong> problem, where we are given N D-dimensional vectors and a label +1/-1 for each. We saw that we can combine these features with a set of parameters inside a real-valued circuit (such as a <strong>Support Vector Machine</strong> circuit in our example). Then, we can repeatedly pass our data through the circuit and each time tweak the parameters so that the circuit’s output value is consistent with the provided labels. The tweaking relied, crucially, on our ability to <strong>backpropagate</strong> gradients through the circuit. In the end, the final circuit can be used to predict values for unseen instances!</p>

<h4 id="generalizing-the-svm-into-a-neural-network">Generalizing the SVM into a Neural Network</h4>

<p>Of interest is the fact that an SVM is just a particular type of a very simple circuit (circuit that computes <code class="language-plaintext highlighter-rouge">score = a*x + b*y + c</code> where <code class="language-plaintext highlighter-rouge">a,b,c</code> are weights and <code class="language-plaintext highlighter-rouge">x,y</code> are data points). This can be easily extended to more complicated functions. For example, lets write a 2-layer Neural Network that does the binary classification. The forward pass will look like this:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// assume inputs x,y</span>
<span class="kd">var</span> <span class="nx">n1</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">a1</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b1</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c1</span><span class="p">);</span> <span class="c1">// activation of 1st hidden neuron</span>
<span class="kd">var</span> <span class="nx">n2</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">a2</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b2</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c2</span><span class="p">);</span> <span class="c1">// 2nd neuron</span>
<span class="kd">var</span> <span class="nx">n3</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">a3</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b3</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c3</span><span class="p">);</span> <span class="c1">// 3rd neuron</span>
<span class="kd">var</span> <span class="nx">score</span> <span class="o">=</span> <span class="nx">a4</span><span class="o">*</span><span class="nx">n1</span> <span class="o">+</span> <span class="nx">b4</span><span class="o">*</span><span class="nx">n2</span> <span class="o">+</span> <span class="nx">c4</span><span class="o">*</span><span class="nx">n3</span> <span class="o">+</span> <span class="nx">d4</span><span class="p">;</span> <span class="c1">// the score</span>
</code></pre></div></div>

<p>The specification above is a 2-layer Neural Network with 3 hidden neurons (n1, n2, n3) that uses Rectified Linear Unit (ReLU) non-linearity on each hidden neuron. As you can see, there are now several parameters involved, which means that our classifier is more complex and can represent more intricate decision boundaries than just a simple linear decision rule such as an SVM. Another way to think about it is that every one of the three hidden neurons is a linear classifier and now we’re putting an extra linear classifier on top of that. Now we’re starting to go <em>deeper</em> :). Okay, lets train this 2-layer Neural Network. The code looks very similar to the SVM example code above, we just have to change the forward pass and the backward pass:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// random initial parameters</span>
<span class="kd">var</span> <span class="nx">a1</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()</span> <span class="o">-</span> <span class="mf">0.5</span><span class="p">;</span> <span class="c1">// a random number between -0.5 and 0.5</span>
<span class="c1">// ... similarly initialize all other parameters to randoms</span>
<span class="k">for</span><span class="p">(</span><span class="kd">var</span> <span class="nx">iter</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">iter</span> <span class="o">&lt;</span> <span class="mi">400</span><span class="p">;</span> <span class="nx">iter</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// pick a random data point</span>
  <span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">floor</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()</span> <span class="o">*</span> <span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
  <span class="kd">var</span> <span class="nx">x</span> <span class="o">=</span> <span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">0</span><span class="p">];</span>
  <span class="kd">var</span> <span class="nx">y</span> <span class="o">=</span> <span class="nx">data</span><span class="p">[</span><span class="nx">i</span><span class="p">][</span><span class="mi">1</span><span class="p">];</span>
  <span class="kd">var</span> <span class="nx">label</span> <span class="o">=</span> <span class="nx">labels</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>

  <span class="c1">// compute forward pass</span>
  <span class="kd">var</span> <span class="nx">n1</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">a1</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b1</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c1</span><span class="p">);</span> <span class="c1">// activation of 1st hidden neuron</span>
  <span class="kd">var</span> <span class="nx">n2</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">a2</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b2</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c2</span><span class="p">);</span> <span class="c1">// 2nd neuron</span>
  <span class="kd">var</span> <span class="nx">n3</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">a3</span><span class="o">*</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b3</span><span class="o">*</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">c3</span><span class="p">);</span> <span class="c1">// 3rd neuron</span>
  <span class="kd">var</span> <span class="nx">score</span> <span class="o">=</span> <span class="nx">a4</span><span class="o">*</span><span class="nx">n1</span> <span class="o">+</span> <span class="nx">b4</span><span class="o">*</span><span class="nx">n2</span> <span class="o">+</span> <span class="nx">c4</span><span class="o">*</span><span class="nx">n3</span> <span class="o">+</span> <span class="nx">d4</span><span class="p">;</span> <span class="c1">// the score</span>

  <span class="c1">// compute the pull on top</span>
  <span class="kd">var</span> <span class="nx">pull</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>
  <span class="k">if</span><span class="p">(</span><span class="nx">label</span> <span class="o">===</span> <span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="nx">score</span> <span class="o">&lt;</span> <span class="mi">1</span><span class="p">)</span> <span class="nx">pull</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span> <span class="c1">// we want higher output! Pull up.</span>
  <span class="k">if</span><span class="p">(</span><span class="nx">label</span> <span class="o">===</span> <span class="o">-</span><span class="mi">1</span> <span class="o">&amp;&amp;</span> <span class="nx">score</span> <span class="o">&gt;</span> <span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="nx">pull</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="p">;</span> <span class="c1">// we want lower output! Pull down.</span>

  <span class="c1">// now compute backward pass to all parameters of the model</span>

  <span class="c1">// backprop through the last "score" neuron</span>
  <span class="kd">var</span> <span class="nx">dscore</span> <span class="o">=</span> <span class="nx">pull</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">da4</span> <span class="o">=</span> <span class="nx">n1</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dn1</span> <span class="o">=</span> <span class="nx">a4</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">db4</span> <span class="o">=</span> <span class="nx">n2</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dn2</span> <span class="o">=</span> <span class="nx">b4</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dc4</span> <span class="o">=</span> <span class="nx">n3</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dn3</span> <span class="o">=</span> <span class="nx">c4</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dd4</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dscore</span><span class="p">;</span> <span class="c1">// phew</span>

  <span class="c1">// backprop the ReLU non-linearities, in place</span>
  <span class="c1">// i.e. just set gradients to zero if the neurons did not "fire"</span>
  <span class="kd">var</span> <span class="nx">dn3</span> <span class="o">=</span> <span class="nx">n3</span> <span class="o">===</span> <span class="mi">0</span> <span class="p">?</span> <span class="mi">0</span> <span class="p">:</span> <span class="nx">dn3</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dn2</span> <span class="o">=</span> <span class="nx">n2</span> <span class="o">===</span> <span class="mi">0</span> <span class="p">?</span> <span class="mi">0</span> <span class="p">:</span> <span class="nx">dn2</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dn1</span> <span class="o">=</span> <span class="nx">n1</span> <span class="o">===</span> <span class="mi">0</span> <span class="p">?</span> <span class="mi">0</span> <span class="p">:</span> <span class="nx">dn1</span><span class="p">;</span>

  <span class="c1">// backprop to parameters of neuron 1</span>
  <span class="kd">var</span> <span class="nx">da1</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">*</span> <span class="nx">dn1</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">db1</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">*</span> <span class="nx">dn1</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dc1</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dn1</span><span class="p">;</span>
  
  <span class="c1">// backprop to parameters of neuron 2</span>
  <span class="kd">var</span> <span class="nx">da2</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">*</span> <span class="nx">dn2</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">db2</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">*</span> <span class="nx">dn2</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dc2</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dn2</span><span class="p">;</span>

  <span class="c1">// backprop to parameters of neuron 3</span>
  <span class="kd">var</span> <span class="nx">da3</span> <span class="o">=</span> <span class="nx">x</span> <span class="o">*</span> <span class="nx">dn3</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">db3</span> <span class="o">=</span> <span class="nx">y</span> <span class="o">*</span> <span class="nx">dn3</span><span class="p">;</span>
  <span class="kd">var</span> <span class="nx">dc3</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">*</span> <span class="nx">dn3</span><span class="p">;</span>

  <span class="c1">// phew! End of backprop!</span>
  <span class="c1">// note we could have also backpropped into x,y</span>
  <span class="c1">// but we do not need these gradients. We only use the gradients</span>
  <span class="c1">// on our parameters in the parameter update, and we discard x,y</span>

  <span class="c1">// add the pulls from the regularization, tugging all multiplicative</span>
  <span class="c1">// parameters (i.e. not the biases) downward, proportional to their value</span>
  <span class="nx">da1</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">a1</span><span class="p">;</span> <span class="nx">da2</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">a2</span><span class="p">;</span> <span class="nx">da3</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">a3</span><span class="p">;</span>
  <span class="nx">db1</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">b1</span><span class="p">;</span> <span class="nx">db2</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">b2</span><span class="p">;</span> <span class="nx">db3</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">b3</span><span class="p">;</span>
  <span class="nx">da4</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">a4</span><span class="p">;</span> <span class="nx">db4</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">b4</span><span class="p">;</span> <span class="nx">dc4</span> <span class="o">+=</span> <span class="o">-</span><span class="nx">c4</span><span class="p">;</span>

  <span class="c1">// finally, do the parameter update</span>
  <span class="kd">var</span> <span class="nx">step_size</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
  <span class="nx">a1</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">da1</span><span class="p">;</span> 
  <span class="nx">b1</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">db1</span><span class="p">;</span> 
  <span class="nx">c1</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">dc1</span><span class="p">;</span>
  <span class="nx">a2</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">da2</span><span class="p">;</span> 
  <span class="nx">b2</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">db2</span><span class="p">;</span>
  <span class="nx">c2</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">dc2</span><span class="p">;</span>
  <span class="nx">a3</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">da3</span><span class="p">;</span> 
  <span class="nx">b3</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">db3</span><span class="p">;</span> 
  <span class="nx">c3</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">dc3</span><span class="p">;</span>
  <span class="nx">a4</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">da4</span><span class="p">;</span> 
  <span class="nx">b4</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">db4</span><span class="p">;</span> 
  <span class="nx">c4</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">dc4</span><span class="p">;</span> 
  <span class="nx">d4</span> <span class="o">+=</span> <span class="nx">step_size</span> <span class="o">*</span> <span class="nx">dd4</span><span class="p">;</span>
  <span class="c1">// wow this is tedious, please use for loops in prod.</span>
  <span class="c1">// we're done!</span>
<span class="p">}</span>
</code></pre></div></div>

<p>And that’s how you train a neural network. Obviously, you want to modularize your code nicely but I expended this example for you in the hope that it makes things much more concrete and simpler to understand. Later, we will look at best practices when implementing these networks and we will structure the code much more neatly in a modular and more sensible way.</p>

<p>But for now, I hope your takeaway is that a 2-layer Neural Net is really not such a scary thing: we write a forward pass expression, interpret the value at the end as a score, and then we pull on that value in a positive or negative direction depending on what we want that value to be for our current particular example. The parameter update after backprop will ensure that when we see this particular example in the future, the network will be more likely to give us a value we desire, not the one it gave just before the update.</p>

<h3 id="a-more-conventional-approach-loss-functions">A more Conventional Approach: Loss Functions</h3>

<p>Now that we understand the basics of how these circuits function with data, lets adopt a more conventional approach that you might see elsewhere on the internet and in other tutorials and books. You won’t see people talking too much about <strong>force specifications</strong>. Instead, Machine Learning algorithms are specified in terms of <strong>loss functions</strong> (or <strong>cost functions</strong>, or <strong>objectives</strong>).</p>

<p>As I develop this formalism I would also like to start to be a little more careful with how we name our variables and parameters. I’d like these equations to look similar to what you might see in a book or some other tutorial, so let me use more standard naming conventions.</p>

<h4 id="example-2-d-support-vector-machine">Example: 2-D Support Vector Machine</h4>
<p>Lets start with an example of a 2-dimensional SVM. We are given a dataset of \( N \) examples \( (x_{i0}, x_{i1}) \) and their corresponding labels \( y_{i} \) which are allowed to be either \( +1/-1 \) for positive or negative example respectively. Most importantly, as you recall we have three parameters \( (w_0, w_1, w_2) \). The SVM loss function is then defined as follows:</p>

\[L = [\sum\_{i=1}^N max(0, -y\_{i}( w\_0x\_{i0} + w\_1x\_{i1} + w\_2 ) + 1 )] + \alpha [w\_0^2 + w\_1^2]\]

<p>Notice that this expression is always positive, due to the thresholding at zero in the first expression and the squaring in the regularization. The idea is that we will want this expression to be as small as possible. Before we dive into some of its subtleties let me first translate it to code:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">var</span> <span class="nx">X</span> <span class="o">=</span> <span class="p">[</span> <span class="p">[</span><span class="mf">1.2</span><span class="p">,</span> <span class="mf">0.7</span><span class="p">],</span> <span class="p">[</span><span class="o">-</span><span class="mf">0.3</span><span class="p">,</span> <span class="mf">0.5</span><span class="p">],</span> <span class="p">[</span><span class="mi">3</span><span class="p">,</span> <span class="mf">2.5</span><span class="p">]</span> <span class="p">]</span> <span class="c1">// array of 2-dimensional data</span>
<span class="kd">var</span> <span class="nx">y</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">]</span> <span class="c1">// array of labels</span>
<span class="kd">var</span> <span class="nx">w</span> <span class="o">=</span> <span class="p">[</span><span class="mf">0.1</span><span class="p">,</span> <span class="mf">0.2</span><span class="p">,</span> <span class="mf">0.3</span><span class="p">]</span> <span class="c1">// example: random numbers</span>
<span class="kd">var</span> <span class="nx">alpha</span> <span class="o">=</span> <span class="mf">0.1</span><span class="p">;</span> <span class="c1">// regularization strength</span>

<span class="kd">function</span> <span class="nx">cost</span><span class="p">(</span><span class="nx">X</span><span class="p">,</span> <span class="nx">y</span><span class="p">,</span> <span class="nx">w</span><span class="p">)</span> <span class="p">{</span>
  
  <span class="kd">var</span> <span class="nx">total_cost</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span> <span class="c1">// L, in SVM loss function above</span>
  <span class="nx">N</span> <span class="o">=</span> <span class="nx">X</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>
  <span class="k">for</span><span class="p">(</span><span class="kd">var</span> <span class="nx">i</span><span class="o">=</span><span class="mi">0</span><span class="p">;</span><span class="nx">i</span><span class="o">&lt;</span><span class="nx">N</span><span class="p">;</span><span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// loop over all data points and compute their score</span>
    <span class="kd">var</span> <span class="nx">xi</span> <span class="o">=</span> <span class="nx">X</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>
    <span class="kd">var</span> <span class="nx">score</span> <span class="o">=</span> <span class="nx">w</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">*</span> <span class="nx">xi</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">+</span> <span class="nx">w</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span> <span class="o">*</span> <span class="nx">xi</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span> <span class="o">+</span> <span class="nx">w</span><span class="p">[</span><span class="mi">2</span><span class="p">];</span>
    
    <span class="c1">// accumulate cost based on how compatible the score is with the label</span>
    <span class="kd">var</span> <span class="nx">yi</span> <span class="o">=</span> <span class="nx">y</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span> <span class="c1">// label</span>
    <span class="kd">var</span> <span class="nx">costi</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">max</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="o">-</span> <span class="nx">yi</span> <span class="o">*</span> <span class="nx">score</span> <span class="o">+</span> <span class="mi">1</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">example </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">i</span> <span class="o">+</span> <span class="dl">'</span><span class="s1">: xi = (</span><span class="dl">'</span> <span class="o">+</span> <span class="nx">xi</span> <span class="o">+</span> <span class="dl">'</span><span class="s1">) and label = </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">yi</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">  score computed to be </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">score</span><span class="p">.</span><span class="nx">toFixed</span><span class="p">(</span><span class="mi">3</span><span class="p">));</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">  =&gt; cost computed to be </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">costi</span><span class="p">.</span><span class="nx">toFixed</span><span class="p">(</span><span class="mi">3</span><span class="p">));</span>
    <span class="nx">total_cost</span> <span class="o">+=</span> <span class="nx">costi</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="c1">// regularization cost: we want small weights</span>
  <span class="nx">reg_cost</span> <span class="o">=</span> <span class="nx">alpha</span> <span class="o">*</span> <span class="p">(</span><span class="nx">w</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="o">*</span><span class="nx">w</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">+</span> <span class="nx">w</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="o">*</span><span class="nx">w</span><span class="p">[</span><span class="mi">1</span><span class="p">])</span>
  <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">regularization cost for current model is </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">reg_cost</span><span class="p">.</span><span class="nx">toFixed</span><span class="p">(</span><span class="mi">3</span><span class="p">));</span>
  <span class="nx">total_cost</span> <span class="o">+=</span> <span class="nx">reg_cost</span><span class="p">;</span>

  <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">total cost is </span><span class="dl">'</span> <span class="o">+</span> <span class="nx">total_cost</span><span class="p">.</span><span class="nx">toFixed</span><span class="p">(</span><span class="mi">3</span><span class="p">));</span>
  <span class="k">return</span> <span class="nx">total_cost</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>

<p>And here is the output:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>cost for example 0 is 0.440
cost for example 1 is 1.370
cost for example 2 is 0.000
regularization cost for current model is 0.005
total cost is 1.815 
</code></pre></div></div>

<p>Notice how this expression works: It measures how <em>bad</em> our SVM classifier is. Lets step through this explicitly:</p>

<ul>
  <li>The first datapoint <code class="language-plaintext highlighter-rouge">xi = [1.2, 0.7]</code> with label <code class="language-plaintext highlighter-rouge">yi = 1</code> will give score <code class="language-plaintext highlighter-rouge">0.1*1.2 + 0.2*0.7 + 0.3</code>, which is <code class="language-plaintext highlighter-rouge">0.56</code>. Notice, this is a positive example so we want to the score to be greater than <code class="language-plaintext highlighter-rouge">+1</code>. <code class="language-plaintext highlighter-rouge">0.56</code> is not enough. And indeed, the expression for cost for this datapoint will compute: <code class="language-plaintext highlighter-rouge">costi = Math.max(0, -1*0.56 + 1)</code>, which is <code class="language-plaintext highlighter-rouge">0.44</code>. You can think of the cost as quantifying the SVM’s unhappiness.</li>
  <li>The second datapoint <code class="language-plaintext highlighter-rouge">xi = [-0.3, 0.5]</code> with label <code class="language-plaintext highlighter-rouge">yi = -1</code> will give score <code class="language-plaintext highlighter-rouge">0.1*(-0.3) + 0.2*0.5 + 0.3</code>, which is <code class="language-plaintext highlighter-rouge">0.37</code>. This isn’t looking very good: This score is very high for a negative example. It should be less than -1. Indeed, when we compute the cost: <code class="language-plaintext highlighter-rouge">costi = Math.max(0, 1*0.37 + 1)</code>, we get <code class="language-plaintext highlighter-rouge">1.37</code>. That’s a very high cost from this example, as it is being misclassified.</li>
  <li>The last example <code class="language-plaintext highlighter-rouge">xi = [3, 2.5]</code> with label <code class="language-plaintext highlighter-rouge">yi = 1</code> gives score <code class="language-plaintext highlighter-rouge">0.1*3 + 0.2*2.5 + 0.3</code>, and that is <code class="language-plaintext highlighter-rouge">1.1</code>. In this case, the SVM will compute <code class="language-plaintext highlighter-rouge">costi = Math.max(0, -1*1.1 + 1)</code>, which is in fact zero. This datapoint is being classified correctly and there is no cost associated with it.</li>
</ul>

<blockquote>
  <p>A cost function is an expression that measuress how bad your classifier is. When the training set is perfectly classified, the cost (ignoring the regularization) will be zero.</p>
</blockquote>

<p>Notice that the last term in the loss is the regularization cost, which says that our model parameters should be small values. Due to this term the cost will never actually become zero (because this would mean all parameters of the model except the bias are exactly zero), but the closer we get, the better our classifier will become.</p>

<blockquote>
  <p>The majority of cost functions in Machine Learning consist of two parts: 1. A part that measures how well a model fits the data, and 2: Regularization, which measures some notion of how complex or likely a model is.</p>
</blockquote>

<p>I hope I convinced you then, that to get a very good SVM we really want to make the <strong>cost as small as possible</strong>. Sounds familiar? We know exactly what to do: The cost function written above is our circuit. We will forward all examples through the circuit, compute the backward pass and update all parameters such that the circuit will output a <em>smaller</em> cost in the future. Specifically, we will compute the <em>gradient</em> and then update the parameters in the <em>opposite direction</em> of the gradient (since we want to make the cost small, not large).</p>

<blockquote>
  <p>“We know exactly what to do: The cost function written above is our circuit.”</p>
</blockquote>

<p>todo: clean up this section and flesh it out a bit…</p>

<h2 id="chapter-3-backprop-in-practice">Chapter 3: Backprop in Practice</h2>

<h3 id="building-up-a-library">Building up a library</h3>

<h3 id="example-practical-neural-network-classifier">Example: Practical Neural Network Classifier</h3>

<ul>
  <li>Multiclass: Structured SVM</li>
  <li>Multiclass: Logistic Regression, Softmax</li>
</ul>

<h3 id="example-regression">Example: Regression</h3>

<p>Tiny changes needed to cost function. L2 regularization.</p>

<h3 id="example-structured-prediction">Example: Structured Prediction</h3>

<p>Basic idea is to train an (unnormalized) energy model</p>

<h3 id="vectorized-implementations">Vectorized Implementations</h3>

<p>Writing a Neural Net classfier in Python with numpy….</p>

<h3 id="backprop-in-practice-tipstricks">Backprop in practice: Tips/Tricks</h3>

<ul>
  <li>Monitoring of Cost function</li>
  <li>Monitoring training/validation performance</li>
  <li>Tweaking initial learning rates, learning rate schedules</li>
  <li>Optimization: Using Momentum</li>
  <li>Optimization: LBFGS, Nesterov accelerated gradient</li>
  <li>Importance of Initialization: weights and biases</li>
  <li>Regularization: L2, L1, Group sparsity, Dropout</li>
  <li>Hyperparameter search, cross-validations</li>
  <li>Common pitfalls: (e.g. dying ReLUs)</li>
  <li>Handling unbalanced datasets</li>
  <li>Approaches to debugging nets when something doesnt work</li>
</ul>

<h2 id="chapter-4-networks-in-the-wild">Chapter 4: Networks in the Wild</h2>

<p>Case studies of models that work well in practice and have been deployed in the wild.</p>

<h3 id="case-study-convolutional-neural-networks-for-images">Case Study: Convolutional Neural Networks for images</h3>

<p>Convolutional layers, pooling, AlexNet, etc.</p>

<h3 id="case-study-recurrent-neural-networks-for-speech-and-text">Case Study: Recurrent Neural Networks for Speech and Text</h3>

<p>Vanilla Recurrent nets, bi-directional recurrent nets. Maybe overview of LSTM</p>

<h3 id="case-study-word2vec">Case Study: Word2Vec</h3>

<p>Training word vector representations in NLP</p>

<h3 id="case-study-t-sne">Case Study: t-SNE</h3>

<p>Training embeddings for visualizing data</p>

<h2 id="acknowledgements">Acknowledgements</h2>

<p>Thanks a lot to the following people who made this guide better: wodenokoto (HN), zackmorris (HN).</p>

<h2 id="comments">Comments</h2>

<p>This guide is a work in progress and I appreciate feedback, especially regarding parts that were unclear or only made half sense. Thank you!</p>

<p>Some of the Javascript code in this tutorial has been translated to Python by Ajit, find it over on <a href="https://github.com/urwithajit9/HG_NeuralNetwork">Github</a>.</p>

  </article>

  <!-- disqus comments -->
 
 <div id="disqus_thread"></div>
  <script>
  /**
  *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
  *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
  var disqus_config = function () {
    this.page.url = "http://karpathy.github.io/neuralnets/";  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "/neuralnets/"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
  };
  (function() { // DON'T EDIT BELOW THIS LINE
  var d = document, s = d.createElement('script');
  s.src = 'https://karpathyblog.disqus.com/embed.js';
  s.setAttribute('data-timestamp', +new Date());
  (d.head || d.body).appendChild(s);
  })();
  </script>
  <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
  
  
</div>

<!-- mathjax -->

<script type="text/javascript" src="//cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>


      </div>
    </div>

    <footer class="site-footer">

  <div class="wrap">

    <!-- <h2 class="footer-heading">Andrej Karpathy blog</h2> -->

    <div class="footer-col-1 column">
      <ul>
        <li>Andrej Karpathy blog</li>
        <!-- <li><a href="mailto:"></a></li> -->
      </ul>
    </div>

    <div class="footer-col-2 column">
      <ul>
        <li>
          <a href="https://github.com/karpathy">
            <span class="icon github">
              <svg version="1.1" class="github-icon-svg" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                 viewBox="0 0 16 16" enable-background="new 0 0 16 16" xml:space="preserve">
                <path fill-rule="evenodd" clip-rule="evenodd" fill="#C2C2C2" d="M7.999,0.431c-4.285,0-7.76,3.474-7.76,7.761
                c0,3.428,2.223,6.337,5.307,7.363c0.388,0.071,0.53-0.168,0.53-0.374c0-0.184-0.007-0.672-0.01-1.32
                c-2.159,0.469-2.614-1.04-2.614-1.04c-0.353-0.896-0.862-1.135-0.862-1.135c-0.705-0.481,0.053-0.472,0.053-0.472
                c0.779,0.055,1.189,0.8,1.189,0.8c0.692,1.186,1.816,0.843,2.258,0.645c0.071-0.502,0.271-0.843,0.493-1.037
                C4.86,11.425,3.049,10.76,3.049,7.786c0-0.847,0.302-1.54,0.799-2.082C3.768,5.507,3.501,4.718,3.924,3.65
                c0,0,0.652-0.209,2.134,0.796C6.677,4.273,7.34,4.187,8,4.184c0.659,0.003,1.323,0.089,1.943,0.261
                c1.482-1.004,2.132-0.796,2.132-0.796c0.423,1.068,0.157,1.857,0.077,2.054c0.497,0.542,0.798,1.235,0.798,2.082
                c0,2.981-1.814,3.637-3.543,3.829c0.279,0.24,0.527,0.713,0.527,1.437c0,1.037-0.01,1.874-0.01,2.129
                c0,0.208,0.14,0.449,0.534,0.373c3.081-1.028,5.302-3.935,5.302-7.362C15.76,3.906,12.285,0.431,7.999,0.431z"/>
              </svg>
            </span>
            <span class="username">karpathy</span>
          </a>
        </li>
        <li>
          <a href="https://twitter.com/karpathy">
            <span class="icon twitter">
              <svg version="1.1" class="twitter-icon-svg" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                 viewBox="0 0 16 16" enable-background="new 0 0 16 16" xml:space="preserve">
                <path fill="#C2C2C2" d="M15.969,3.058c-0.586,0.26-1.217,0.436-1.878,0.515c0.675-0.405,1.194-1.045,1.438-1.809
                c-0.632,0.375-1.332,0.647-2.076,0.793c-0.596-0.636-1.446-1.033-2.387-1.033c-1.806,0-3.27,1.464-3.27,3.27
                c0,0.256,0.029,0.506,0.085,0.745C5.163,5.404,2.753,4.102,1.14,2.124C0.859,2.607,0.698,3.168,0.698,3.767
                c0,1.134,0.577,2.135,1.455,2.722C1.616,6.472,1.112,6.325,0.671,6.08c0,0.014,0,0.027,0,0.041c0,1.584,1.127,2.906,2.623,3.206
                C3.02,9.402,2.731,9.442,2.433,9.442c-0.211,0-0.416-0.021-0.615-0.059c0.416,1.299,1.624,2.245,3.055,2.271
                c-1.119,0.877-2.529,1.4-4.061,1.4c-0.264,0-0.524-0.015-0.78-0.046c1.447,0.928,3.166,1.469,5.013,1.469
                c6.015,0,9.304-4.983,9.304-9.304c0-0.142-0.003-0.283-0.009-0.423C14.976,4.29,15.531,3.714,15.969,3.058z"/>
              </svg>
            </span>
            <span class="username">karpathy</span>
          </a>
        </li>
      </ul>
    </div>

    <div class="footer-col-3 column">
      <p class="text">Musings of a Computer Scientist.</p>
    </div>

  </div>

</footer>


    </body>
</html>
