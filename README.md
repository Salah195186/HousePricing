<!DOCTYPE html>
<html>

  <head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>CS231n Convolutional Neural Networks for Visual Recognition</title>
  <meta name="viewport" content="width=device-width">
  <meta name="description" content="Course materials and notes for Stanford class CS231n: Convolutional Neural Networks for Visual Recognition.">
  <link rel="canonical" href="https://cs231n.github.io/">

  <!-- Custom CSS -->
  <link rel="stylesheet" href="/css/main.css">

  <!-- Google fonts -->
  <link href='https://fonts.googleapis.com/css?family=Roboto:400,300' rel='stylesheet' type='text/css'>

  <!-- Google tracking -->
  <script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-46895817-2', 'auto');
    ga('send', 'pageview');
  </script>
</head>


    <body>

      <script src="https://unpkg.com/vanilla-back-to-top@7.2.1/dist/vanilla-back-to-top.min.js"></script>
      <script>addBackToTop({
        backgroundColor: '#fff',
        innerHTML: 'Back to Top',
        textColor: '#333'
      })</script>
      <style>
        #back-to-top {
          border: 1px solid #ccc;
          border-radius: 0;
          font-family: sans-serif;
          font-size: 14px;
          width: 100px;
          text-align: center;
          line-height: 30px;
          height: 30px;
        }
      </style>

    <header class="site-header">

  <a class="site-title" href="https://cs231n.github.io">CS231n Convolutional Neural Networks for Visual Recognition</a>
  <a class="site-link" href="http://cs231n.stanford.edu/">Course Website</a>

</header>


    <div class="page-content">
      <div class="wrap">
      <div>

  These notes accompany the Stanford CS class <a href="http://cs231n.stanford.edu/">CS231n: Convolutional Neural Networks for Visual Recognition</a>. For questions/concerns/bug reports, please submit a pull request directly to our <a href="https://github.com/cs231n/cs231n.github.io">git repo</a>.
  <br>
  <!-- For questions/concerns/bug reports contact <a href="http://cs.stanford.edu/people/jcjohns/">Justin Johnson</a> regarding the assignments, or contact <a href="http://cs.stanford.edu/people/karpathy/">Andrej Karpathy</a> regarding the course notes. You can also submit a pull request directly to our <a href="https://github.com/cs231n/cs231n.github.io">git repo</a>. -->
  <!-- <br> -->
  <!-- We encourage the use of the <a href="https://hypothes.is/">hypothes.is</a> extension to annote comments and discuss these notes inline. -->
</div>

<div class="home">
  <div class="materials-wrap">
    <div class="module-header">Spring 2020 Assignments</div>
    <div class="materials-item">
      <a href="assignments2020/assignment1/">
        Assignment #1: Image Classification, kNN, SVM, Softmax, Fully-Connected Neural Network
      </a>
    </div>
    <div class="materials-item">
      <a href="assignments2020/assignment2/">
        Assignment #2: Fully-Connected Nets, BatchNorm, Dropout, ConvNets, Tensorflow/Pytorch
      </a>
    </div>
    <div class="materials-item">
      <a href="assignments2020/assignment3/">
        Assignment #3: Image Captioning with Vanilla RNNs and LSTMs, Neural Net Visualization, Style Transfer, Generative Adversarial Networks
      </a>
    </div>
<!--
    <div class="materials-item">
      <a href="assignments2019/assignment2/">
        Assignment #2: Fully-Connected Nets, Batch Normalization, Dropout,
        Convolutional Nets
      </a>
    </div>

    <div class="materials-item">
      <a href="assignments2019/assignment3/">
        Assignment #3: Image Captioning with Vanilla RNNs, Image Captioning
  with LSTMs, Network Visualization, Style Transfer, Generative Adversarial Networks
      </a>
    </div> -->
    <!--
    <div class="module-header">Spring 2018 Assignments</div>

    <div class="materials-item">
      <a href="assignments2018/assignment1/">
        Assignment #1: Image Classification, kNN, SVM, Softmax, Neural Network
      </a>
    </div>

    <div class="materials-item">
      <a href="assignments2018/assignment2/">
        Assignment #2: Fully-Connected Nets, Batch Normalization, Dropout,
        Convolutional Nets
      </a>
    </div>

    <div class="materials-item">
      <a href="assignments2018/assignment3/">
        Assignment #3: Image Captioning with Vanilla RNNs, Image Captioning
        with LSTMs, Network Visualization, Style Transfer, Generative Adversarial Networks
      </a>
    </div>
    -->

    <!--
    <div class="module-header">Winter 2016 Assignments</div>

    <div class="materials-item">
      <a href="assignments2016/assignment1/">
        Assignment #1: Image Classification, kNN, SVM, Softmax, Neural Network
      </a>
    </div>

    <div class="materials-item">
      <a href="assignments2016/assignment2/">
        Assignment #2: Fully-Connected Nets, Batch Normalization, Dropout,
        Convolutional Nets
      </a>
    </div>

    <div class="materials-item">
      <a href="assignments2016/assignment3/">
        Assignment #3: Recurrent Neural Networks, Image Captioning,
        Image Gradients, DeepDream
      </a>
    </div>
    -->

    <!--
    <div class="module-header">Winter 2015 Assignments</div>

    <div class="materials-item">
      <a href="assignment1/">
        Assignment #1: Image Classification, kNN, SVM, Softmax
      </a>
    </div>

    <div class="materials-item">
      <a href="assignment2/">
        Assignment #2: Neural Networks, ConvNets I
      </a>
    </div>

    <div class="materials-item">
      <a href="assignment3/">
        Assignment #3: ConvNets II, Transfer Learning, Visualization
      </a>
    </div>
  -->

    <div class="module-header">Module 0: Preparation</div>

    <div class="materials-item">
      <a href="setup-instructions/">
        Software Setup
      </a>
    </div>

    <div class="materials-item">
      <a href="python-numpy-tutorial/">
        Python / Numpy Tutorial (with Jupyter and Colab)
      </a>
    </div>
