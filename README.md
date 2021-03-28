# Prediction of House Price Based on The Back Propagation Neural Network in The Keras Deep Learning Framework

#  Introduction

House prices have been increasing in recent years, in tandem with the global economy's rapid growth. The real estate sector has gradually grown to become a significant pillar of the global economy. In this environment, nearly everybody is paying attention to the home market trend and attempting to use more empirical and effective approaches to make correct housing price forecasts. Aside from the effect of the house's characteristics, other factors, especially the characteristics of the parties to the sale, have an impact on the house's price. The house price dilemma can be seen as a large open complex system with a lot of complexity, volatility, nonlinearity, and dynamics.



This study uses the chain home network's housing data to forecast the price of used housing in Shanghai. To begin, this paper uses crawler technologies to decode the URL text information using the BeautifulSoup parser and the json request address. Then, using the Keras deep learning library, a multi-layer feedforward neural network model is equipped using the error inverse propagation algorithm. 

The data used in this paper was crawled from the Web Host using crawler technology, and experiments were performed using a back propagation neural network (BP neural network) model based on the Keras paradigm. The BP neural network is a multilayer feedforward network that has been trained using the error inverse propagation algorithm. It's one of the most common neural network models out there. It can learn and store a huge number of input-output mode mappings without disclosing the mapping relationship's mathematical equations. Its learning rule is to use the gradient steepest descent approach to continuously change the network's weight and threshold through error back propagation until the network's square error reaches a minimum. For housing price forecasting, 12 variables influencing real estate are chosen; the selected indicators have different aspects such as geographic coordinates, traffic orientation, and housing type. The abnormal value was processed using random forest during the experiment, and a better prediction outcome was obtained as a result.


### Keras 

Keras is a Python-based neural network library with a high degree of modularity. The backend may be either TensorFlow or Theano [1]. Keras also includes modules such as activation function modules, sheet modules, preprocessing modules, objective function modules, optimization system selection modules, and so on. Activation function modules and optimization process selection modules [2] are two of them, and they combine all of the most recent and best optimization approaches and activation functions. Network models can be easily developed with these components, and core parameters of neural networks can be enhanced.

### Back Propagation NN

The BP NN [3] is a multi-layer feedforward NN trained by using the error back propagation algorithm suggested by the Mccelland  and Rumelhart in 1986. It typically consists of three layers: input, hidden, and output. Figure 1 depicts the network structure. 
![image](https://user-images.githubusercontent.com/81248615/112744876-44c0a980-8fbd-11eb-9a1c-2d1ecd4c06ce.png)


# Results 

The analysis with the absolute value of the relative error between the expected value and the real value is within 5%, accounting for 95.59 percent, according to the measurement and statistical result. The neural network built by other documents of the same type was applied to the data of this experiment for comparison in order to verify the validity of the experiment. The results of the studies indicate that the NN developed in this analysis are marginally better than previous neural networks and have good model performance.


## Paper Analyzed for this blog
   Z. Jiang and G. Shen, "Prediction of House Price Based on The Back Propagation Neural Network in The Keras Deep Learning Framework," 2019 6th International Conference on        Systems and Informatics (ICSAI), Shanghai, China, 2019, pp. 1408-1412, doi: 10.1109/ICSAI48974.2019.9010071.

## Work Cited

Ang L I , Yi-Xiang L I , Xue-Hui L I . TensorFlow and Keras-based Convolutional Neural Network in CAT Image Recognition. 2017.
Wisanlaya Pornprakun, et al. Determining optimal policies for sugarcane harvesting in Thailand using bi-objective and quasi-Newton optimization methods. Advances in Difference Equations, 2019, Vol.2019 (1), pp.1-15
Dayhoff J E, Deleo J M. Artificial neural networks. Cancer, 2001, 91(8):1615-1634. 

# Our Trained Networks

In this study, we’ve written Python code to:
  •	 Explore and Process the Dataset
  •	Build and Train our NN
  •	Visualize  Accuracy and Loss
  •	Add Regularization to NN
We've been through a lot, but we haven't written too much code! It only took about 4 to 5 lines of code to construct and train our Neural Network, and playing with different model architectures is as easy as swapping in different layers or modifying different hyperparameters.

We have trained three different networks and achieved an highest accuracy of 90.69%. Training and testing accuracy of our bestfitted model is shown in figure 2.

![download](https://user-images.githubusercontent.com/81248615/112745201-0b3d6d80-8fc0-11eb-987d-0c2d6712a680.png)



