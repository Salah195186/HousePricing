# Prediction of House Price Based on The Back Propagation Neural Network in The Keras Deep Learning Framework

#  Introduction

House prices have been increasing in recent years, in tandem with the global economy's rapid growth. The real estate sector has gradually grown to become a significant pillar of the global economy. In this environment, nearly everybody is paying attention to the home market trend and attempting to use more empirical and effective approaches to make correct housing price forecasts. Aside from the effect of the house's characteristics, other factors, especially the characteristics of the parties to the sale, have an impact on the house's price. The house price dilemma can be seen as a large open complex system with a lot of complexity, volatility, nonlinearity, and dynamics.



This study uses the chain home network's housing data to forecast the price of used housing in Shanghai. To begin, this paper uses crawler technologies to decode the URL text information using the BeautifulSoup parser and the json request address. Then, using the Keras deep learning library, a multi-layer feedforward neural network model is equipped using the error inverse propagation algorithm. 

The data used in this paper was crawled from the Web Host using crawler technology, and experiments were performed using a back propagation neural network (BP neural network) model based on the Keras paradigm. The BP neural network is a multilayer feedforward network that has been trained using the error inverse propagation algorithm. It's one of the most common neural network models out there. It can learn and store a huge number of input-output mode mappings without disclosing the mapping relationship's mathematical equations. Its learning rule is to use the gradient steepest descent approach to continuously change the network's weight and threshold through error back propagation until the network's square error reaches a minimum. For housing price forecasting, 12 variables influencing real estate are chosen; the selected indicators have different aspects such as geographic coordinates, traffic orientation, and housing type. The abnormal value was processed using random forest during the experiment, and a better prediction outcome was obtained as a result.


### Keras 

Keras is a Python-based neural network library with a high degree of modularity. The backend may be either TensorFlow or Theano [1]. Keras also includes modules such as activation function modules, sheet modules, preprocessing modules, objective function modules, optimization system selection modules, and so on. Activation function modules and optimization process selection modules [2] are two of them, and they combine all of the most recent and best optimization approaches and activation functions. Network models can be easily developed with these components, and core parameters of neural networks can be enhanced.

### Back Propagation NN

The BP NN [3] is a multi-layer feedforward NN trained by using the error back propagation algorithm suggested by the Mccelland  and Rumelhart in 1986. It typically consists of three layers: input, hidden, and output. Figure 1 depicts the network structure. 

# Results 

The analysis with the absolute value of the relative error between the expected value and the real value is within 5%, accounting for 95.59 percent, according to the measurement and statistical result. The neural network built by other documents of the same type was applied to the data of this experiment for comparison in order to verify the validity of the experiment. The results of the studies indicate that the NN developed in this analysis are marginally better than previous neural networks and have good model performance.