<!--
    <div class="materials-item">
      <a href="terminal-tutorial/">
        Terminal.com Tutorial
      </a>
    </div>
-->
    <div class="materials-item">
      <a href="https://github.com/cs231n/gcloud">
        Google Cloud Tutorial
      </a>
    </div>
    <!-- <div class="materials-item">
      <a href="aws-tutorial/">
        AWS Tutorial
      </a>
    </div> -->

    <!-- hardcoding items here to force a specific order -->
    <div class="module-header">Module 1: Neural Networks</div>

    <div class="materials-item">
      <a href="classification/">
        Image Classification: Data-driven Approach, k-Nearest Neighbor, train/val/test splits
      </a>
      <div class="kw">
        L1/L2 distances, hyperparameter search, cross-validation
      </div>
    </div>

    <div class="materials-item">
      <a href="linear-classify/">
         Linear classification: Support Vector Machine, Softmax
      </a>
      <div class="kw">
        parameteric approach, bias trick, hinge loss, cross-entropy loss, L2 regularization, web demo
      </div>
    </div>

    <div class="materials-item">
      <a href="optimization-1/">
        Optimization: Stochastic Gradient Descent
      </a>
      <div class="kw">
        optimization landscapes, local search, learning rate, analytic/numerical gradient
      </div>
    </div>

    <div class="materials-item">
      <a href="optimization-2/">
        Backpropagation, Intuitions
      </a>
      <div class="kw">
        chain rule interpretation, real-valued circuits, patterns in gradient flow
      </div>
    </div>

    <div class="materials-item">
      <a href="neural-networks-1/">
          Neural Networks Part 1: Setting up the Architecture
      </a>
      <div class="kw">
        model of a biological neuron, activation functions, neural net architecture, representational power
      </div>
    </div>

    <div class="materials-item">
      <a href="neural-networks-2/">
          Neural Networks Part 2: Setting up the Data and the Loss
      </a>
      <div class="kw">
          preprocessing, weight initialization, batch normalization, regularization (L2/dropout), loss functions
      </div>
    </div>

    <div class="materials-item">
      <a href="neural-networks-3/">
        Neural Networks Part 3: Learning and Evaluation
      </a>
      <div class="kw">
        gradient checks, sanity checks, babysitting the learning process, momentum (+nesterov), second-order methods, Adagrad/RMSprop, hyperparameter optimization, model ensembles
      </div>
    </div>

    <div class="materials-item">
      <a href="neural-networks-case-study/">
          Putting it together: Minimal Neural Network Case Study
      </a>
      <div class="kw">
        minimal 2D toy data example
      </div>
    </div>

    <div class="module-header">Module 2: Convolutional Neural Networks</div>

    <div class="materials-item">
      <a href="convolutional-networks/">
        Convolutional Neural Networks: Architectures, Convolution / Pooling Layers
      </a>
      <div class="kw">
          layers, spatial arrangement, layer patterns, layer sizing patterns, AlexNet/ZFNet/VGGNet case studies, computational considerations
      </div>
    </div>

    <div class="materials-item">
      <a href="understanding-cnn/">
        Understanding and Visualizing Convolutional Neural Networks
      </a>
      <div class="kw">
        tSNE embeddings, deconvnets, data gradients, fooling ConvNets, human comparisons
      </div>
    </div>

    <div class="materials-item">
      <a href="transfer-learning/">
        Transfer Learning and Fine-tuning Convolutional Neural Networks
      </a>
    </div>

    <div class="module-header">Student-Contributed Posts</div>

    <div class="materials-item">
      <a href="choose-project/">
        Taking a Course Project to Publication
      </a>
    </div>

  </div>
</div>

      </div>
    </div>

    <footer class="site-footer">

  <div class="wrap">

    <div class="footer-col-1 column">
      <ul>

        <li>
          <a href="https://github.com/cs231n">
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
            <span class="username">cs231n</span>
          </a>
        </li>
        <li>
          <a href="https://twitter.com/cs231n">
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
            <span class="username">cs231n</span>
          </a>
        </li>
        <!-- <li>
          <a href="mailto:karpathy@cs.stanford.edu">karpathy@cs.stanford.edu</a>
        </li> -->
      </ul>
    </div>

    <div class="footer-col-2 column">

    </div>

    <div class="footer-col-3 column">

    </div>

  </div>

</footer>


    <!-- mathjax -->
    <script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
    <script type="text/x-mathjax-config">
      // Make responsive
      MathJax.Hub.Config({
       "HTML-CSS": { linebreaks: { automatic: true } },
       "SVG": { linebreaks: { automatic: true } },
      });
    </script>

    </body>
</html>
