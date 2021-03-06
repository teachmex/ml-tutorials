# Neural Nets :Tuning


```python
%matplotlib inline
%load_ext tensorboard
```


```python
import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
import os
import pandas as pd
import sklearn
import sys
import tensorflow as tf
from tensorflow import keras  # tf.keras
import time
import seaborn as sns
sns.set()
```


```python
print("python", sys.version)
for module in mpl, np, pd, sklearn, tf, keras:
    print(module.__name__, module.__version__)
```

    python 3.7.1 (default, Dec 14 2018, 13:28:58) 
    [Clang 4.0.1 (tags/RELEASE_401/final)]
    matplotlib 3.0.2
    numpy 1.15.4
    pandas 0.23.4
    sklearn 0.20.1
    tensorflow 2.0.0-beta0
    tensorflow.python.keras.api._v2.keras 2.2.4-tf



```python
assert sys.version_info >= (3, 5) # Python ≥3.5 required
assert tf.__version__ >= "2.0"    # TensorFlow ≥2.0 required
```

### A neural net for regression

Load the California housing dataset using `sklearn.datasets.fetch_california_housing`. This returns an object with a `DESCR` attribute describing the dataset, a `data` attribute with the input features, and a `target` attribute with the labels. The goal is to predict the price of houses in a district (a census block) given some stats about that district. This is a regression task (predicting values).


```python
from sklearn.datasets import fetch_california_housing
housing = fetch_california_housing()
```


```python
print(housing.DESCR)
```

    .. _california_housing_dataset:
    
    California Housing dataset
    --------------------------
    
    **Data Set Characteristics:**
    
        :Number of Instances: 20640
    
        :Number of Attributes: 8 numeric, predictive attributes and the target
    
        :Attribute Information:
            - MedInc        median income in block
            - HouseAge      median house age in block
            - AveRooms      average number of rooms
            - AveBedrms     average number of bedrooms
            - Population    block population
            - AveOccup      average house occupancy
            - Latitude      house block latitude
            - Longitude     house block longitude
    
        :Missing Attribute Values: None
    
    This dataset was obtained from the StatLib repository.
    http://lib.stat.cmu.edu/datasets/
    
    The target variable is the median house value for California districts.
    
    This dataset was derived from the 1990 U.S. census, using one row per census
    block group. A block group is the smallest geographical unit for which the U.S.
    Census Bureau publishes sample data (a block group typically has a population
    of 600 to 3,000 people).
    
    It can be downloaded/loaded using the
    :func:`sklearn.datasets.fetch_california_housing` function.
    
    .. topic:: References
    
        - Pace, R. Kelley and Ronald Barry, Sparse Spatial Autoregressions,
          Statistics and Probability Letters, 33 (1997) 291-297
    



```python
housing.data.shape
```




    (20640, 8)




```python
housing.target.shape
```




    (20640,)



Split the dataset into a training set, a validation set and a test set using Scikit-Learn's `sklearn.model_selection.train_test_split()` function.


```python
from sklearn.model_selection import train_test_split

X_train_full, X_test, y_train_full, y_test = train_test_split(housing.data, housing.target, random_state=42)
X_train, X_valid, y_train, y_valid = train_test_split(X_train_full, y_train_full, random_state=42)
```


```python
len(X_train), len(X_valid), len(X_test)
```




    (11610, 3870, 5160)



Scale the input features (e.g., using a `sklearn.preprocessing.StandardScaler`). Once again, don't forget that you should not fit the validation set or the test set, only the training set.


```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_valid_scaled = scaler.transform(X_valid)
X_test_scaled = scaler.transform(X_test)
```

Now build, train and evaluate a neural network to tackle this problem. Then use it to make predictions on the test set.

**Tips**:
* Since you are predicting a single value per district (the median house price), there should only be one neuron in the output layer.
* Usually for regression tasks you don't want to use any activation function in the output layer (in some cases you may want to use `"relu"` or `"softplus"` if you want to constrain the predicted values to be positive, or `"sigmoid"` or `"tanh"` if you want to constrain the predicted values to 0-1 or -1-1).
* A good loss function for regression is generally the `"mean_squared_error"` (aka `"mse"`). When there are many outliers in your dataset, you may prefer to use the `"mean_absolute_error"` (aka `"mae"`), which is a bit less precise but less sensitive to outliers.


```python
model = keras.models.Sequential([
    
    keras.layers.Dense(30, activation="relu",\
                       input_shape=X_train.shape[1:]),
    keras.layers.Dense(1)
    
])
```


```python
model.compile(loss="mean_squared_error",\
              optimizer=keras.optimizers.SGD(1e-3))
```


```python
callbacks = [keras.callbacks.EarlyStopping(patience=10)]
```


```python
history = model.fit(X_train_scaled,\
                    y_train,
                    validation_data=(X_valid_scaled, y_valid),\
                    epochs=100,
                    callbacks=callbacks)
```

    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 41us/sample - loss: 1.4373 - val_loss: 4.9912
    Epoch 2/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.8371 - val_loss: 0.8625
    Epoch 3/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.7555 - val_loss: 0.7340
    Epoch 4/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.7094 - val_loss: 0.7095
    Epoch 5/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6720 - val_loss: 0.6471
    Epoch 6/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6392 - val_loss: 0.6335
    Epoch 7/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6105 - val_loss: 0.5858
    Epoch 8/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5852 - val_loss: 0.5641
    Epoch 9/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5626 - val_loss: 0.5456
    Epoch 10/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5425 - val_loss: 0.5173
    Epoch 11/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5253 - val_loss: 0.5070
    Epoch 12/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5096 - val_loss: 0.4918
    Epoch 13/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4964 - val_loss: 0.4793
    Epoch 14/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4848 - val_loss: 0.4752
    Epoch 15/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4743 - val_loss: 0.4597
    Epoch 16/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4658 - val_loss: 0.4685
    Epoch 17/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4586 - val_loss: 0.4433
    Epoch 18/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4520 - val_loss: 0.4364
    Epoch 19/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4464 - val_loss: 0.4511
    Epoch 20/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4418 - val_loss: 0.4285
    Epoch 21/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4377 - val_loss: 0.4347
    Epoch 22/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4341 - val_loss: 0.4288
    Epoch 23/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4309 - val_loss: 0.4265
    Epoch 24/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4281 - val_loss: 0.4219
    Epoch 25/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4256 - val_loss: 0.4368
    Epoch 26/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4233 - val_loss: 0.4404
    Epoch 27/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4210 - val_loss: 0.4438
    Epoch 28/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4192 - val_loss: 0.4358
    Epoch 29/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4172 - val_loss: 0.4398
    Epoch 30/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4154 - val_loss: 0.4463
    Epoch 31/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4141 - val_loss: 0.4247
    Epoch 32/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4126 - val_loss: 0.4308
    Epoch 33/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4112 - val_loss: 0.4330
    Epoch 34/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4099 - val_loss: 0.4273



```python
model.evaluate(X_test_scaled, y_test)
```

    5160/5160 [==============================] - 0s 15us/sample - loss: 0.4072





    0.40716409780258356




```python
model.predict(X_test_scaled)
```




    array([[0.6690421],
           [1.6140628],
           [3.5537164],
           ...,
           [1.504305 ],
           [2.528902 ],
           [3.6941016]], dtype=float32)




```python
def plot_learning_curves(history):
    pd.DataFrame(history.history).plot(figsize=(8, 5))
    plt.grid(True)
    plt.gca().set_ylim(0, 1)
    plt.show()
```


```python
plot_learning_curves(history)
```


![png](output_24_0.png)


### Hyperparameter search

Try training your model multiple times, with different a learning rate each time (e.g., 1e-4, 3e-4, 1e-3, 3e-3, 3e-2), and compare the learning curves. For this, you need to create a `keras.optimizers.SGD` optimizer and specify the `learning_rate` in its constructor, then pass this `SGD` instance to the `compile()` method using the `optimizer` argument.


```python
learning_rates = [1e-4, 3e-4, 1e-3, 3e-3, 1e-2, 3e-2]
histories = []
for learning_rate in learning_rates:
    model = keras.models.Sequential([
        keras.layers.Dense(30, activation="relu", input_shape=X_train.shape[1:]),
        keras.layers.Dense(1)
    ])
    optimizer = keras.optimizers.SGD(learning_rate)
    model.compile(loss="mean_squared_error", optimizer=optimizer)
    callbacks = [keras.callbacks.EarlyStopping(patience=10)]
    history = model.fit(X_train_scaled, y_train,
                        validation_data=(X_valid_scaled, y_valid), epochs=100,
                        callbacks=callbacks)
    histories.append(history)
```

    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 1s 46us/sample - loss: 5.4083 - val_loss: 4.2292
    Epoch 2/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 3.7043 - val_loss: 3.6743
    Epoch 3/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 2.6746 - val_loss: 4.1190
    Epoch 4/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 2.0397 - val_loss: 4.4212
    Epoch 5/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 1.6337 - val_loss: 4.5097
    Epoch 6/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 1.3677 - val_loss: 4.2815
    Epoch 7/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 1.1891 - val_loss: 3.9109
    Epoch 8/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 1.0654 - val_loss: 3.4672
    Epoch 9/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.9781 - val_loss: 2.9750
    Epoch 10/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.9146 - val_loss: 2.5285
    Epoch 11/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.8676 - val_loss: 2.1245
    Epoch 12/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.8319 - val_loss: 1.7892
    Epoch 13/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.8042 - val_loss: 1.5134
    Epoch 14/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.7823 - val_loss: 1.2938
    Epoch 15/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.7646 - val_loss: 1.1226
    Epoch 16/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.7500 - val_loss: 0.9907
    Epoch 17/100
    11610/11610 [==============================] - 0s 36us/sample - loss: 0.7377 - val_loss: 0.8902
    Epoch 18/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.7272 - val_loss: 0.8149
    Epoch 19/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.7180 - val_loss: 0.7574
    Epoch 20/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.7098 - val_loss: 0.7162
    Epoch 21/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.7025 - val_loss: 0.6860
    Epoch 22/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.6958 - val_loss: 0.6657
    Epoch 23/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6897 - val_loss: 0.6500
    Epoch 24/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.6839 - val_loss: 0.6394
    Epoch 25/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.6786 - val_loss: 0.6314
    Epoch 26/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6734 - val_loss: 0.6258
    Epoch 27/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.6686 - val_loss: 0.6221
    Epoch 28/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.6640 - val_loss: 0.6193
    Epoch 29/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.6595 - val_loss: 0.6173
    Epoch 30/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6553 - val_loss: 0.6154
    Epoch 31/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6512 - val_loss: 0.6134
    Epoch 32/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6472 - val_loss: 0.6118
    Epoch 33/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6433 - val_loss: 0.6101
    Epoch 34/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6396 - val_loss: 0.6085
    Epoch 35/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6360 - val_loss: 0.6073
    Epoch 36/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6325 - val_loss: 0.6056
    Epoch 37/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6290 - val_loss: 0.6041
    Epoch 38/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6256 - val_loss: 0.6026
    Epoch 39/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6224 - val_loss: 0.6003
    Epoch 40/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6192 - val_loss: 0.5986
    Epoch 41/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6160 - val_loss: 0.5954
    Epoch 42/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6130 - val_loss: 0.5924
    Epoch 43/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6100 - val_loss: 0.5901
    Epoch 44/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6070 - val_loss: 0.5885
    Epoch 45/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6042 - val_loss: 0.5857
    Epoch 46/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.6013 - val_loss: 0.5834
    Epoch 47/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.5986 - val_loss: 0.5801
    Epoch 48/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5959 - val_loss: 0.5777
    Epoch 49/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5932 - val_loss: 0.5749
    Epoch 50/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5906 - val_loss: 0.5728
    Epoch 51/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5881 - val_loss: 0.5694
    Epoch 52/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5856 - val_loss: 0.5663
    Epoch 53/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5832 - val_loss: 0.5633
    Epoch 54/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5807 - val_loss: 0.5614
    Epoch 55/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5784 - val_loss: 0.5595
    Epoch 56/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5761 - val_loss: 0.5578
    Epoch 57/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5738 - val_loss: 0.5548
    Epoch 58/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5716 - val_loss: 0.5518
    Epoch 59/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5694 - val_loss: 0.5495
    Epoch 60/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5672 - val_loss: 0.5472
    Epoch 61/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5651 - val_loss: 0.5446
    Epoch 62/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5630 - val_loss: 0.5427
    Epoch 63/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5609 - val_loss: 0.5390
    Epoch 64/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5589 - val_loss: 0.5361
    Epoch 65/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5569 - val_loss: 0.5333
    Epoch 66/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5549 - val_loss: 0.5312
    Epoch 67/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5530 - val_loss: 0.5285
    Epoch 68/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5511 - val_loss: 0.5271
    Epoch 69/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5492 - val_loss: 0.5245
    Epoch 70/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5474 - val_loss: 0.5217
    Epoch 71/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5456 - val_loss: 0.5199
    Epoch 72/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5438 - val_loss: 0.5180
    Epoch 73/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5420 - val_loss: 0.5161
    Epoch 74/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5403 - val_loss: 0.5140
    Epoch 75/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5386 - val_loss: 0.5117
    Epoch 76/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5369 - val_loss: 0.5099
    Epoch 77/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5353 - val_loss: 0.5079
    Epoch 78/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5336 - val_loss: 0.5062
    Epoch 79/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5320 - val_loss: 0.5043
    Epoch 80/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5305 - val_loss: 0.5026
    Epoch 81/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5289 - val_loss: 0.5010
    Epoch 82/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5274 - val_loss: 0.4988
    Epoch 83/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5259 - val_loss: 0.4967
    Epoch 84/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5244 - val_loss: 0.4952
    Epoch 85/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5230 - val_loss: 0.4931
    Epoch 86/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5215 - val_loss: 0.4911
    Epoch 87/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5201 - val_loss: 0.4896
    Epoch 88/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5187 - val_loss: 0.4880
    Epoch 89/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5173 - val_loss: 0.4864
    Epoch 90/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5160 - val_loss: 0.4846
    Epoch 91/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5146 - val_loss: 0.4829
    Epoch 92/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5133 - val_loss: 0.4816
    Epoch 93/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5120 - val_loss: 0.4802
    Epoch 94/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5107 - val_loss: 0.4786
    Epoch 95/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5095 - val_loss: 0.4772
    Epoch 96/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5082 - val_loss: 0.4759
    Epoch 97/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5070 - val_loss: 0.4743
    Epoch 98/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5058 - val_loss: 0.4728
    Epoch 99/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5046 - val_loss: 0.4715
    Epoch 100/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5034 - val_loss: 0.4702
    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 39us/sample - loss: 4.2273 - val_loss: 2.7904
    Epoch 2/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 2.0515 - val_loss: 2.1264
    Epoch 3/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 1.3341 - val_loss: 1.4705
    Epoch 4/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 1.0088 - val_loss: 1.0015
    Epoch 5/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.8489 - val_loss: 0.7896
    Epoch 6/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.7662 - val_loss: 0.7104
    Epoch 7/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.7200 - val_loss: 0.6770
    Epoch 8/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6912 - val_loss: 0.6601
    Epoch 9/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6708 - val_loss: 0.6464
    Epoch 10/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.6548 - val_loss: 0.6345
    Epoch 11/100
    11610/11610 [==============================] - 0s 39us/sample - loss: 0.6411 - val_loss: 0.6242
    Epoch 12/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6291 - val_loss: 0.6284
    Epoch 13/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6183 - val_loss: 0.6130
    Epoch 14/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6082 - val_loss: 0.5974
    Epoch 15/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5988 - val_loss: 0.6021
    Epoch 16/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5903 - val_loss: 0.5907
    Epoch 17/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5823 - val_loss: 0.5736
    Epoch 18/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5747 - val_loss: 0.5723
    Epoch 19/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5676 - val_loss: 0.5652
    Epoch 20/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5610 - val_loss: 0.5589
    Epoch 21/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5547 - val_loss: 0.5514
    Epoch 22/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5487 - val_loss: 0.5474
    Epoch 23/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5432 - val_loss: 0.5348
    Epoch 24/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5379 - val_loss: 0.5247
    Epoch 25/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5329 - val_loss: 0.5162
    Epoch 26/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5282 - val_loss: 0.5123
    Epoch 27/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5236 - val_loss: 0.5157
    Epoch 28/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5194 - val_loss: 0.5115
    Epoch 29/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5154 - val_loss: 0.5064
    Epoch 30/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5116 - val_loss: 0.4988
    Epoch 31/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5079 - val_loss: 0.5025
    Epoch 32/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5045 - val_loss: 0.4979
    Epoch 33/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5012 - val_loss: 0.4884
    Epoch 34/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4981 - val_loss: 0.4813
    Epoch 35/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4951 - val_loss: 0.4756
    Epoch 36/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4922 - val_loss: 0.4775
    Epoch 37/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4895 - val_loss: 0.4719
    Epoch 38/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4869 - val_loss: 0.4719
    Epoch 39/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4844 - val_loss: 0.4638
    Epoch 40/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4821 - val_loss: 0.4610
    Epoch 41/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4798 - val_loss: 0.4614
    Epoch 42/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4777 - val_loss: 0.4602
    Epoch 43/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4756 - val_loss: 0.4555
    Epoch 44/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4735 - val_loss: 0.4536
    Epoch 45/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4716 - val_loss: 0.4511
    Epoch 46/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4697 - val_loss: 0.4465
    Epoch 47/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4679 - val_loss: 0.4472
    Epoch 48/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.4662 - val_loss: 0.4427
    Epoch 49/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4645 - val_loss: 0.4379
    Epoch 50/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4629 - val_loss: 0.4380
    Epoch 51/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4613 - val_loss: 0.4363
    Epoch 52/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4598 - val_loss: 0.4352
    Epoch 53/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4583 - val_loss: 0.4353
    Epoch 54/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4569 - val_loss: 0.4346
    Epoch 55/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4555 - val_loss: 0.4300
    Epoch 56/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4541 - val_loss: 0.4291
    Epoch 57/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4529 - val_loss: 0.4283
    Epoch 58/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4516 - val_loss: 0.4252
    Epoch 59/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4503 - val_loss: 0.4226
    Epoch 60/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4492 - val_loss: 0.4231
    Epoch 61/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4479 - val_loss: 0.4219
    Epoch 62/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4468 - val_loss: 0.4204
    Epoch 63/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4456 - val_loss: 0.4184
    Epoch 64/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4445 - val_loss: 0.4180
    Epoch 65/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4434 - val_loss: 0.4167
    Epoch 66/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4424 - val_loss: 0.4151
    Epoch 67/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4413 - val_loss: 0.4144
    Epoch 68/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4403 - val_loss: 0.4138
    Epoch 69/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4393 - val_loss: 0.4112
    Epoch 70/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4383 - val_loss: 0.4103
    Epoch 71/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4373 - val_loss: 0.4097
    Epoch 72/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4364 - val_loss: 0.4079
    Epoch 73/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4355 - val_loss: 0.4084
    Epoch 74/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4345 - val_loss: 0.4061
    Epoch 75/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.4337 - val_loss: 0.4057
    Epoch 76/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4328 - val_loss: 0.4049
    Epoch 77/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.4320 - val_loss: 0.4044
    Epoch 78/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.4311 - val_loss: 0.4028
    Epoch 79/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4304 - val_loss: 0.4026
    Epoch 80/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4296 - val_loss: 0.4016
    Epoch 81/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4288 - val_loss: 0.4008
    Epoch 82/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4280 - val_loss: 0.3999
    Epoch 83/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4273 - val_loss: 0.3992
    Epoch 84/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4265 - val_loss: 0.3986
    Epoch 85/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4258 - val_loss: 0.3975
    Epoch 86/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4251 - val_loss: 0.3971
    Epoch 87/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4244 - val_loss: 0.3967
    Epoch 88/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4237 - val_loss: 0.3961
    Epoch 89/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4230 - val_loss: 0.3950
    Epoch 90/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4224 - val_loss: 0.3948
    Epoch 91/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4217 - val_loss: 0.3940
    Epoch 92/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4211 - val_loss: 0.3933
    Epoch 93/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4204 - val_loss: 0.3941
    Epoch 94/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4198 - val_loss: 0.3928
    Epoch 95/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4192 - val_loss: 0.3917
    Epoch 96/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4186 - val_loss: 0.3913
    Epoch 97/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4180 - val_loss: 0.3908
    Epoch 98/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4174 - val_loss: 0.3902
    Epoch 99/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4169 - val_loss: 0.3894
    Epoch 100/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4163 - val_loss: 0.3892
    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 38us/sample - loss: 2.4134 - val_loss: 1.3718
    Epoch 2/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.7545 - val_loss: 0.6598
    Epoch 3/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6376 - val_loss: 0.5935
    Epoch 4/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6023 - val_loss: 0.5817
    Epoch 5/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5779 - val_loss: 0.5530
    Epoch 6/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.5579 - val_loss: 0.5253
    Epoch 7/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5411 - val_loss: 0.5445
    Epoch 8/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5265 - val_loss: 0.5110
    Epoch 9/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5136 - val_loss: 0.5042
    Epoch 10/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.5024 - val_loss: 0.4798
    Epoch 11/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4927 - val_loss: 0.4648
    Epoch 12/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4840 - val_loss: 0.4726
    Epoch 13/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4764 - val_loss: 0.4456
    Epoch 14/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4700 - val_loss: 0.4380
    Epoch 15/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4644 - val_loss: 0.4392
    Epoch 16/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4593 - val_loss: 0.4401
    Epoch 17/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4548 - val_loss: 0.4267
    Epoch 18/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4508 - val_loss: 0.4354
    Epoch 19/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4472 - val_loss: 0.4276
    Epoch 20/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4437 - val_loss: 0.4317
    Epoch 21/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4407 - val_loss: 0.4150
    Epoch 22/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4379 - val_loss: 0.4173
    Epoch 23/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4351 - val_loss: 0.4187
    Epoch 24/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4328 - val_loss: 0.4153
    Epoch 25/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4304 - val_loss: 0.4099
    Epoch 26/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4283 - val_loss: 0.4065
    Epoch 27/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4261 - val_loss: 0.4083
    Epoch 28/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4242 - val_loss: 0.4039
    Epoch 29/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4224 - val_loss: 0.3960
    Epoch 30/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4207 - val_loss: 0.3992
    Epoch 31/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4188 - val_loss: 0.3920
    Epoch 32/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4174 - val_loss: 0.4008
    Epoch 33/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4159 - val_loss: 0.3919
    Epoch 34/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.4143 - val_loss: 0.4006
    Epoch 35/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.4129 - val_loss: 0.3990
    Epoch 36/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4116 - val_loss: 0.3882
    Epoch 37/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4104 - val_loss: 0.3860
    Epoch 38/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4090 - val_loss: 0.3875
    Epoch 39/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4079 - val_loss: 0.3806
    Epoch 40/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4068 - val_loss: 0.3814
    Epoch 41/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4056 - val_loss: 0.3827
    Epoch 42/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4045 - val_loss: 0.3786
    Epoch 43/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4035 - val_loss: 0.3812
    Epoch 44/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4024 - val_loss: 0.3789
    Epoch 45/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4015 - val_loss: 0.3787
    Epoch 46/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.4005 - val_loss: 0.3805
    Epoch 47/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3995 - val_loss: 0.3754
    Epoch 48/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3987 - val_loss: 0.3753
    Epoch 49/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3977 - val_loss: 0.3772
    Epoch 50/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3967 - val_loss: 0.3706
    Epoch 51/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3958 - val_loss: 0.3808
    Epoch 52/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3950 - val_loss: 0.3865
    Epoch 53/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3942 - val_loss: 0.3874
    Epoch 54/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3933 - val_loss: 0.3936
    Epoch 55/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3927 - val_loss: 0.3897
    Epoch 56/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3919 - val_loss: 0.3940
    Epoch 57/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3912 - val_loss: 0.3777
    Epoch 58/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3905 - val_loss: 0.3831
    Epoch 59/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3898 - val_loss: 0.3797
    Epoch 60/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3892 - val_loss: 0.3834
    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 39us/sample - loss: 1.3072 - val_loss: 5.4533
    Epoch 2/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.6424 - val_loss: 2.8452
    Epoch 3/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.5206 - val_loss: 2.5899
    Epoch 4/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4927 - val_loss: 6.1177
    Epoch 5/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4958 - val_loss: 7.5540
    Epoch 6/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4943 - val_loss: 3.3703
    Epoch 7/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4485 - val_loss: 0.7394
    Epoch 8/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4171 - val_loss: 0.4349
    Epoch 9/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4102 - val_loss: 0.4125
    Epoch 10/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4055 - val_loss: 0.3867
    Epoch 11/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4017 - val_loss: 0.3774
    Epoch 12/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3982 - val_loss: 0.3820
    Epoch 13/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3944 - val_loss: 0.4447
    Epoch 14/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3926 - val_loss: 0.3701
    Epoch 15/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3900 - val_loss: 0.3717
    Epoch 16/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3876 - val_loss: 0.3667
    Epoch 17/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3851 - val_loss: 0.3589
    Epoch 18/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3829 - val_loss: 0.3570
    Epoch 19/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3807 - val_loss: 0.3977
    Epoch 20/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3802 - val_loss: 0.4129
    Epoch 21/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3783 - val_loss: 0.4629
    Epoch 22/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3772 - val_loss: 0.3535
    Epoch 23/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3749 - val_loss: 0.3909
    Epoch 24/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3742 - val_loss: 0.3567
    Epoch 25/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3726 - val_loss: 0.3504
    Epoch 26/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3710 - val_loss: 0.4323
    Epoch 27/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3716 - val_loss: 0.3593
    Epoch 28/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3697 - val_loss: 0.3478
    Epoch 29/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3688 - val_loss: 0.3642
    Epoch 30/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3681 - val_loss: 0.3518
    Epoch 31/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3669 - val_loss: 0.3702
    Epoch 32/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3662 - val_loss: 0.3608
    Epoch 33/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3656 - val_loss: 0.3444
    Epoch 34/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3649 - val_loss: 0.3676
    Epoch 35/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3645 - val_loss: 0.4251
    Epoch 36/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3644 - val_loss: 0.6289
    Epoch 37/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3664 - val_loss: 0.4964
    Epoch 38/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3626 - val_loss: 0.9082
    Epoch 39/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3679 - val_loss: 0.7426
    Epoch 40/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3645 - val_loss: 0.8972
    Epoch 41/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3665 - val_loss: 0.6627
    Epoch 42/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3618 - val_loss: 0.8439
    Epoch 43/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3624 - val_loss: 0.8183
    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 38us/sample - loss: 0.7428 - val_loss: 0.5310
    Epoch 2/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4772 - val_loss: 0.7309
    Epoch 3/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4394 - val_loss: 0.4341
    Epoch 4/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4377 - val_loss: 0.9035
    Epoch 5/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4206 - val_loss: 2.1735
    Epoch 6/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4003 - val_loss: 3.6124
    Epoch 7/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4402 - val_loss: 5.5876
    Epoch 8/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.4307 - val_loss: 1.7575
    Epoch 9/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3921 - val_loss: 0.3940
    Epoch 10/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3885 - val_loss: 0.4234
    Epoch 11/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3743 - val_loss: 0.4203
    Epoch 12/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3730 - val_loss: 0.4551
    Epoch 13/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3709 - val_loss: 0.3796
    Epoch 14/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3668 - val_loss: 0.3914
    Epoch 15/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3675 - val_loss: 0.3904
    Epoch 16/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3647 - val_loss: 0.4041
    Epoch 17/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3670 - val_loss: 0.4419
    Epoch 18/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3622 - val_loss: 0.3666
    Epoch 19/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3594 - val_loss: 0.3740
    Epoch 20/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3613 - val_loss: 0.3728
    Epoch 21/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3586 - val_loss: 0.4143
    Epoch 22/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3552 - val_loss: 0.3474
    Epoch 23/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3556 - val_loss: 0.3595
    Epoch 24/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3694 - val_loss: 0.3725
    Epoch 25/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3554 - val_loss: 0.3636
    Epoch 26/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3517 - val_loss: 0.3741
    Epoch 27/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3532 - val_loss: 0.3486
    Epoch 28/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3495 - val_loss: 0.3586
    Epoch 29/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3587 - val_loss: 0.3759
    Epoch 30/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3484 - val_loss: 0.3318
    Epoch 31/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3463 - val_loss: 0.3511
    Epoch 32/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3460 - val_loss: 0.3682
    Epoch 33/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3453 - val_loss: 0.5032
    Epoch 34/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3471 - val_loss: 0.3380
    Epoch 35/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3419 - val_loss: 0.3651
    Epoch 36/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3407 - val_loss: 0.3318
    Epoch 37/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3433 - val_loss: 0.3625
    Epoch 38/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3430 - val_loss: 0.3544
    Epoch 39/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3370 - val_loss: 0.3249
    Epoch 40/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3412 - val_loss: 0.3564
    Epoch 41/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3378 - val_loss: 0.3569
    Epoch 42/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3343 - val_loss: 0.3182
    Epoch 43/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3381 - val_loss: 0.3800
    Epoch 44/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3325 - val_loss: 0.3294
    Epoch 45/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3336 - val_loss: 0.3423
    Epoch 46/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3324 - val_loss: 0.3299
    Epoch 47/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3344 - val_loss: 0.3148
    Epoch 48/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3298 - val_loss: 0.3498
    Epoch 49/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3413 - val_loss: 0.3286
    Epoch 50/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3271 - val_loss: 0.5932
    Epoch 51/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3313 - val_loss: 0.3845
    Epoch 52/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3261 - val_loss: 0.6172
    Epoch 53/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3282 - val_loss: 0.6108
    Epoch 54/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3302 - val_loss: 0.3378
    Epoch 55/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3258 - val_loss: 0.4014
    Epoch 56/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3244 - val_loss: 0.3276
    Epoch 57/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3222 - val_loss: 0.4102
    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 38us/sample - loss: 0.6626 - val_loss: 1.9500
    Epoch 2/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4895 - val_loss: 0.4488
    Epoch 3/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4463 - val_loss: 4.8228
    Epoch 4/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3891 - val_loss: 8.5697
    Epoch 5/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4137 - val_loss: 0.3434
    Epoch 6/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3676 - val_loss: 0.3476
    Epoch 7/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3674 - val_loss: 0.4051
    Epoch 8/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.4190 - val_loss: 0.3482
    Epoch 9/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3666 - val_loss: 0.3420
    Epoch 10/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3602 - val_loss: 0.3352
    Epoch 11/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3579 - val_loss: 0.3361
    Epoch 12/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3557 - val_loss: 0.3494
    Epoch 13/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3565 - val_loss: 0.3343
    Epoch 14/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3527 - val_loss: 0.3270
    Epoch 15/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3491 - val_loss: 0.3259
    Epoch 16/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3452 - val_loss: 0.3255
    Epoch 17/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3875 - val_loss: 0.3663
    Epoch 18/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3572 - val_loss: 0.3324
    Epoch 19/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3551 - val_loss: 0.3254
    Epoch 20/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3532 - val_loss: 0.3350
    Epoch 21/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3404 - val_loss: 0.3183
    Epoch 22/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3382 - val_loss: 0.4119
    Epoch 23/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3356 - val_loss: 0.3146
    Epoch 24/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3334 - val_loss: 0.3138
    Epoch 25/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3315 - val_loss: 0.3144
    Epoch 26/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3301 - val_loss: 0.3158
    Epoch 27/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3263 - val_loss: 0.3053
    Epoch 28/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3251 - val_loss: 0.3817
    Epoch 29/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3245 - val_loss: 0.3011
    Epoch 30/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3224 - val_loss: 0.3064
    Epoch 31/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3211 - val_loss: 0.3338
    Epoch 32/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3206 - val_loss: 0.2942
    Epoch 33/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3185 - val_loss: 0.3047
    Epoch 34/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3182 - val_loss: 0.2964
    Epoch 35/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3240 - val_loss: 0.2974
    Epoch 36/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3160 - val_loss: 0.2945
    Epoch 37/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3143 - val_loss: 0.2956
    Epoch 38/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3138 - val_loss: 0.2969
    Epoch 39/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3122 - val_loss: 0.2938
    Epoch 40/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3106 - val_loss: 0.2912
    Epoch 41/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3121 - val_loss: 0.3569
    Epoch 42/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3137 - val_loss: 0.2975
    Epoch 43/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3093 - val_loss: 0.2939
    Epoch 44/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3078 - val_loss: 0.2913
    Epoch 45/100
    11610/11610 [==============================] - 0s 29us/sample - loss: 0.3082 - val_loss: 0.2980
    Epoch 46/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3084 - val_loss: 0.2925
    Epoch 47/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3104 - val_loss: 0.2883
    Epoch 48/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3075 - val_loss: 0.2887
    Epoch 49/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3050 - val_loss: 0.3013
    Epoch 50/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3027 - val_loss: 0.3024
    Epoch 51/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3049 - val_loss: 0.3493
    Epoch 52/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3023 - val_loss: 0.3955
    Epoch 53/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3038 - val_loss: 0.2910
    Epoch 54/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3037 - val_loss: 0.3976
    Epoch 55/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3027 - val_loss: 0.2860
    Epoch 56/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3010 - val_loss: 0.2943
    Epoch 57/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3011 - val_loss: 0.2864
    Epoch 58/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3035 - val_loss: 0.3168
    Epoch 59/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3008 - val_loss: 0.2943
    Epoch 60/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3017 - val_loss: 0.2994
    Epoch 61/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3009 - val_loss: 0.3375
    Epoch 62/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3075 - val_loss: 0.3327
    Epoch 63/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3022 - val_loss: 0.3012
    Epoch 64/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3009 - val_loss: 0.3123
    Epoch 65/100
    11610/11610 [==============================] - 0s 30us/sample - loss: 0.3008 - val_loss: 0.2923



```python
for learning_rate, history in zip(learning_rates, histories):
    print("Learning rate:", learning_rate)
    plot_learning_curves(history)
```

    Learning rate: 0.0001



![png](output_28_1.png)


    Learning rate: 0.0003



![png](output_28_3.png)


    Learning rate: 0.001



![png](output_28_5.png)


    Learning rate: 0.003



![png](output_28_7.png)


    Learning rate: 0.01



![png](output_28_9.png)


    Learning rate: 0.03



![png](output_28_11.png)


Let's look at a more sophisticated way to tune hyperparameters. Create a `build_model()` function that takes three arguments, `n_hidden`, `n_neurons`, `learning_rate`, and builds, compiles and returns a model with the given number of hidden layers, the given number of neurons and the given learning rate. It is good practice to give a reasonable default value to each argument.


```python
def build_model(n_hidden=1, n_neurons=30, learning_rate=3e-3):
    model = keras.models.Sequential()
    options = {"input_shape": X_train.shape[1:]}
    for layer in range(n_hidden + 1):
        model.add(keras.layers.Dense(n_neurons, activation="relu", **options))
        options = {}
    model.add(keras.layers.Dense(1, **options))
    optimizer = keras.optimizers.SGD(learning_rate)
    model.compile(loss="mse", optimizer=optimizer)
    return model
```

Create a `keras.wrappers.scikit_learn.KerasRegressor` and pass the `build_model` function to the constructor. This gives you a Scikit-Learn compatible predictor. Try training it and using it to make predictions. Note that you can pass the `n_epochs`, `callbacks` and `validation_data` to the `fit()` method.


```python
keras_reg = keras.wrappers.scikit_learn.KerasRegressor(build_model)
```


```python
keras_reg.fit(X_train_scaled, y_train, epochs=100,
              validation_data=(X_valid_scaled, y_valid),
              callbacks=[keras.callbacks.EarlyStopping(patience=10)])
```

    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 0s 41us/sample - loss: 0.9440 - val_loss: 9.4997
    Epoch 2/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.6070 - val_loss: 34.8291
    Epoch 3/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.6444 - val_loss: 2.0556
    Epoch 4/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.4426 - val_loss: 0.4424
    Epoch 5/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.4072 - val_loss: 0.3828
    Epoch 6/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3978 - val_loss: 0.3810
    Epoch 7/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3908 - val_loss: 0.3813
    Epoch 8/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3859 - val_loss: 0.3877
    Epoch 9/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3821 - val_loss: 0.3704
    Epoch 10/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3788 - val_loss: 0.3779
    Epoch 11/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3761 - val_loss: 0.3823
    Epoch 12/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3742 - val_loss: 0.3783
    Epoch 13/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3715 - val_loss: 0.3787
    Epoch 14/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3689 - val_loss: 0.3870
    Epoch 15/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3677 - val_loss: 0.3929
    Epoch 16/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.3653 - val_loss: 0.3907
    Epoch 17/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3632 - val_loss: 0.3841
    Epoch 18/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.3628 - val_loss: 0.3691
    Epoch 19/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3606 - val_loss: 0.3888
    Epoch 20/100
    11610/11610 [==============================] - 0s 31us/sample - loss: 0.3593 - val_loss: 0.3602
    Epoch 21/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3577 - val_loss: 0.3654
    Epoch 22/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3569 - val_loss: 0.3606
    Epoch 23/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3551 - val_loss: 0.4164
    Epoch 24/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3544 - val_loss: 0.3564
    Epoch 25/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.3533 - val_loss: 0.3717
    Epoch 26/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3522 - val_loss: 0.3663
    Epoch 27/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3516 - val_loss: 0.3929
    Epoch 28/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3508 - val_loss: 0.3429
    Epoch 29/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3493 - val_loss: 0.3521
    Epoch 30/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.3488 - val_loss: 0.3937
    Epoch 31/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3478 - val_loss: 0.3611
    Epoch 32/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3467 - val_loss: 0.3792
    Epoch 33/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3455 - val_loss: 0.3740
    Epoch 34/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3452 - val_loss: 0.3629
    Epoch 35/100
    11610/11610 [==============================] - 0s 33us/sample - loss: 0.3443 - val_loss: 0.3792
    Epoch 36/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3432 - val_loss: 0.3710
    Epoch 37/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3428 - val_loss: 0.3737
    Epoch 38/100
    11610/11610 [==============================] - 0s 32us/sample - loss: 0.3420 - val_loss: 0.3575





    <tensorflow.python.keras.callbacks.History at 0x1a3b91d9b0>




```python
keras_reg.predict(X_test_scaled)
```




    array([0.76681733, 1.8267894 , 4.295369  , ..., 1.3182094 , 2.648156  ,
           4.0545692 ], dtype=float32)



Use a `sklearn.model_selection.RandomizedSearchCV` to search the hyperparameter space of your `KerasRegressor`.


```python
from scipy.stats import reciprocal

param_distribs = {
    "n_hidden": [0, 1, 2, 3],
    "n_neurons": np.arange(1, 100),
    "learning_rate": reciprocal(3e-4, 3e-2),
}
```


```python
from sklearn.model_selection import RandomizedSearchCV

rnd_search_cv = RandomizedSearchCV(keras_reg, param_distribs, n_iter=10, cv=3, verbose=2)
```


```python
rnd_search_cv.fit(X_train_scaled, y_train, epochs=100,
                  validation_data=(X_valid_scaled, y_valid),
                  callbacks=[keras.callbacks.EarlyStopping(patience=10)])
```

    Fitting 3 folds for each of 10 candidates, totalling 30 fits
    [CV] learning_rate=0.0023302047292153563, n_hidden=2, n_neurons=41 ...


    [Parallel(n_jobs=1)]: Using backend SequentialBackend with 1 concurrent workers.


    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 61us/sample - loss: 1.6032 - val_loss: 1.7070
    Epoch 2/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.6348 - val_loss: 0.5833
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5404 - val_loss: 0.4916
    Epoch 4/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4899 - val_loss: 0.4363
    Epoch 5/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4539 - val_loss: 0.4108
    Epoch 6/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4299 - val_loss: 0.4061
    Epoch 7/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4147 - val_loss: 0.4094
    Epoch 8/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4044 - val_loss: 0.4162
    Epoch 9/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3966 - val_loss: 0.4136
    Epoch 10/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3907 - val_loss: 0.4069
    Epoch 11/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3860 - val_loss: 0.4050
    Epoch 12/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3811 - val_loss: 0.4054
    Epoch 13/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3777 - val_loss: 0.4089
    Epoch 14/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3745 - val_loss: 0.4118
    Epoch 15/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3712 - val_loss: 0.4270
    Epoch 16/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3682 - val_loss: 0.3820
    Epoch 17/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3652 - val_loss: 0.3883
    Epoch 18/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3628 - val_loss: 0.4042
    Epoch 19/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3605 - val_loss: 0.3741
    Epoch 20/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3581 - val_loss: 0.3736
    Epoch 21/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3557 - val_loss: 0.3662
    Epoch 22/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3535 - val_loss: 0.3517
    Epoch 23/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3511 - val_loss: 0.3583
    Epoch 24/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3493 - val_loss: 0.3570
    Epoch 25/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3477 - val_loss: 0.3631
    Epoch 26/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3463 - val_loss: 0.3423
    Epoch 27/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3446 - val_loss: 0.3695
    Epoch 28/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3420 - val_loss: 0.3397
    Epoch 29/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3414 - val_loss: 0.3492
    Epoch 30/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3404 - val_loss: 0.3500
    Epoch 31/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3385 - val_loss: 0.3407
    Epoch 32/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3379 - val_loss: 0.3365
    Epoch 33/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3365 - val_loss: 0.3482
    Epoch 34/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3353 - val_loss: 0.3435
    Epoch 35/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3334 - val_loss: 0.3367
    Epoch 36/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3329 - val_loss: 0.3609
    Epoch 37/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3318 - val_loss: 0.3485
    Epoch 38/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3304 - val_loss: 0.3455
    Epoch 39/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3301 - val_loss: 0.3264
    Epoch 40/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3291 - val_loss: 0.3259
    Epoch 41/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3274 - val_loss: 0.3431
    Epoch 42/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3265 - val_loss: 0.3371
    Epoch 43/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3258 - val_loss: 0.3332
    Epoch 44/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3249 - val_loss: 0.3310
    Epoch 45/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3240 - val_loss: 0.3318
    Epoch 46/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3236 - val_loss: 0.3268
    Epoch 47/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3222 - val_loss: 0.3270
    Epoch 48/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3215 - val_loss: 0.3296
    Epoch 49/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3204 - val_loss: 0.3274
    Epoch 50/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3198 - val_loss: 0.3180
    Epoch 51/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3185 - val_loss: 0.3408
    Epoch 52/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3177 - val_loss: 0.3154
    Epoch 53/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3173 - val_loss: 0.3288
    Epoch 54/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3154 - val_loss: 0.3332
    Epoch 55/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3148 - val_loss: 0.3334
    Epoch 56/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3145 - val_loss: 0.3171
    Epoch 57/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3140 - val_loss: 0.3272
    Epoch 58/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3129 - val_loss: 0.3203
    Epoch 59/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3125 - val_loss: 0.3195
    Epoch 60/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3119 - val_loss: 0.3197
    Epoch 61/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3101 - val_loss: 0.3084
    Epoch 62/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3102 - val_loss: 0.3236
    Epoch 63/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3092 - val_loss: 0.3236
    Epoch 64/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3089 - val_loss: 0.3137
    Epoch 65/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3076 - val_loss: 0.3309
    Epoch 66/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3071 - val_loss: 0.3164
    Epoch 67/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3066 - val_loss: 0.3116
    Epoch 68/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3053 - val_loss: 0.3275
    Epoch 69/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3058 - val_loss: 0.3333
    Epoch 70/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3058 - val_loss: 0.3057
    Epoch 71/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3042 - val_loss: 0.3118
    Epoch 72/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3034 - val_loss: 0.3156
    Epoch 73/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3023 - val_loss: 0.3021
    Epoch 74/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3018 - val_loss: 0.3117
    Epoch 75/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3012 - val_loss: 0.3137
    Epoch 76/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3009 - val_loss: 0.3042
    Epoch 77/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3001 - val_loss: 0.3010
    Epoch 78/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2996 - val_loss: 0.2999
    Epoch 79/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2981 - val_loss: 0.3223
    Epoch 80/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2989 - val_loss: 0.3033
    Epoch 81/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2978 - val_loss: 0.3024
    Epoch 82/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2972 - val_loss: 0.2972
    Epoch 83/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2968 - val_loss: 0.3024
    Epoch 84/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2964 - val_loss: 0.2988
    Epoch 85/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2953 - val_loss: 0.3088
    Epoch 86/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2944 - val_loss: 0.3030
    Epoch 87/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2944 - val_loss: 0.2996
    Epoch 88/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2941 - val_loss: 0.2984
    Epoch 89/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2934 - val_loss: 0.3058
    Epoch 90/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2928 - val_loss: 0.3072
    Epoch 91/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2923 - val_loss: 0.3268
    Epoch 92/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.2918 - val_loss: 0.3025
    3870/3870 [==============================] - 0s 14us/sample - loss: 0.3291
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.2901
    [CV]  learning_rate=0.0023302047292153563, n_hidden=2, n_neurons=41, total=  27.1s
    [CV] learning_rate=0.0023302047292153563, n_hidden=2, n_neurons=41 ...


    [Parallel(n_jobs=1)]: Done   1 out of   1 | elapsed:   27.3s remaining:    0.0s


    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 55us/sample - loss: 1.3620 - val_loss: 1.9705
    Epoch 2/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5921 - val_loss: 0.6401
    Epoch 3/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5145 - val_loss: 0.4716
    Epoch 4/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4709 - val_loss: 0.4411
    Epoch 5/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4432 - val_loss: 0.4216
    Epoch 6/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4248 - val_loss: 0.3981
    Epoch 7/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4126 - val_loss: 0.3831
    Epoch 8/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4020 - val_loss: 0.3851
    Epoch 9/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3949 - val_loss: 0.4085
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3881 - val_loss: 0.4479
    Epoch 11/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3839 - val_loss: 0.4876
    Epoch 12/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3786 - val_loss: 0.5219
    Epoch 13/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3754 - val_loss: 0.6181
    Epoch 14/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3716 - val_loss: 0.6809
    Epoch 15/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3688 - val_loss: 0.7110
    Epoch 16/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3663 - val_loss: 0.7682
    Epoch 17/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3642 - val_loss: 0.8752
    3870/3870 [==============================] - 0s 14us/sample - loss: 0.3741
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.3604
    [CV]  learning_rate=0.0023302047292153563, n_hidden=2, n_neurons=41, total=   5.5s
    [CV] learning_rate=0.0023302047292153563, n_hidden=2, n_neurons=41 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 55us/sample - loss: 1.5622 - val_loss: 0.6830
    Epoch 2/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.6203 - val_loss: 0.5885
    Epoch 3/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5482 - val_loss: 0.5091
    Epoch 4/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4988 - val_loss: 0.4652
    Epoch 5/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4643 - val_loss: 0.4598
    Epoch 6/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4398 - val_loss: 0.4594
    Epoch 7/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4229 - val_loss: 0.3934
    Epoch 8/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4119 - val_loss: 0.5014
    Epoch 9/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4040 - val_loss: 0.4281
    Epoch 10/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3975 - val_loss: 0.3707
    Epoch 11/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3922 - val_loss: 0.4708
    Epoch 12/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3880 - val_loss: 0.4576
    Epoch 13/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3850 - val_loss: 0.3639
    Epoch 14/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3817 - val_loss: 0.4434
    Epoch 15/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3793 - val_loss: 0.3625
    Epoch 16/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3756 - val_loss: 0.4428
    Epoch 17/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3742 - val_loss: 0.3508
    Epoch 18/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3716 - val_loss: 0.3632
    Epoch 19/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3692 - val_loss: 0.4190
    Epoch 20/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3664 - val_loss: 0.4335
    Epoch 21/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3653 - val_loss: 0.3813
    Epoch 22/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3628 - val_loss: 0.4380
    Epoch 23/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3617 - val_loss: 0.3523
    Epoch 24/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3600 - val_loss: 0.3579
    Epoch 25/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3582 - val_loss: 0.4265
    Epoch 26/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3569 - val_loss: 0.4404
    Epoch 27/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3556 - val_loss: 0.3602
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3564
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.3544
    [CV]  learning_rate=0.0023302047292153563, n_hidden=2, n_neurons=41, total=   8.3s
    [CV] learning_rate=0.015956195942385693, n_hidden=0, n_neurons=97 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 47us/sample - loss: 0.7604 - val_loss: 5.4232
    Epoch 2/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.6441 - val_loss: 14.1695
    Epoch 3/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.6713 - val_loss: 3.7025
    Epoch 4/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5019 - val_loss: 4.6624
    Epoch 5/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.8346 - val_loss: 49.8473
    Epoch 6/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 3.0755 - val_loss: 0.3787
    Epoch 7/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4175 - val_loss: 0.3511
    Epoch 8/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3728 - val_loss: 0.3492
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5112 - val_loss: 0.3382
    Epoch 10/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3575 - val_loss: 0.3333
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4000 - val_loss: 0.3305
    Epoch 12/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5751 - val_loss: 1.1609
    Epoch 13/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4492 - val_loss: 0.3487
    Epoch 14/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3481 - val_loss: 0.3288
    Epoch 15/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3440 - val_loss: 0.3263
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3395 - val_loss: 0.3361
    Epoch 17/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3359 - val_loss: 0.3362
    Epoch 18/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3359 - val_loss: 0.3213
    Epoch 19/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3335 - val_loss: 0.3174
    Epoch 20/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3284 - val_loss: 0.3236
    Epoch 21/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3275 - val_loss: 0.3134
    Epoch 22/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3257 - val_loss: 0.3121
    Epoch 23/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3243 - val_loss: 0.3127
    Epoch 24/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3218 - val_loss: 0.3214
    Epoch 25/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3217 - val_loss: 0.3085
    Epoch 26/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3191 - val_loss: 0.3113
    Epoch 27/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3189 - val_loss: 0.3051
    Epoch 28/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3172 - val_loss: 0.3059
    Epoch 29/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3154 - val_loss: 0.3059
    Epoch 30/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3146 - val_loss: 0.3134
    Epoch 31/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3111 - val_loss: 0.3168
    Epoch 32/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3134 - val_loss: 0.3356
    Epoch 33/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3130 - val_loss: 0.3226
    Epoch 34/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3100 - val_loss: 0.3243
    Epoch 35/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3097 - val_loss: 0.3224
    Epoch 36/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3088 - val_loss: 0.3283
    Epoch 37/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3113 - val_loss: 0.3471
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.4386
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3676
    [CV]  learning_rate=0.015956195942385693, n_hidden=0, n_neurons=97, total=  10.0s
    [CV] learning_rate=0.015956195942385693, n_hidden=0, n_neurons=97 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 46us/sample - loss: 0.7714 - val_loss: 0.7641
    Epoch 2/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4561 - val_loss: 1.2963
    Epoch 3/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4122 - val_loss: 0.3778
    Epoch 4/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4151 - val_loss: 0.4568
    Epoch 5/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3952 - val_loss: 0.7350
    Epoch 6/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3759 - val_loss: 0.7615
    Epoch 7/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3707 - val_loss: 0.5305
    Epoch 8/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3696 - val_loss: 0.3510
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3634 - val_loss: 0.3410
    Epoch 10/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3602 - val_loss: 0.3868
    Epoch 11/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3531 - val_loss: 0.8745
    Epoch 12/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3574 - val_loss: 0.3859
    Epoch 13/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3542 - val_loss: 0.6109
    Epoch 14/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3512 - val_loss: 0.4101
    Epoch 15/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3496 - val_loss: 0.3618
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3458 - val_loss: 0.3277
    Epoch 17/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3454 - val_loss: 0.4292
    Epoch 18/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3431 - val_loss: 0.3737
    Epoch 19/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3434 - val_loss: 0.6402
    Epoch 20/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3444 - val_loss: 0.3269
    Epoch 21/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3388 - val_loss: 0.6092
    Epoch 22/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3431 - val_loss: 0.3750
    Epoch 23/100
    7740/7740 [==============================] - 0s 43us/sample - loss: 0.3417 - val_loss: 0.3615
    Epoch 24/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3409 - val_loss: 0.6085
    Epoch 25/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3331 - val_loss: 0.4557
    Epoch 26/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3308 - val_loss: 0.4441
    Epoch 27/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3303 - val_loss: 0.3698
    Epoch 28/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3282 - val_loss: 0.3496
    Epoch 29/100
    7740/7740 [==============================] - 0s 45us/sample - loss: 0.3314 - val_loss: 0.3630
    Epoch 30/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3258 - val_loss: 0.3160
    Epoch 31/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3255 - val_loss: 0.3887
    Epoch 32/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3229 - val_loss: 0.7654
    Epoch 33/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3269 - val_loss: 0.3289
    Epoch 34/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3192 - val_loss: 0.3337
    Epoch 35/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3304 - val_loss: 0.4950
    Epoch 36/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3192 - val_loss: 1.9769
    Epoch 37/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3214 - val_loss: 0.6508
    Epoch 38/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3190 - val_loss: 0.3250
    Epoch 39/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3191 - val_loss: 0.7641
    Epoch 40/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3161 - val_loss: 0.4083
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3752
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3483
    [CV]  learning_rate=0.015956195942385693, n_hidden=0, n_neurons=97, total=  11.3s
    [CV] learning_rate=0.015956195942385693, n_hidden=0, n_neurons=97 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 46us/sample - loss: 0.7449 - val_loss: 1.8796
    Epoch 2/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4798 - val_loss: 0.5772
    Epoch 3/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4285 - val_loss: 0.3831
    Epoch 4/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4161 - val_loss: 1.4244
    Epoch 5/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4231 - val_loss: 18.9345
    Epoch 6/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3970 - val_loss: 6.8569
    Epoch 7/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6343 - val_loss: 3.4721
    Epoch 8/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4484 - val_loss: 103.4562
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5382 - val_loss: 0.3930
    Epoch 10/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3965 - val_loss: 0.3660
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4080 - val_loss: 0.3647
    Epoch 12/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3822 - val_loss: 0.3534
    Epoch 13/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3735 - val_loss: 0.3537
    Epoch 14/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3672 - val_loss: 0.3453
    Epoch 15/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3628 - val_loss: 0.3444
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3766 - val_loss: 0.3401
    Epoch 17/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3711 - val_loss: 0.3460
    Epoch 18/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3623 - val_loss: 0.3325
    Epoch 19/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3570 - val_loss: 0.3644
    Epoch 20/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3536 - val_loss: 0.3517
    Epoch 21/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3496 - val_loss: 0.3682
    Epoch 22/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3599 - val_loss: 0.3256
    Epoch 23/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3469 - val_loss: 0.3334
    Epoch 24/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3439 - val_loss: 0.3432
    Epoch 25/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3469 - val_loss: 0.3219
    Epoch 26/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3416 - val_loss: 0.3368
    Epoch 27/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3394 - val_loss: 0.3201
    Epoch 28/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3370 - val_loss: 0.3207
    Epoch 29/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3346 - val_loss: 0.3202
    Epoch 30/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3315 - val_loss: 0.3141
    Epoch 31/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3335 - val_loss: 0.3171
    Epoch 32/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3306 - val_loss: 0.3172
    Epoch 33/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3389 - val_loss: 0.3142
    Epoch 34/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3332 - val_loss: 0.3638
    Epoch 35/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3315 - val_loss: 0.3186
    Epoch 36/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3256 - val_loss: 0.3102
    Epoch 37/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3233 - val_loss: 0.3238
    Epoch 38/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3291 - val_loss: 0.3299
    Epoch 39/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3339 - val_loss: 0.3433
    Epoch 40/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3250 - val_loss: 0.3063
    Epoch 41/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3228 - val_loss: 0.3480
    Epoch 42/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3186 - val_loss: 0.3669
    Epoch 43/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3251 - val_loss: 0.3058
    Epoch 44/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3247 - val_loss: 0.3364
    Epoch 45/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3180 - val_loss: 0.3104
    Epoch 46/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3232 - val_loss: 0.3273
    Epoch 47/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3166 - val_loss: 0.3027
    Epoch 48/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3136 - val_loss: 0.3203
    Epoch 49/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3131 - val_loss: 0.3446
    Epoch 50/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3147 - val_loss: 0.3076
    Epoch 51/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3226 - val_loss: 0.3362
    Epoch 52/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3121 - val_loss: 0.3097
    Epoch 53/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3116 - val_loss: 0.4066
    Epoch 54/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3176 - val_loss: 0.3047
    Epoch 55/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3096 - val_loss: 0.3028
    Epoch 56/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3083 - val_loss: 0.3614
    Epoch 57/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3074 - val_loss: 0.3032
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3190
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3051
    [CV]  learning_rate=0.015956195942385693, n_hidden=0, n_neurons=97, total=  15.3s
    [CV] learning_rate=0.0017196854145436866, n_hidden=2, n_neurons=77 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.5431 - val_loss: 2.1386
    Epoch 2/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.6880 - val_loss: 0.5965
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5931 - val_loss: 0.5344
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5405 - val_loss: 0.5002
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4996 - val_loss: 0.5025
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4679 - val_loss: 0.4393
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4435 - val_loss: 0.4388
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4256 - val_loss: 0.3981
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4115 - val_loss: 0.4108
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4016 - val_loss: 0.4239
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3936 - val_loss: 0.4047
    Epoch 12/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3870 - val_loss: 0.4046
    Epoch 13/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3808 - val_loss: 0.3774
    Epoch 14/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3757 - val_loss: 0.4053
    Epoch 15/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3706 - val_loss: 0.3574
    Epoch 16/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3669 - val_loss: 0.3605
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3628 - val_loss: 0.4119
    Epoch 18/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3604 - val_loss: 0.4028
    Epoch 19/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3572 - val_loss: 0.3772
    Epoch 20/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3547 - val_loss: 0.3834
    Epoch 21/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3522 - val_loss: 0.3464
    Epoch 22/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3494 - val_loss: 0.3959
    Epoch 23/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3477 - val_loss: 0.3378
    Epoch 24/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3456 - val_loss: 0.4059
    Epoch 25/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3446 - val_loss: 0.3477
    Epoch 26/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3420 - val_loss: 0.3920
    Epoch 27/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3410 - val_loss: 0.3325
    Epoch 28/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3389 - val_loss: 0.3782
    Epoch 29/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3373 - val_loss: 0.3309
    Epoch 30/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3366 - val_loss: 0.3642
    Epoch 31/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3350 - val_loss: 0.3590
    Epoch 32/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3333 - val_loss: 0.3305
    Epoch 33/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3319 - val_loss: 0.3564
    Epoch 34/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3305 - val_loss: 0.3445
    Epoch 35/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3300 - val_loss: 0.3723
    Epoch 36/100
    7740/7740 [==============================] - 0s 42us/sample - loss: 0.3287 - val_loss: 0.3593
    Epoch 37/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3278 - val_loss: 0.3276
    Epoch 38/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3272 - val_loss: 0.4318
    Epoch 39/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3264 - val_loss: 0.3511
    Epoch 40/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3252 - val_loss: 0.3221
    Epoch 41/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3240 - val_loss: 0.3791
    Epoch 42/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3236 - val_loss: 0.3305
    Epoch 43/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3225 - val_loss: 0.3361
    Epoch 44/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3210 - val_loss: 0.3781
    Epoch 45/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3205 - val_loss: 0.3291
    Epoch 46/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3197 - val_loss: 0.3419
    Epoch 47/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3189 - val_loss: 0.3624
    Epoch 48/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3179 - val_loss: 0.3237
    Epoch 49/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3168 - val_loss: 0.3165
    Epoch 50/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3160 - val_loss: 0.3233
    Epoch 51/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3148 - val_loss: 0.4150
    Epoch 52/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3152 - val_loss: 0.3100
    Epoch 53/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3138 - val_loss: 0.4137
    Epoch 54/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3140 - val_loss: 0.3866
    Epoch 55/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3125 - val_loss: 0.3258
    Epoch 56/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3112 - val_loss: 0.3279
    Epoch 57/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3109 - val_loss: 0.4006
    Epoch 58/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3111 - val_loss: 0.3077
    Epoch 59/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3094 - val_loss: 0.3585
    Epoch 60/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3097 - val_loss: 0.3600
    Epoch 61/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3089 - val_loss: 0.3933
    Epoch 62/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3080 - val_loss: 0.3050
    Epoch 63/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3066 - val_loss: 0.3039
    Epoch 64/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3057 - val_loss: 0.3046
    Epoch 65/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3053 - val_loss: 0.3040
    Epoch 66/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3047 - val_loss: 0.3039
    Epoch 67/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3036 - val_loss: 0.4107
    Epoch 68/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3036 - val_loss: 0.3624
    Epoch 69/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3031 - val_loss: 0.4021
    Epoch 70/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3026 - val_loss: 0.3481
    Epoch 71/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3021 - val_loss: 0.5111
    Epoch 72/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3034 - val_loss: 0.3270
    Epoch 73/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2998 - val_loss: 0.4186
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3326
    7740/7740 [==============================] - 0s 17us/sample - loss: 0.2985
    [CV]  learning_rate=0.0017196854145436866, n_hidden=2, n_neurons=77, total=  22.5s
    [CV] learning_rate=0.0017196854145436866, n_hidden=2, n_neurons=77 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.6301 - val_loss: 4.2039
    Epoch 2/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.6970 - val_loss: 0.6768
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.6090 - val_loss: 0.8804
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5529 - val_loss: 1.6374
    Epoch 5/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5127 - val_loss: 2.1626
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4821 - val_loss: 2.8482
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4595 - val_loss: 2.3233
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4420 - val_loss: 2.1937
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4277 - val_loss: 2.1441
    Epoch 10/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.4166 - val_loss: 1.2412
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4067 - val_loss: 1.0166
    Epoch 12/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3987 - val_loss: 0.7848
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.4227
    7740/7740 [==============================] - 0s 17us/sample - loss: 0.3932
    [CV]  learning_rate=0.0017196854145436866, n_hidden=2, n_neurons=77, total=   4.1s
    [CV] learning_rate=0.0017196854145436866, n_hidden=2, n_neurons=77 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.5410 - val_loss: 1.7610
    Epoch 2/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.6859 - val_loss: 0.6030
    Epoch 3/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.6079 - val_loss: 0.5475
    Epoch 4/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5578 - val_loss: 0.5058
    Epoch 5/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5189 - val_loss: 0.4733
    Epoch 6/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4871 - val_loss: 0.4425
    Epoch 7/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4617 - val_loss: 0.4225
    Epoch 8/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4422 - val_loss: 0.4064
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4270 - val_loss: 0.4141
    Epoch 10/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4156 - val_loss: 0.4001
    Epoch 11/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4056 - val_loss: 0.4111
    Epoch 12/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3978 - val_loss: 0.3833
    Epoch 13/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3928 - val_loss: 0.4107
    Epoch 14/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3881 - val_loss: 0.3695
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3824 - val_loss: 0.4339
    Epoch 16/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3784 - val_loss: 0.4015
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3751 - val_loss: 0.3944
    Epoch 18/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3715 - val_loss: 0.3888
    Epoch 19/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3681 - val_loss: 0.4058
    Epoch 20/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3654 - val_loss: 0.3471
    Epoch 21/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3638 - val_loss: 0.3469
    Epoch 22/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3600 - val_loss: 0.3725
    Epoch 23/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3581 - val_loss: 0.4448
    Epoch 24/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3571 - val_loss: 0.3868
    Epoch 25/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3549 - val_loss: 0.3871
    Epoch 26/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3532 - val_loss: 0.3699
    Epoch 27/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3510 - val_loss: 0.3456
    Epoch 28/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3492 - val_loss: 0.4108
    Epoch 29/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3481 - val_loss: 0.3753
    Epoch 30/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3461 - val_loss: 0.3960
    Epoch 31/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3445 - val_loss: 0.3945
    Epoch 32/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3434 - val_loss: 0.3682
    Epoch 33/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3419 - val_loss: 0.3298
    Epoch 34/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3411 - val_loss: 0.3609
    Epoch 35/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3398 - val_loss: 0.3399
    Epoch 36/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3385 - val_loss: 0.3297
    Epoch 37/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3368 - val_loss: 0.3665
    Epoch 38/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3364 - val_loss: 0.3196
    Epoch 39/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3353 - val_loss: 0.4043
    Epoch 40/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3344 - val_loss: 0.3602
    Epoch 41/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3329 - val_loss: 0.3659
    Epoch 42/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3321 - val_loss: 0.3848
    Epoch 43/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3310 - val_loss: 0.3145
    Epoch 44/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3296 - val_loss: 0.3836
    Epoch 45/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3294 - val_loss: 0.3203
    Epoch 46/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3272 - val_loss: 0.3455
    Epoch 47/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3271 - val_loss: 0.3557
    Epoch 48/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3262 - val_loss: 0.3176
    Epoch 49/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3252 - val_loss: 0.3626
    Epoch 50/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3246 - val_loss: 0.3258
    Epoch 51/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3235 - val_loss: 0.3191
    Epoch 52/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3231 - val_loss: 0.3160
    Epoch 53/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3216 - val_loss: 0.3293
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3243
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.3189
    [CV]  learning_rate=0.0017196854145436866, n_hidden=2, n_neurons=77, total=  16.3s
    [CV] learning_rate=0.003284284607688981, n_hidden=2, n_neurons=75 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.4842 - val_loss: 2.1657
    Epoch 2/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.5991 - val_loss: 1.9354
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5146 - val_loss: 0.6279
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4586 - val_loss: 0.5010
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4262 - val_loss: 0.3873
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4064 - val_loss: 0.3757
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3914 - val_loss: 0.3797
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3825 - val_loss: 0.3684
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3742 - val_loss: 0.3503
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3679 - val_loss: 0.3745
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3622 - val_loss: 0.3999
    Epoch 12/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3582 - val_loss: 0.3889
    Epoch 13/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3541 - val_loss: 0.3470
    Epoch 14/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3500 - val_loss: 0.4274
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3486 - val_loss: 0.3568
    Epoch 16/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3443 - val_loss: 0.4425
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3429 - val_loss: 0.3352
    Epoch 18/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3390 - val_loss: 0.3316
    Epoch 19/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3365 - val_loss: 0.4038
    Epoch 20/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3357 - val_loss: 0.3435
    Epoch 21/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3330 - val_loss: 0.3226
    Epoch 22/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3316 - val_loss: 0.3796
    Epoch 23/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3293 - val_loss: 0.3240
    Epoch 24/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3274 - val_loss: 0.3713
    Epoch 25/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3268 - val_loss: 0.3210
    Epoch 26/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3225 - val_loss: 0.3229
    Epoch 27/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3219 - val_loss: 0.3197
    Epoch 28/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3205 - val_loss: 0.3354
    Epoch 29/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3195 - val_loss: 0.3137
    Epoch 30/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3180 - val_loss: 0.3713
    Epoch 31/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3170 - val_loss: 0.3105
    Epoch 32/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3153 - val_loss: 0.3809
    Epoch 33/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3137 - val_loss: 0.3351
    Epoch 34/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3127 - val_loss: 0.3326
    Epoch 35/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3109 - val_loss: 0.3667
    Epoch 36/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3089 - val_loss: 0.3167
    Epoch 37/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3073 - val_loss: 0.3544
    Epoch 38/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3082 - val_loss: 0.3108
    Epoch 39/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3065 - val_loss: 0.3324
    Epoch 40/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3063 - val_loss: 0.3016
    Epoch 41/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3038 - val_loss: 0.3123
    Epoch 42/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3025 - val_loss: 0.3372
    Epoch 43/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3027 - val_loss: 0.3443
    Epoch 44/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3021 - val_loss: 0.3409
    Epoch 45/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3002 - val_loss: 0.3001
    Epoch 46/100
    7740/7740 [==============================] - 0s 43us/sample - loss: 0.2992 - val_loss: 0.3370
    Epoch 47/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2981 - val_loss: 0.3304
    Epoch 48/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2979 - val_loss: 0.3872
    Epoch 49/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2984 - val_loss: 0.3408
    Epoch 50/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2955 - val_loss: 0.3122
    Epoch 51/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2942 - val_loss: 0.3237
    Epoch 52/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2932 - val_loss: 0.3119
    Epoch 53/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2924 - val_loss: 0.3206
    Epoch 54/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2910 - val_loss: 0.4156
    Epoch 55/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2920 - val_loss: 0.3537
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3294
    7740/7740 [==============================] - 0s 17us/sample - loss: 0.2875
    [CV]  learning_rate=0.003284284607688981, n_hidden=2, n_neurons=75, total=  17.3s
    [CV] learning_rate=0.003284284607688981, n_hidden=2, n_neurons=75 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.0847 - val_loss: 5.8122
    Epoch 2/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.6135 - val_loss: 0.7386
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5218 - val_loss: 0.6025
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4632 - val_loss: 0.7112
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4265 - val_loss: 0.5877
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4039 - val_loss: 0.4176
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3898 - val_loss: 0.3684
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3788 - val_loss: 0.4108
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3707 - val_loss: 0.4843
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3641 - val_loss: 0.6541
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3576 - val_loss: 0.7952
    Epoch 12/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3545 - val_loss: 0.8378
    Epoch 13/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3505 - val_loss: 0.8978
    Epoch 14/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3467 - val_loss: 0.8281
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3450 - val_loss: 0.9674
    Epoch 16/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3422 - val_loss: 0.9459
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3400 - val_loss: 0.8946
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3573
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.3355
    [CV]  learning_rate=0.003284284607688981, n_hidden=2, n_neurons=75, total=   5.7s
    [CV] learning_rate=0.003284284607688981, n_hidden=2, n_neurons=75 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.3023 - val_loss: 2.6927
    Epoch 2/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5547 - val_loss: 0.5447
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4631 - val_loss: 0.4596
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4296 - val_loss: 0.4099
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4112 - val_loss: 0.4182
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3992 - val_loss: 0.4050
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3907 - val_loss: 0.3833
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3824 - val_loss: 0.4306
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3773 - val_loss: 0.3806
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3720 - val_loss: 0.3851
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3677 - val_loss: 0.3504
    Epoch 12/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3632 - val_loss: 0.3865
    Epoch 13/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3596 - val_loss: 0.4215
    Epoch 14/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3577 - val_loss: 0.3906
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3538 - val_loss: 0.4588
    Epoch 16/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3525 - val_loss: 0.3350
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3488 - val_loss: 0.4020
    Epoch 18/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3459 - val_loss: 0.4451
    Epoch 19/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3445 - val_loss: 0.4082
    Epoch 20/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3429 - val_loss: 0.3690
    Epoch 21/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3413 - val_loss: 0.3326
    Epoch 22/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3389 - val_loss: 0.3475
    Epoch 23/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3376 - val_loss: 0.3381
    Epoch 24/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3352 - val_loss: 0.3333
    Epoch 25/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3340 - val_loss: 0.4147
    Epoch 26/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3325 - val_loss: 0.4207
    Epoch 27/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3319 - val_loss: 0.3328
    Epoch 28/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3302 - val_loss: 0.3656
    Epoch 29/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3288 - val_loss: 0.3180
    Epoch 30/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3266 - val_loss: 0.3994
    Epoch 31/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3269 - val_loss: 0.3344
    Epoch 32/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3252 - val_loss: 0.3281
    Epoch 33/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3226 - val_loss: 0.3339
    Epoch 34/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3233 - val_loss: 0.3283
    Epoch 35/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3208 - val_loss: 0.3834
    Epoch 36/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3209 - val_loss: 0.3260
    Epoch 37/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3187 - val_loss: 0.3149
    Epoch 38/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3175 - val_loss: 0.3136
    Epoch 39/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3159 - val_loss: 0.4119
    Epoch 40/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3158 - val_loss: 0.3252
    Epoch 41/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3144 - val_loss: 0.3174
    Epoch 42/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3129 - val_loss: 0.3911
    Epoch 43/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3131 - val_loss: 0.3167
    Epoch 44/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3117 - val_loss: 0.4159
    Epoch 45/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3113 - val_loss: 0.3414
    Epoch 46/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3078 - val_loss: 0.3597
    Epoch 47/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3089 - val_loss: 0.3062
    Epoch 48/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3074 - val_loss: 0.3133
    Epoch 49/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3052 - val_loss: 0.3587
    Epoch 50/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3060 - val_loss: 0.3166
    Epoch 51/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3038 - val_loss: 0.3144
    Epoch 52/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3038 - val_loss: 0.3803
    Epoch 53/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3024 - val_loss: 0.3023
    Epoch 54/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3012 - val_loss: 0.3206
    Epoch 55/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3006 - val_loss: 0.3453
    Epoch 56/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3008 - val_loss: 0.3081
    Epoch 57/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2989 - val_loss: 0.3795
    Epoch 58/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2982 - val_loss: 0.3208
    Epoch 59/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2973 - val_loss: 0.3284
    Epoch 60/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2957 - val_loss: 0.3058
    Epoch 61/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2966 - val_loss: 0.3155
    Epoch 62/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2947 - val_loss: 0.3156
    Epoch 63/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2948 - val_loss: 0.3417
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3149
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.2935
    [CV]  learning_rate=0.003284284607688981, n_hidden=2, n_neurons=75, total=  19.2s
    [CV] learning_rate=0.004038022680351922, n_hidden=2, n_neurons=69 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.1144 - val_loss: 1.6181
    Epoch 2/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5747 - val_loss: 1.2753
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4971 - val_loss: 0.5712
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4514 - val_loss: 0.5843
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4202 - val_loss: 0.3850
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4007 - val_loss: 0.3774
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3869 - val_loss: 0.4252
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3780 - val_loss: 0.3768
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3698 - val_loss: 0.3744
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3627 - val_loss: 0.3604
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3577 - val_loss: 0.4952
    Epoch 12/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3563 - val_loss: 0.4150
    Epoch 13/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3503 - val_loss: 0.3910
    Epoch 14/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3469 - val_loss: 0.3373
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3428 - val_loss: 0.3744
    Epoch 16/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3414 - val_loss: 0.3325
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3371 - val_loss: 0.4917
    Epoch 18/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3410 - val_loss: 0.6252
    Epoch 19/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3394 - val_loss: 1.0122
    Epoch 20/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3407 - val_loss: 0.9887
    Epoch 21/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3384 - val_loss: 0.7173
    Epoch 22/100
    7740/7740 [==============================] - 0s 43us/sample - loss: 0.3325 - val_loss: 0.3328
    Epoch 23/100
    7740/7740 [==============================] - 0s 45us/sample - loss: 0.3257 - val_loss: 0.3620
    Epoch 24/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3255 - val_loss: 0.3204
    Epoch 25/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3224 - val_loss: 0.3355
    Epoch 26/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3205 - val_loss: 0.3574
    Epoch 27/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3197 - val_loss: 0.3340
    Epoch 28/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3172 - val_loss: 0.3134
    Epoch 29/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3155 - val_loss: 0.3252
    Epoch 30/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3151 - val_loss: 0.3228
    Epoch 31/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3131 - val_loss: 0.3122
    Epoch 32/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3109 - val_loss: 0.3072
    Epoch 33/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3095 - val_loss: 0.4399
    Epoch 34/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3111 - val_loss: 0.5012
    Epoch 35/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3116 - val_loss: 0.4788
    Epoch 36/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3081 - val_loss: 0.3058
    Epoch 37/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3061 - val_loss: 0.3707
    Epoch 38/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3038 - val_loss: 0.3291
    Epoch 39/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3021 - val_loss: 0.3020
    Epoch 40/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3014 - val_loss: 0.3682
    Epoch 41/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3014 - val_loss: 0.3114
    Epoch 42/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2999 - val_loss: 0.3104
    Epoch 43/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2986 - val_loss: 0.3037
    Epoch 44/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2966 - val_loss: 0.3074
    Epoch 45/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2953 - val_loss: 0.3298
    Epoch 46/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2956 - val_loss: 0.4312
    Epoch 47/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2953 - val_loss: 0.4607
    Epoch 48/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2959 - val_loss: 0.3917
    Epoch 49/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2955 - val_loss: 0.3009
    Epoch 50/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2912 - val_loss: 0.3113
    Epoch 51/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2909 - val_loss: 0.3125
    Epoch 52/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2903 - val_loss: 0.3306
    Epoch 53/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2884 - val_loss: 0.2966
    Epoch 54/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2876 - val_loss: 0.3536
    Epoch 55/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2877 - val_loss: 0.4683
    Epoch 56/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2902 - val_loss: 0.4054
    Epoch 57/100
    7740/7740 [==============================] - 0s 42us/sample - loss: 0.2866 - val_loss: 0.3190
    Epoch 58/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2854 - val_loss: 0.3752
    Epoch 59/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2839 - val_loss: 0.2895
    Epoch 60/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2828 - val_loss: 0.3034
    Epoch 61/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2829 - val_loss: 0.3833
    Epoch 62/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2839 - val_loss: 0.2852
    Epoch 63/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2806 - val_loss: 0.2916
    Epoch 64/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2787 - val_loss: 0.2897
    Epoch 65/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2794 - val_loss: 0.2838
    Epoch 66/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2785 - val_loss: 0.3772
    Epoch 67/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2787 - val_loss: 0.3277
    Epoch 68/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2771 - val_loss: 0.5036
    Epoch 69/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2791 - val_loss: 0.3969
    Epoch 70/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2765 - val_loss: 0.6491
    Epoch 71/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2857 - val_loss: 0.6069
    Epoch 72/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2822 - val_loss: 0.8506
    Epoch 73/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2840 - val_loss: 0.4954
    Epoch 74/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2785 - val_loss: 0.4551
    Epoch 75/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2754 - val_loss: 0.2963
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3199
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.2753
    [CV]  learning_rate=0.004038022680351922, n_hidden=2, n_neurons=69, total=  23.2s
    [CV] learning_rate=0.004038022680351922, n_hidden=2, n_neurons=69 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 55us/sample - loss: 1.1371 - val_loss: 1.3452
    Epoch 2/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5859 - val_loss: 0.5920
    Epoch 3/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5003 - val_loss: 0.4443
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4476 - val_loss: 0.4072
    Epoch 5/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4162 - val_loss: 0.3839
    Epoch 6/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3974 - val_loss: 0.3671
    Epoch 7/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3837 - val_loss: 0.3693
    Epoch 8/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3740 - val_loss: 0.3850
    Epoch 9/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3688 - val_loss: 0.4103
    Epoch 10/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3634 - val_loss: 0.4479
    Epoch 11/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3587 - val_loss: 0.5044
    Epoch 12/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3554 - val_loss: 0.5236
    Epoch 13/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3508 - val_loss: 0.5970
    Epoch 14/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3474 - val_loss: 0.6589
    Epoch 15/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3454 - val_loss: 0.6735
    Epoch 16/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3421 - val_loss: 0.7284
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3575
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.3373
    [CV]  learning_rate=0.004038022680351922, n_hidden=2, n_neurons=69, total=   5.2s
    [CV] learning_rate=0.004038022680351922, n_hidden=2, n_neurons=69 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 56us/sample - loss: 1.2970 - val_loss: 7.5366
    Epoch 2/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.6151 - val_loss: 2.4204
    Epoch 3/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5010 - val_loss: 0.4373
    Epoch 4/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4434 - val_loss: 0.4774
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4185 - val_loss: 0.4510
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4012 - val_loss: 0.4292
    Epoch 7/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3888 - val_loss: 0.3930
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3802 - val_loss: 0.4631
    Epoch 9/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3740 - val_loss: 0.3844
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3680 - val_loss: 0.4246
    Epoch 11/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3648 - val_loss: 0.4520
    Epoch 12/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3609 - val_loss: 0.4205
    Epoch 13/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3570 - val_loss: 0.3746
    Epoch 14/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3529 - val_loss: 0.3670
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3511 - val_loss: 0.3522
    Epoch 16/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3471 - val_loss: 0.4065
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3453 - val_loss: 0.3991
    Epoch 18/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3426 - val_loss: 0.3844
    Epoch 19/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3402 - val_loss: 0.3834
    Epoch 20/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3375 - val_loss: 0.4403
    Epoch 21/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3362 - val_loss: 0.3458
    Epoch 22/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3335 - val_loss: 0.3672
    Epoch 23/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3317 - val_loss: 0.3860
    Epoch 24/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3304 - val_loss: 0.3250
    Epoch 25/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3278 - val_loss: 0.3844
    Epoch 26/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3265 - val_loss: 0.4097
    Epoch 27/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3250 - val_loss: 0.3139
    Epoch 28/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3238 - val_loss: 0.3452
    Epoch 29/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3215 - val_loss: 0.3300
    Epoch 30/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3198 - val_loss: 0.3281
    Epoch 31/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3173 - val_loss: 0.3994
    Epoch 32/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3172 - val_loss: 0.3340
    Epoch 33/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3153 - val_loss: 0.3991
    Epoch 34/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3152 - val_loss: 0.3153
    Epoch 35/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3128 - val_loss: 0.3425
    Epoch 36/100
    7740/7740 [==============================] - 0s 48us/sample - loss: 0.3115 - val_loss: 0.3047
    Epoch 37/100
    7740/7740 [==============================] - 0s 44us/sample - loss: 0.3098 - val_loss: 0.3154
    Epoch 38/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3085 - val_loss: 0.3139
    Epoch 39/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3074 - val_loss: 0.3774
    Epoch 40/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3071 - val_loss: 0.3249
    Epoch 41/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3071 - val_loss: 0.3420
    Epoch 42/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3026 - val_loss: 0.3788
    Epoch 43/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3027 - val_loss: 0.3213
    Epoch 44/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3014 - val_loss: 0.3695
    Epoch 45/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3001 - val_loss: 0.2987
    Epoch 46/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2987 - val_loss: 0.3171
    Epoch 47/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2981 - val_loss: 0.3631
    Epoch 48/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2975 - val_loss: 0.2914
    Epoch 49/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2959 - val_loss: 0.3695
    Epoch 50/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.2942 - val_loss: 0.2942
    Epoch 51/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2925 - val_loss: 0.3925
    Epoch 52/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2939 - val_loss: 0.2917
    Epoch 53/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2916 - val_loss: 0.3588
    Epoch 54/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.2899 - val_loss: 0.3232
    Epoch 55/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2914 - val_loss: 0.2941
    Epoch 56/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2878 - val_loss: 0.2984
    Epoch 57/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2871 - val_loss: 0.3576
    Epoch 58/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2862 - val_loss: 0.3397
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3052
    7740/7740 [==============================] - 0s 17us/sample - loss: 0.2829
    [CV]  learning_rate=0.004038022680351922, n_hidden=2, n_neurons=69, total=  18.3s
    [CV] learning_rate=0.0006477779307312751, n_hidden=1, n_neurons=59 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 51us/sample - loss: 2.7840 - val_loss: 3.1844
    Epoch 2/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 1.2596 - val_loss: 0.9812
    Epoch 3/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.8758 - val_loss: 0.7441
    Epoch 4/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.7312 - val_loss: 0.6454
    Epoch 5/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.6688 - val_loss: 0.6013
    Epoch 6/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6341 - val_loss: 0.5979
    Epoch 7/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.6103 - val_loss: 0.5533
    Epoch 8/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5899 - val_loss: 0.5331
    Epoch 9/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5719 - val_loss: 0.5214
    Epoch 10/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5560 - val_loss: 0.5139
    Epoch 11/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5418 - val_loss: 0.4965
    Epoch 12/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5286 - val_loss: 0.4943
    Epoch 13/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5171 - val_loss: 0.4726
    Epoch 14/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5061 - val_loss: 0.4608
    Epoch 15/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4962 - val_loss: 0.4600
    Epoch 16/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4876 - val_loss: 0.4459
    Epoch 17/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4794 - val_loss: 0.4467
    Epoch 18/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4722 - val_loss: 0.4322
    Epoch 19/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4656 - val_loss: 0.4305
    Epoch 20/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4597 - val_loss: 0.4219
    Epoch 21/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4540 - val_loss: 0.4187
    Epoch 22/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4492 - val_loss: 0.4126
    Epoch 23/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4442 - val_loss: 0.4112
    Epoch 24/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4402 - val_loss: 0.4057
    Epoch 25/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4363 - val_loss: 0.4048
    Epoch 26/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4327 - val_loss: 0.4011
    Epoch 27/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4291 - val_loss: 0.3973
    Epoch 28/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4259 - val_loss: 0.3977
    Epoch 29/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4230 - val_loss: 0.3935
    Epoch 30/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4201 - val_loss: 0.3995
    Epoch 31/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4176 - val_loss: 0.3938
    Epoch 32/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4149 - val_loss: 0.3924
    Epoch 33/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4125 - val_loss: 0.3842
    Epoch 34/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4102 - val_loss: 0.3922
    Epoch 35/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4080 - val_loss: 0.3810
    Epoch 36/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4059 - val_loss: 0.3883
    Epoch 37/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4039 - val_loss: 0.3848
    Epoch 38/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4018 - val_loss: 0.3784
    Epoch 39/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4001 - val_loss: 0.3841
    Epoch 40/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3984 - val_loss: 0.3793
    Epoch 41/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3966 - val_loss: 0.3837
    Epoch 42/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3951 - val_loss: 0.3736
    Epoch 43/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3934 - val_loss: 0.3825
    Epoch 44/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3921 - val_loss: 0.3693
    Epoch 45/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3905 - val_loss: 0.3694
    Epoch 46/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3891 - val_loss: 0.3673
    Epoch 47/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3875 - val_loss: 0.3753
    Epoch 48/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3862 - val_loss: 0.3868
    Epoch 49/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3850 - val_loss: 0.3735
    Epoch 50/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3837 - val_loss: 0.3745
    Epoch 51/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3825 - val_loss: 0.3742
    Epoch 52/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3814 - val_loss: 0.3672
    Epoch 53/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3800 - val_loss: 0.3624
    Epoch 54/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3787 - val_loss: 0.3769
    Epoch 55/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3778 - val_loss: 0.3609
    Epoch 56/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3766 - val_loss: 0.3719
    Epoch 57/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3757 - val_loss: 0.3732
    Epoch 58/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3748 - val_loss: 0.3588
    Epoch 59/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3736 - val_loss: 0.3681
    Epoch 60/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3726 - val_loss: 0.3557
    Epoch 61/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3716 - val_loss: 0.3600
    Epoch 62/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3706 - val_loss: 0.3524
    Epoch 63/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3699 - val_loss: 0.3550
    Epoch 64/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3689 - val_loss: 0.3609
    Epoch 65/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3681 - val_loss: 0.3504
    Epoch 66/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3670 - val_loss: 0.3500
    Epoch 67/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3662 - val_loss: 0.3657
    Epoch 68/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3655 - val_loss: 0.3561
    Epoch 69/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3647 - val_loss: 0.3559
    Epoch 70/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3638 - val_loss: 0.3566
    Epoch 71/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3631 - val_loss: 0.3482
    Epoch 72/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3623 - val_loss: 0.3550
    Epoch 73/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3617 - val_loss: 0.3494
    Epoch 74/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3609 - val_loss: 0.3559
    Epoch 75/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3602 - val_loss: 0.3543
    Epoch 76/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3595 - val_loss: 0.3439
    Epoch 77/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3588 - val_loss: 0.3552
    Epoch 78/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3582 - val_loss: 0.3423
    Epoch 79/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3576 - val_loss: 0.3480
    Epoch 80/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3569 - val_loss: 0.3429
    Epoch 81/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3562 - val_loss: 0.3474
    Epoch 82/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3555 - val_loss: 0.3482
    Epoch 83/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3551 - val_loss: 0.3444
    Epoch 84/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3545 - val_loss: 0.3417
    Epoch 85/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3538 - val_loss: 0.3400
    Epoch 86/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3533 - val_loss: 0.3438
    Epoch 87/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3526 - val_loss: 0.3545
    Epoch 88/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3523 - val_loss: 0.3430
    Epoch 89/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3515 - val_loss: 0.3515
    Epoch 90/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3511 - val_loss: 0.3432
    Epoch 91/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3505 - val_loss: 0.3404
    Epoch 92/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3499 - val_loss: 0.3428
    Epoch 93/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3496 - val_loss: 0.3470
    Epoch 94/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3490 - val_loss: 0.3530
    Epoch 95/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3485 - val_loss: 0.3673
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3662
    7740/7740 [==============================] - 0s 15us/sample - loss: 0.3478
    [CV]  learning_rate=0.0006477779307312751, n_hidden=1, n_neurons=59, total=  27.3s
    [CV] learning_rate=0.0006477779307312751, n_hidden=1, n_neurons=59 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 51us/sample - loss: 2.9243 - val_loss: 18.8442
    Epoch 2/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 1.2074 - val_loss: 16.0418
    Epoch 3/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.8573 - val_loss: 11.0685
    Epoch 4/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.7260 - val_loss: 7.3959
    Epoch 5/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6690 - val_loss: 5.0236
    Epoch 6/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6350 - val_loss: 3.4955
    Epoch 7/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6096 - val_loss: 2.3247
    Epoch 8/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5878 - val_loss: 1.5633
    Epoch 9/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5687 - val_loss: 1.0776
    Epoch 10/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5517 - val_loss: 0.7538
    Epoch 11/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5365 - val_loss: 0.5741
    Epoch 12/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5228 - val_loss: 0.5018
    Epoch 13/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5106 - val_loss: 0.4949
    Epoch 14/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4997 - val_loss: 0.5297
    Epoch 15/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4899 - val_loss: 0.5825
    Epoch 16/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4812 - val_loss: 0.6560
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.4736 - val_loss: 0.7104
    Epoch 18/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4666 - val_loss: 0.7667
    Epoch 19/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4604 - val_loss: 0.7889
    Epoch 20/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4546 - val_loss: 0.8336
    Epoch 21/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4495 - val_loss: 0.8446
    Epoch 22/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4448 - val_loss: 0.8488
    Epoch 23/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4403 - val_loss: 0.8735
    3870/3870 [==============================] - 0s 14us/sample - loss: 0.4598
    7740/7740 [==============================] - 0s 15us/sample - loss: 0.4382
    [CV]  learning_rate=0.0006477779307312751, n_hidden=1, n_neurons=59, total=   6.9s
    [CV] learning_rate=0.0006477779307312751, n_hidden=1, n_neurons=59 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 51us/sample - loss: 2.6866 - val_loss: 8.0515
    Epoch 2/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 1.0829 - val_loss: 1.5668
    Epoch 3/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.7425 - val_loss: 0.6838
    Epoch 4/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6664 - val_loss: 0.6391
    Epoch 5/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6360 - val_loss: 0.6036
    Epoch 6/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6139 - val_loss: 0.5849
    Epoch 7/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5957 - val_loss: 0.5632
    Epoch 8/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5793 - val_loss: 0.5505
    Epoch 9/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5648 - val_loss: 0.5358
    Epoch 10/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5513 - val_loss: 0.5225
    Epoch 11/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5389 - val_loss: 0.5139
    Epoch 12/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5273 - val_loss: 0.4968
    Epoch 13/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5164 - val_loss: 0.4871
    Epoch 14/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.5063 - val_loss: 0.4761
    Epoch 15/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4972 - val_loss: 0.4666
    Epoch 16/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4882 - val_loss: 0.4606
    Epoch 17/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4801 - val_loss: 0.4513
    Epoch 18/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4728 - val_loss: 0.4440
    Epoch 19/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4654 - val_loss: 0.4390
    Epoch 20/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4594 - val_loss: 0.4303
    Epoch 21/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4531 - val_loss: 0.4275
    Epoch 22/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4476 - val_loss: 0.4231
    Epoch 23/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4425 - val_loss: 0.4187
    Epoch 24/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4376 - val_loss: 0.4136
    Epoch 25/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4333 - val_loss: 0.4054
    Epoch 26/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4293 - val_loss: 0.4118
    Epoch 27/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4256 - val_loss: 0.4052
    Epoch 28/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4221 - val_loss: 0.3981
    Epoch 29/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4190 - val_loss: 0.3979
    Epoch 30/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4159 - val_loss: 0.3940
    Epoch 31/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4132 - val_loss: 0.3969
    Epoch 32/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4106 - val_loss: 0.3897
    Epoch 33/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4080 - val_loss: 0.3888
    Epoch 34/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4058 - val_loss: 0.3832
    Epoch 35/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4036 - val_loss: 0.3949
    Epoch 36/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4017 - val_loss: 0.3838
    Epoch 37/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3996 - val_loss: 0.3866
    Epoch 38/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3978 - val_loss: 0.3872
    Epoch 39/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3961 - val_loss: 0.3955
    Epoch 40/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3944 - val_loss: 0.3855
    Epoch 41/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3929 - val_loss: 0.3826
    Epoch 42/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3914 - val_loss: 0.3714
    Epoch 43/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3896 - val_loss: 0.3914
    Epoch 44/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3885 - val_loss: 0.3794
    Epoch 45/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3870 - val_loss: 0.3651
    Epoch 46/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3854 - val_loss: 0.3952
    Epoch 47/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3843 - val_loss: 0.3603
    Epoch 48/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3831 - val_loss: 0.3862
    Epoch 49/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3819 - val_loss: 0.3959
    Epoch 50/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3807 - val_loss: 0.3726
    Epoch 51/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3797 - val_loss: 0.3636
    Epoch 52/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3783 - val_loss: 0.3939
    Epoch 53/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3776 - val_loss: 0.3632
    Epoch 54/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3760 - val_loss: 0.3598
    Epoch 55/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3753 - val_loss: 0.3850
    Epoch 56/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3745 - val_loss: 0.3665
    Epoch 57/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3738 - val_loss: 0.3574
    Epoch 58/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3724 - val_loss: 0.3859
    Epoch 59/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3720 - val_loss: 0.3780
    Epoch 60/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3710 - val_loss: 0.3745
    Epoch 61/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3703 - val_loss: 0.3562
    Epoch 62/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3694 - val_loss: 0.3705
    Epoch 63/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3685 - val_loss: 0.3532
    Epoch 64/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3675 - val_loss: 0.3531
    Epoch 65/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3670 - val_loss: 0.3522
    Epoch 66/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3660 - val_loss: 0.3824
    Epoch 67/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3654 - val_loss: 0.3472
    Epoch 68/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3649 - val_loss: 0.3528
    Epoch 69/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3639 - val_loss: 0.3488
    Epoch 70/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3634 - val_loss: 0.3521
    Epoch 71/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3628 - val_loss: 0.3633
    Epoch 72/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3621 - val_loss: 0.3724
    Epoch 73/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3615 - val_loss: 0.3504
    Epoch 74/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3606 - val_loss: 0.3757
    Epoch 75/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3601 - val_loss: 0.3428
    Epoch 76/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3597 - val_loss: 0.3600
    Epoch 77/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3590 - val_loss: 0.3468
    Epoch 78/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3584 - val_loss: 0.3391
    Epoch 79/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3576 - val_loss: 0.3450
    Epoch 80/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3571 - val_loss: 0.3658
    Epoch 81/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3566 - val_loss: 0.3612
    Epoch 82/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3561 - val_loss: 0.3525
    Epoch 83/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3554 - val_loss: 0.3548
    Epoch 84/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3550 - val_loss: 0.3449
    Epoch 85/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3542 - val_loss: 0.3533
    Epoch 86/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3542 - val_loss: 0.3527
    Epoch 87/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3534 - val_loss: 0.3655
    Epoch 88/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3530 - val_loss: 0.3569
    3870/3870 [==============================] - 0s 14us/sample - loss: 0.3494
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3521
    [CV]  learning_rate=0.0006477779307312751, n_hidden=1, n_neurons=59, total=  24.8s
    [CV] learning_rate=0.0032123503761513073, n_hidden=0, n_neurons=87 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 46us/sample - loss: 1.3665 - val_loss: 30.0074
    Epoch 2/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 1.0455 - val_loss: 22.4981
    Epoch 3/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.8411 - val_loss: 0.5801
    Epoch 4/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5291 - val_loss: 0.5369
    Epoch 5/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4956 - val_loss: 0.5124
    Epoch 6/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4722 - val_loss: 0.4677
    Epoch 7/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4550 - val_loss: 0.4384
    Epoch 8/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4421 - val_loss: 0.4157
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4312 - val_loss: 0.3990
    Epoch 10/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4227 - val_loss: 0.3904
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4157 - val_loss: 0.3844
    Epoch 12/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4095 - val_loss: 0.3846
    Epoch 13/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4053 - val_loss: 0.3783
    Epoch 14/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4016 - val_loss: 0.3811
    Epoch 15/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3976 - val_loss: 0.3855
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3946 - val_loss: 0.3793
    Epoch 17/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3917 - val_loss: 0.3790
    Epoch 18/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3892 - val_loss: 0.3795
    Epoch 19/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3868 - val_loss: 0.3751
    Epoch 20/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3844 - val_loss: 0.3905
    Epoch 21/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3830 - val_loss: 0.3758
    Epoch 22/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3806 - val_loss: 0.3810
    Epoch 23/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3790 - val_loss: 0.3810
    Epoch 24/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3773 - val_loss: 0.3796
    Epoch 25/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3763 - val_loss: 0.3838
    Epoch 26/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3739 - val_loss: 0.3877
    Epoch 27/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3731 - val_loss: 0.3776
    Epoch 28/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3716 - val_loss: 0.3778
    Epoch 29/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3702 - val_loss: 0.3812
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3857
    7740/7740 [==============================] - 0s 13us/sample - loss: 0.3677
    [CV]  learning_rate=0.0032123503761513073, n_hidden=0, n_neurons=87, total=   7.9s
    [CV] learning_rate=0.0032123503761513073, n_hidden=0, n_neurons=87 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 46us/sample - loss: 1.3253 - val_loss: 1.7528
    Epoch 2/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6961 - val_loss: 0.6533
    Epoch 3/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6193 - val_loss: 1.3104
    Epoch 4/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5629 - val_loss: 2.2476
    Epoch 5/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5220 - val_loss: 2.6183
    Epoch 6/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4918 - val_loss: 2.7712
    Epoch 7/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4692 - val_loss: 2.4734
    Epoch 8/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4537 - val_loss: 2.0569
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4415 - val_loss: 1.6704
    Epoch 10/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4329 - val_loss: 1.3330
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4249 - val_loss: 0.9791
    Epoch 12/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4199 - val_loss: 0.8599
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.4479
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.4163
    [CV]  learning_rate=0.0032123503761513073, n_hidden=0, n_neurons=87, total=   3.5s
    [CV] learning_rate=0.0032123503761513073, n_hidden=0, n_neurons=87 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 47us/sample - loss: 1.5445 - val_loss: 15.2564
    Epoch 2/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.9030 - val_loss: 5.7446
    Epoch 3/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6692 - val_loss: 0.6443
    Epoch 4/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5637 - val_loss: 0.5236
    Epoch 5/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5269 - val_loss: 0.4840
    Epoch 6/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4986 - val_loss: 0.4984
    Epoch 7/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4779 - val_loss: 0.4490
    Epoch 8/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4620 - val_loss: 0.4321
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4504 - val_loss: 0.4292
    Epoch 10/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4414 - val_loss: 0.4454
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4341 - val_loss: 0.4018
    Epoch 12/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4281 - val_loss: 0.4089
    Epoch 13/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4226 - val_loss: 0.4120
    Epoch 14/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4179 - val_loss: 0.4455
    Epoch 15/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4138 - val_loss: 0.4541
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4108 - val_loss: 0.4184
    Epoch 17/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4075 - val_loss: 0.4369
    Epoch 18/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4044 - val_loss: 0.3751
    Epoch 19/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4026 - val_loss: 0.3771
    Epoch 20/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3994 - val_loss: 0.4666
    Epoch 21/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3988 - val_loss: 0.4131
    Epoch 22/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3952 - val_loss: 0.3811
    Epoch 23/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3930 - val_loss: 0.3736
    Epoch 24/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3915 - val_loss: 0.3913
    Epoch 25/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3897 - val_loss: 0.3709
    Epoch 26/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3880 - val_loss: 0.3942
    Epoch 27/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3863 - val_loss: 0.3592
    Epoch 28/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3848 - val_loss: 0.3652
    Epoch 29/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3830 - val_loss: 0.3596
    Epoch 30/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3815 - val_loss: 0.4866
    Epoch 31/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3811 - val_loss: 0.3524
    Epoch 32/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3783 - val_loss: 0.5189
    Epoch 33/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3787 - val_loss: 0.3859
    Epoch 34/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3761 - val_loss: 0.3516
    Epoch 35/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3747 - val_loss: 0.5188
    Epoch 36/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3748 - val_loss: 0.3542
    Epoch 37/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3734 - val_loss: 0.5031
    Epoch 38/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3737 - val_loss: 0.3494
    Epoch 39/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3713 - val_loss: 0.3880
    Epoch 40/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3697 - val_loss: 0.3591
    Epoch 41/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3686 - val_loss: 0.4488
    Epoch 42/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3683 - val_loss: 0.3543
    Epoch 43/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3671 - val_loss: 0.3664
    Epoch 44/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3662 - val_loss: 0.3420
    Epoch 45/100
    7740/7740 [==============================] - 0s 32us/sample - loss: 0.3657 - val_loss: 0.3827
    Epoch 46/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3648 - val_loss: 0.3587
    Epoch 47/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3640 - val_loss: 0.3558
    Epoch 48/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3629 - val_loss: 0.5029
    Epoch 49/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3633 - val_loss: 0.3404
    Epoch 50/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3614 - val_loss: 0.5297
    Epoch 51/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3627 - val_loss: 0.3581
    Epoch 52/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3605 - val_loss: 0.4724
    Epoch 53/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3612 - val_loss: 0.3414
    Epoch 54/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3593 - val_loss: 0.3880
    Epoch 55/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3583 - val_loss: 0.3962
    Epoch 56/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3576 - val_loss: 0.4286
    Epoch 57/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3569 - val_loss: 0.3945
    Epoch 58/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3561 - val_loss: 0.3448
    Epoch 59/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3563 - val_loss: 0.4321
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3558
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3563
    [CV]  learning_rate=0.0032123503761513073, n_hidden=0, n_neurons=87, total=  15.7s
    [CV] learning_rate=0.004576909498448105, n_hidden=2, n_neurons=87 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 58us/sample - loss: 1.0841 - val_loss: 1.6728
    Epoch 2/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.5127 - val_loss: 3.2044
    Epoch 3/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.4594 - val_loss: 0.4278
    Epoch 4/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.4003 - val_loss: 0.4229
    Epoch 5/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3793 - val_loss: 0.4236
    Epoch 6/100
    7740/7740 [==============================] - 0s 42us/sample - loss: 0.3653 - val_loss: 0.4169
    Epoch 7/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3580 - val_loss: 0.4311
    Epoch 8/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3524 - val_loss: 0.3587
    Epoch 9/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3437 - val_loss: 0.4140
    Epoch 10/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3405 - val_loss: 0.3898
    Epoch 11/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3361 - val_loss: 0.3298
    Epoch 12/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3320 - val_loss: 0.3523
    Epoch 13/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3285 - val_loss: 0.3785
    Epoch 14/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3275 - val_loss: 0.3206
    Epoch 15/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3239 - val_loss: 0.3414
    Epoch 16/100
    7740/7740 [==============================] - 0s 42us/sample - loss: 0.3224 - val_loss: 0.3518
    Epoch 17/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3189 - val_loss: 0.3833
    Epoch 18/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3168 - val_loss: 0.3239
    Epoch 19/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3165 - val_loss: 0.4082
    Epoch 20/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3133 - val_loss: 0.3160
    Epoch 21/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3114 - val_loss: 0.3317
    Epoch 22/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3098 - val_loss: 0.3171
    Epoch 23/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3073 - val_loss: 0.3239
    Epoch 24/100
    7740/7740 [==============================] - 0s 44us/sample - loss: 0.3062 - val_loss: 0.3073
    Epoch 25/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3044 - val_loss: 0.3243
    Epoch 26/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3034 - val_loss: 0.3801
    Epoch 27/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3025 - val_loss: 0.3579
    Epoch 28/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3026 - val_loss: 0.3355
    Epoch 29/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3003 - val_loss: 0.3455
    Epoch 30/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.2983 - val_loss: 0.3128
    Epoch 31/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2955 - val_loss: 0.3165
    Epoch 32/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.2940 - val_loss: 0.3149
    Epoch 33/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.2909 - val_loss: 0.3912
    Epoch 34/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.2930 - val_loss: 0.3209
    3870/3870 [==============================] - 0s 16us/sample - loss: 0.3271
    7740/7740 [==============================] - 0s 17us/sample - loss: 0.2891
    [CV]  learning_rate=0.004576909498448105, n_hidden=2, n_neurons=87, total=  11.3s
    [CV] learning_rate=0.004576909498448105, n_hidden=2, n_neurons=87 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 58us/sample - loss: 1.0489 - val_loss: 1.0151
    Epoch 2/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.5291 - val_loss: 0.5506
    Epoch 3/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.4577 - val_loss: 0.4543
    Epoch 4/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.4166 - val_loss: 0.4211
    Epoch 5/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3932 - val_loss: 0.4991
    Epoch 6/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3774 - val_loss: 0.6786
    Epoch 7/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3659 - val_loss: 0.9418
    Epoch 8/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3593 - val_loss: 0.8760
    Epoch 9/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3531 - val_loss: 0.9404
    Epoch 10/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3499 - val_loss: 1.0715
    Epoch 11/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3454 - val_loss: 0.9127
    Epoch 12/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3412 - val_loss: 1.0588
    Epoch 13/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3385 - val_loss: 0.9105
    Epoch 14/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3345 - val_loss: 0.9548
    3870/3870 [==============================] - 0s 16us/sample - loss: 0.3628
    7740/7740 [==============================] - 0s 17us/sample - loss: 0.3311
    [CV]  learning_rate=0.004576909498448105, n_hidden=2, n_neurons=87, total=   4.9s
    [CV] learning_rate=0.004576909498448105, n_hidden=2, n_neurons=87 ....
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 58us/sample - loss: 0.9595 - val_loss: 10.8814
    Epoch 2/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.6306 - val_loss: 2.7405
    Epoch 3/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.4783 - val_loss: 0.4214
    Epoch 4/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.4204 - val_loss: 0.3964
    Epoch 5/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3979 - val_loss: 0.4372
    Epoch 6/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3852 - val_loss: 0.4389
    Epoch 7/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3759 - val_loss: 0.4287
    Epoch 8/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3691 - val_loss: 0.4075
    Epoch 9/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3640 - val_loss: 0.4152
    Epoch 10/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3600 - val_loss: 0.4284
    Epoch 11/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3548 - val_loss: 0.3699
    Epoch 12/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3526 - val_loss: 0.4052
    Epoch 13/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3497 - val_loss: 0.3468
    Epoch 14/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3462 - val_loss: 0.3865
    Epoch 15/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3427 - val_loss: 0.3764
    Epoch 16/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3404 - val_loss: 0.3697
    Epoch 17/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3379 - val_loss: 0.3259
    Epoch 18/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3358 - val_loss: 0.3488
    Epoch 19/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3335 - val_loss: 0.3473
    Epoch 20/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3308 - val_loss: 0.3886
    Epoch 21/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3314 - val_loss: 0.3202
    Epoch 22/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3279 - val_loss: 0.3620
    Epoch 23/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3263 - val_loss: 0.3380
    Epoch 24/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3246 - val_loss: 0.4124
    Epoch 25/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3234 - val_loss: 0.3171
    Epoch 26/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3224 - val_loss: 0.3410
    Epoch 27/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3195 - val_loss: 0.3358
    Epoch 28/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3169 - val_loss: 0.3893
    Epoch 29/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3175 - val_loss: 0.3208
    Epoch 30/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3168 - val_loss: 0.3512
    Epoch 31/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3147 - val_loss: 0.3236
    Epoch 32/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3111 - val_loss: 0.3870
    Epoch 33/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3105 - val_loss: 0.4080
    Epoch 34/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.3108 - val_loss: 0.3167
    Epoch 35/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3102 - val_loss: 0.3398
    Epoch 36/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.3071 - val_loss: 0.3612
    Epoch 37/100
    7740/7740 [==============================] - 0s 42us/sample - loss: 0.3062 - val_loss: 0.3460
    Epoch 38/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3046 - val_loss: 0.4104
    Epoch 39/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3079 - val_loss: 0.2997
    Epoch 40/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3056 - val_loss: 0.3803
    Epoch 41/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3041 - val_loss: 0.3085
    Epoch 42/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3026 - val_loss: 0.3959
    Epoch 43/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.3006 - val_loss: 0.3498
    Epoch 44/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.2981 - val_loss: 0.3024
    Epoch 45/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2986 - val_loss: 0.3475
    Epoch 46/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2949 - val_loss: 0.3191
    Epoch 47/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2946 - val_loss: 0.3201
    Epoch 48/100
    7740/7740 [==============================] - 0s 40us/sample - loss: 0.2937 - val_loss: 0.3001
    Epoch 49/100
    7740/7740 [==============================] - 0s 41us/sample - loss: 0.2929 - val_loss: 0.3431
    3870/3870 [==============================] - 0s 15us/sample - loss: 0.3356
    7740/7740 [==============================] - 0s 16us/sample - loss: 0.3133
    [CV]  learning_rate=0.004576909498448105, n_hidden=2, n_neurons=87, total=  15.6s
    [CV] learning_rate=0.0011714615165291644, n_hidden=0, n_neurons=56 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 46us/sample - loss: 2.4510 - val_loss: 5.1025
    Epoch 2/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.9274 - val_loss: 0.8434
    Epoch 3/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.7575 - val_loss: 0.7224
    Epoch 4/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.7072 - val_loss: 0.6691
    Epoch 5/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.6692 - val_loss: 0.6309
    Epoch 6/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.6383 - val_loss: 0.6009
    Epoch 7/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.6099 - val_loss: 0.5716
    Epoch 8/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5841 - val_loss: 0.5599
    Epoch 9/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5598 - val_loss: 0.5321
    Epoch 10/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5414 - val_loss: 0.5070
    Epoch 11/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5225 - val_loss: 0.5337
    Epoch 12/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5066 - val_loss: 0.4755
    Epoch 13/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4927 - val_loss: 0.4675
    Epoch 14/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4805 - val_loss: 0.4664
    Epoch 15/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4693 - val_loss: 0.4675
    Epoch 16/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4607 - val_loss: 0.4314
    Epoch 17/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4532 - val_loss: 0.4594
    Epoch 18/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4460 - val_loss: 0.4296
    Epoch 19/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4400 - val_loss: 0.4329
    Epoch 20/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4350 - val_loss: 0.4289
    Epoch 21/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4304 - val_loss: 0.4344
    Epoch 22/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4264 - val_loss: 0.4139
    Epoch 23/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4227 - val_loss: 0.4219
    Epoch 24/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4191 - val_loss: 0.4107
    Epoch 25/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4165 - val_loss: 0.4236
    Epoch 26/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4137 - val_loss: 0.4210
    Epoch 27/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4110 - val_loss: 0.4112
    Epoch 28/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4091 - val_loss: 0.4074
    Epoch 29/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4068 - val_loss: 0.4051
    Epoch 30/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4047 - val_loss: 0.3898
    Epoch 31/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4025 - val_loss: 0.4200
    Epoch 32/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4012 - val_loss: 0.3944
    Epoch 33/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3991 - val_loss: 0.3930
    Epoch 34/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3976 - val_loss: 0.4333
    Epoch 35/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3957 - val_loss: 0.3851
    Epoch 36/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3948 - val_loss: 0.3853
    Epoch 37/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3930 - val_loss: 0.3914
    Epoch 38/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3915 - val_loss: 0.4464
    Epoch 39/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3907 - val_loss: 0.3995
    Epoch 40/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3896 - val_loss: 0.4028
    Epoch 41/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3882 - val_loss: 0.3802
    Epoch 42/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3859 - val_loss: 0.3964
    Epoch 43/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3854 - val_loss: 0.4256
    Epoch 44/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3844 - val_loss: 0.3978
    Epoch 45/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3836 - val_loss: 0.3725
    Epoch 46/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3819 - val_loss: 0.4094
    Epoch 47/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3822 - val_loss: 0.3933
    Epoch 48/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3808 - val_loss: 0.3953
    Epoch 49/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3797 - val_loss: 0.4138
    Epoch 50/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3788 - val_loss: 0.3889
    Epoch 51/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3782 - val_loss: 0.3666
    Epoch 52/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3773 - val_loss: 0.4196
    Epoch 53/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3768 - val_loss: 0.3661
    Epoch 54/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3759 - val_loss: 0.3684
    Epoch 55/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3753 - val_loss: 0.3827
    Epoch 56/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3739 - val_loss: 0.3700
    Epoch 57/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3738 - val_loss: 0.4040
    Epoch 58/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3734 - val_loss: 0.4116
    Epoch 59/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3726 - val_loss: 0.3979
    Epoch 60/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3716 - val_loss: 0.3631
    Epoch 61/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3714 - val_loss: 0.4177
    Epoch 62/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3709 - val_loss: 0.3661
    Epoch 63/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3700 - val_loss: 0.4091
    Epoch 64/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3696 - val_loss: 0.3683
    Epoch 65/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3692 - val_loss: 0.3736
    Epoch 66/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3687 - val_loss: 0.3813
    Epoch 67/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3679 - val_loss: 0.3978
    Epoch 68/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3676 - val_loss: 0.3629
    Epoch 69/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3667 - val_loss: 0.3627
    Epoch 70/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3669 - val_loss: 0.3584
    Epoch 71/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3658 - val_loss: 0.3676
    Epoch 72/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3658 - val_loss: 0.3675
    Epoch 73/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3650 - val_loss: 0.4163
    Epoch 74/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3647 - val_loss: 0.3869
    Epoch 75/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3644 - val_loss: 0.3578
    Epoch 76/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3634 - val_loss: 0.4146
    Epoch 77/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3635 - val_loss: 0.3891
    Epoch 78/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3629 - val_loss: 0.3620
    Epoch 79/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3623 - val_loss: 0.3643
    Epoch 80/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3620 - val_loss: 0.3540
    Epoch 81/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3616 - val_loss: 0.4140
    Epoch 82/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3612 - val_loss: 0.3953
    Epoch 83/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3608 - val_loss: 0.4081
    Epoch 84/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3603 - val_loss: 0.3990
    Epoch 85/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3602 - val_loss: 0.3479
    Epoch 86/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3590 - val_loss: 0.4146
    Epoch 87/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3596 - val_loss: 0.3958
    Epoch 88/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3588 - val_loss: 0.3643
    Epoch 89/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3581 - val_loss: 0.3519
    Epoch 90/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3577 - val_loss: 0.4248
    Epoch 91/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3577 - val_loss: 0.3980
    Epoch 92/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3572 - val_loss: 0.3431
    Epoch 93/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3568 - val_loss: 0.4402
    Epoch 94/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3570 - val_loss: 0.3865
    Epoch 95/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3560 - val_loss: 0.3469
    Epoch 96/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3558 - val_loss: 0.3514
    Epoch 97/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3556 - val_loss: 0.4285
    Epoch 98/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3553 - val_loss: 0.3403
    Epoch 99/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3549 - val_loss: 0.3584
    Epoch 100/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3539 - val_loss: 0.3579
    3870/3870 [==============================] - 0s 12us/sample - loss: 0.3695
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3534
    [CV]  learning_rate=0.0011714615165291644, n_hidden=0, n_neurons=56, total=  26.8s
    [CV] learning_rate=0.0011714615165291644, n_hidden=0, n_neurons=56 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 46us/sample - loss: 2.2833 - val_loss: 9.3171
    Epoch 2/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.8871 - val_loss: 6.2420
    Epoch 3/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.7421 - val_loss: 3.4939
    Epoch 4/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6852 - val_loss: 1.9102
    Epoch 5/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6445 - val_loss: 1.0602
    Epoch 6/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6116 - val_loss: 0.6847
    Epoch 7/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5845 - val_loss: 0.5604
    Epoch 8/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5618 - val_loss: 0.5700
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5430 - val_loss: 0.6534
    Epoch 10/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5268 - val_loss: 0.7650
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5133 - val_loss: 0.8208
    Epoch 12/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5014 - val_loss: 0.8494
    Epoch 13/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4916 - val_loss: 0.8949
    Epoch 14/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4826 - val_loss: 0.9046
    Epoch 15/100
    7740/7740 [==============================] - 0s 32us/sample - loss: 0.4747 - val_loss: 0.8815
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4677 - val_loss: 0.8587
    Epoch 17/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4616 - val_loss: 0.7927
    3870/3870 [==============================] - 0s 12us/sample - loss: 0.4803
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.4579
    [CV]  learning_rate=0.0011714615165291644, n_hidden=0, n_neurons=56, total=   4.7s
    [CV] learning_rate=0.0011714615165291644, n_hidden=0, n_neurons=56 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 47us/sample - loss: 2.0724 - val_loss: 1.4705
    Epoch 2/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.7958 - val_loss: 0.7538
    Epoch 3/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6896 - val_loss: 0.6508
    Epoch 4/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6477 - val_loss: 0.6159
    Epoch 5/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.6167 - val_loss: 0.5844
    Epoch 6/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5906 - val_loss: 0.5521
    Epoch 7/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5675 - val_loss: 0.5322
    Epoch 8/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5478 - val_loss: 0.5150
    Epoch 9/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5305 - val_loss: 0.5061
    Epoch 10/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.5161 - val_loss: 0.4909
    Epoch 11/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.5025 - val_loss: 0.4726
    Epoch 12/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4914 - val_loss: 0.4612
    Epoch 13/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4815 - val_loss: 0.4534
    Epoch 14/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4728 - val_loss: 0.4438
    Epoch 15/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4650 - val_loss: 0.4357
    Epoch 16/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4585 - val_loss: 0.4313
    Epoch 17/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4522 - val_loss: 0.4274
    Epoch 18/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4470 - val_loss: 0.4222
    Epoch 19/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4419 - val_loss: 0.4129
    Epoch 20/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4377 - val_loss: 0.4086
    Epoch 21/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4337 - val_loss: 0.4057
    Epoch 22/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4300 - val_loss: 0.4034
    Epoch 23/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4266 - val_loss: 0.3982
    Epoch 24/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4237 - val_loss: 0.3950
    Epoch 25/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4206 - val_loss: 0.4003
    Epoch 26/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4182 - val_loss: 0.3956
    Epoch 27/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4158 - val_loss: 0.3966
    Epoch 28/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4134 - val_loss: 0.3945
    Epoch 29/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4114 - val_loss: 0.3836
    Epoch 30/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4094 - val_loss: 0.3941
    Epoch 31/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4074 - val_loss: 0.3814
    Epoch 32/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4053 - val_loss: 0.3792
    Epoch 33/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4041 - val_loss: 0.3928
    Epoch 34/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.4027 - val_loss: 0.3802
    Epoch 35/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.4010 - val_loss: 0.3835
    Epoch 36/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3997 - val_loss: 0.3741
    Epoch 37/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3981 - val_loss: 0.3885
    Epoch 38/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3974 - val_loss: 0.3860
    Epoch 39/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3962 - val_loss: 0.3697
    Epoch 40/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3949 - val_loss: 0.3734
    Epoch 41/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3940 - val_loss: 0.3706
    Epoch 42/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3928 - val_loss: 0.3909
    Epoch 43/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3920 - val_loss: 0.3772
    Epoch 44/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3908 - val_loss: 0.3750
    Epoch 45/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3900 - val_loss: 0.3648
    Epoch 46/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3891 - val_loss: 0.3807
    Epoch 47/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3887 - val_loss: 0.3625
    Epoch 48/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3876 - val_loss: 0.3746
    Epoch 49/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3867 - val_loss: 0.3754
    Epoch 50/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3864 - val_loss: 0.3645
    Epoch 51/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3851 - val_loss: 0.3951
    Epoch 52/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3845 - val_loss: 0.3832
    Epoch 53/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3841 - val_loss: 0.3613
    Epoch 54/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3829 - val_loss: 0.3953
    Epoch 55/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3825 - val_loss: 0.3757
    Epoch 56/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3818 - val_loss: 0.3647
    Epoch 57/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3810 - val_loss: 0.3959
    Epoch 58/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3807 - val_loss: 0.3563
    Epoch 59/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3802 - val_loss: 0.3882
    Epoch 60/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3795 - val_loss: 0.3784
    Epoch 61/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3786 - val_loss: 0.3908
    Epoch 62/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3785 - val_loss: 0.3719
    Epoch 63/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3778 - val_loss: 0.3788
    Epoch 64/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3774 - val_loss: 0.3566
    Epoch 65/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3766 - val_loss: 0.3630
    Epoch 66/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3764 - val_loss: 0.3537
    Epoch 67/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3756 - val_loss: 0.3850
    Epoch 68/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3755 - val_loss: 0.3769
    Epoch 69/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3749 - val_loss: 0.3656
    Epoch 70/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3744 - val_loss: 0.3584
    Epoch 71/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3739 - val_loss: 0.3706
    Epoch 72/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3735 - val_loss: 0.3533
    Epoch 73/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3731 - val_loss: 0.3556
    Epoch 74/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3724 - val_loss: 0.3960
    Epoch 75/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3724 - val_loss: 0.3755
    Epoch 76/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3718 - val_loss: 0.3667
    Epoch 77/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3715 - val_loss: 0.3575
    Epoch 78/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3708 - val_loss: 0.3851
    Epoch 79/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3707 - val_loss: 0.3694
    Epoch 80/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3702 - val_loss: 0.3824
    Epoch 81/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3700 - val_loss: 0.3673
    Epoch 82/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3696 - val_loss: 0.3491
    Epoch 83/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3691 - val_loss: 0.3479
    Epoch 84/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3683 - val_loss: 0.3513
    Epoch 85/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3683 - val_loss: 0.3882
    Epoch 86/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3679 - val_loss: 0.3776
    Epoch 87/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3677 - val_loss: 0.3607
    Epoch 88/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3670 - val_loss: 0.3911
    Epoch 89/100
    7740/7740 [==============================] - 0s 33us/sample - loss: 0.3670 - val_loss: 0.3468
    Epoch 90/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3667 - val_loss: 0.3470
    Epoch 91/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3661 - val_loss: 0.3466
    Epoch 92/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3658 - val_loss: 0.3778
    Epoch 93/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3654 - val_loss: 0.3447
    Epoch 94/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.3649 - val_loss: 0.3542
    Epoch 95/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3647 - val_loss: 0.3634
    Epoch 96/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3646 - val_loss: 0.3474
    Epoch 97/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3640 - val_loss: 0.3834
    Epoch 98/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3640 - val_loss: 0.3535
    Epoch 99/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3636 - val_loss: 0.3502
    Epoch 100/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3630 - val_loss: 0.3472
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3650
    7740/7740 [==============================] - 0s 14us/sample - loss: 0.3623
    [CV]  learning_rate=0.0011714615165291644, n_hidden=0, n_neurons=56, total=  26.5s
    [CV] learning_rate=0.0012578066689117673, n_hidden=1, n_neurons=60 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 52us/sample - loss: 2.0678 - val_loss: 1.4867
    Epoch 2/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.7582 - val_loss: 0.7377
    Epoch 3/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.6402 - val_loss: 0.5802
    Epoch 4/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5891 - val_loss: 0.5485
    Epoch 5/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.5528 - val_loss: 0.5201
    Epoch 6/100
    7740/7740 [==============================] - 0s 39us/sample - loss: 0.5260 - val_loss: 0.4890
    Epoch 7/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.5025 - val_loss: 0.4603
    Epoch 8/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4839 - val_loss: 0.4542
    Epoch 9/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4686 - val_loss: 0.4417
    Epoch 10/100
    7740/7740 [==============================] - 0s 38us/sample - loss: 0.4565 - val_loss: 0.4411
    Epoch 11/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4466 - val_loss: 0.4201
    Epoch 12/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4381 - val_loss: 0.4113
    Epoch 13/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4314 - val_loss: 0.4170
    Epoch 14/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4250 - val_loss: 0.4002
    Epoch 15/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4195 - val_loss: 0.4316
    Epoch 16/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4149 - val_loss: 0.3887
    Epoch 17/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4103 - val_loss: 0.3882
    Epoch 18/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4060 - val_loss: 0.4235
    Epoch 19/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.4030 - val_loss: 0.4081
    Epoch 20/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3990 - val_loss: 0.4274
    Epoch 21/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3962 - val_loss: 0.3974
    Epoch 22/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3929 - val_loss: 0.3794
    Epoch 23/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3899 - val_loss: 0.4050
    Epoch 24/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3879 - val_loss: 0.3742
    Epoch 25/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3849 - val_loss: 0.3693
    Epoch 26/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3824 - val_loss: 0.3617
    Epoch 27/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3800 - val_loss: 0.3715
    Epoch 28/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3780 - val_loss: 0.3569
    Epoch 29/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3758 - val_loss: 0.3695
    Epoch 30/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3735 - val_loss: 0.3967
    Epoch 31/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3723 - val_loss: 0.3551
    Epoch 32/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3700 - val_loss: 0.3600
    Epoch 33/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3680 - val_loss: 0.3780
    Epoch 34/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3665 - val_loss: 0.3552
    Epoch 35/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3650 - val_loss: 0.3626
    Epoch 36/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3631 - val_loss: 0.3454
    Epoch 37/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3617 - val_loss: 0.3514
    Epoch 38/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3603 - val_loss: 0.3436
    Epoch 39/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3589 - val_loss: 0.3621
    Epoch 40/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3575 - val_loss: 0.3398
    Epoch 41/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3559 - val_loss: 0.3806
    Epoch 42/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3551 - val_loss: 0.3377
    Epoch 43/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3535 - val_loss: 0.3823
    Epoch 44/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3525 - val_loss: 0.3376
    Epoch 45/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3513 - val_loss: 0.3599
    Epoch 46/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3497 - val_loss: 0.3378
    Epoch 47/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3489 - val_loss: 0.3848
    Epoch 48/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3482 - val_loss: 0.3528
    Epoch 49/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3468 - val_loss: 0.3474
    Epoch 50/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3459 - val_loss: 0.3603
    Epoch 51/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3451 - val_loss: 0.3538
    Epoch 52/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3440 - val_loss: 0.3315
    Epoch 53/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3433 - val_loss: 0.3428
    Epoch 54/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3425 - val_loss: 0.3466
    Epoch 55/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3414 - val_loss: 0.3303
    Epoch 56/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3408 - val_loss: 0.3706
    Epoch 57/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3401 - val_loss: 0.3334
    Epoch 58/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3386 - val_loss: 0.3698
    Epoch 59/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3383 - val_loss: 0.3272
    Epoch 60/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3373 - val_loss: 0.4134
    Epoch 61/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3370 - val_loss: 0.3250
    Epoch 62/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3361 - val_loss: 0.4060
    Epoch 63/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3357 - val_loss: 0.3332
    Epoch 64/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3347 - val_loss: 0.3328
    Epoch 65/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3341 - val_loss: 0.3264
    Epoch 66/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3335 - val_loss: 0.3295
    Epoch 67/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3329 - val_loss: 0.3599
    Epoch 68/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3321 - val_loss: 0.3459
    Epoch 69/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3313 - val_loss: 0.3226
    Epoch 70/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3308 - val_loss: 0.3513
    Epoch 71/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3302 - val_loss: 0.3653
    Epoch 72/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3300 - val_loss: 0.3384
    Epoch 73/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3289 - val_loss: 0.3897
    Epoch 74/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3286 - val_loss: 0.3191
    Epoch 75/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3282 - val_loss: 0.3525
    Epoch 76/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3274 - val_loss: 0.3197
    Epoch 77/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3268 - val_loss: 0.4065
    Epoch 78/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3270 - val_loss: 0.3257
    Epoch 79/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3256 - val_loss: 0.3260
    Epoch 80/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3253 - val_loss: 0.3684
    Epoch 81/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3251 - val_loss: 0.3202
    Epoch 82/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3245 - val_loss: 0.4489
    Epoch 83/100
    7740/7740 [==============================] - 0s 35us/sample - loss: 0.3246 - val_loss: 0.3344
    Epoch 84/100
    7740/7740 [==============================] - 0s 34us/sample - loss: 0.3239 - val_loss: 0.4869
    3870/3870 [==============================] - 0s 13us/sample - loss: 0.3442
    7740/7740 [==============================] - 0s 15us/sample - loss: 0.3227
    [CV]  learning_rate=0.0012578066689117673, n_hidden=1, n_neurons=60, total=  23.7s
    [CV] learning_rate=0.0012578066689117673, n_hidden=1, n_neurons=60 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 50us/sample - loss: 1.7817 - val_loss: 9.1129
    Epoch 2/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.7881 - val_loss: 4.1277
    Epoch 3/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6598 - val_loss: 1.6439
    Epoch 4/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6022 - val_loss: 0.8057
    Epoch 5/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5604 - val_loss: 0.5135
    Epoch 6/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5278 - val_loss: 0.5437
    Epoch 7/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5023 - val_loss: 0.7848
    Epoch 8/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4825 - val_loss: 0.9492
    Epoch 9/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4660 - val_loss: 1.0810
    Epoch 10/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4530 - val_loss: 1.1596
    Epoch 11/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4423 - val_loss: 1.1965
    Epoch 12/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4338 - val_loss: 1.2383
    Epoch 13/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.4256 - val_loss: 1.1621
    Epoch 14/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4193 - val_loss: 1.0986
    Epoch 15/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4133 - val_loss: 1.0196
    3870/3870 [==============================] - 0s 14us/sample - loss: 0.4393
    7740/7740 [==============================] - 0s 15us/sample - loss: 0.4098
    [CV]  learning_rate=0.0012578066689117673, n_hidden=1, n_neurons=60, total=   4.6s
    [CV] learning_rate=0.0012578066689117673, n_hidden=1, n_neurons=60 ...
    Train on 7740 samples, validate on 3870 samples
    Epoch 1/100
    7740/7740 [==============================] - 0s 51us/sample - loss: 2.1548 - val_loss: 2.2374
    Epoch 2/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.8591 - val_loss: 0.7902
    Epoch 3/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.7066 - val_loss: 0.6506
    Epoch 4/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6480 - val_loss: 0.6075
    Epoch 5/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.6086 - val_loss: 0.5687
    Epoch 6/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5752 - val_loss: 0.5391
    Epoch 7/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5474 - val_loss: 0.5183
    Epoch 8/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5239 - val_loss: 0.4882
    Epoch 9/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.5039 - val_loss: 0.4673
    Epoch 10/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4862 - val_loss: 0.4519
    Epoch 11/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4721 - val_loss: 0.4395
    Epoch 12/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4603 - val_loss: 0.4273
    Epoch 13/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4498 - val_loss: 0.4201
    Epoch 14/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4415 - val_loss: 0.4119
    Epoch 15/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4346 - val_loss: 0.4132
    Epoch 16/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4276 - val_loss: 0.3996
    Epoch 17/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4230 - val_loss: 0.4010
    Epoch 18/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4180 - val_loss: 0.3988
    Epoch 19/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4138 - val_loss: 0.3940
    Epoch 20/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4100 - val_loss: 0.3974
    Epoch 21/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4067 - val_loss: 0.4028
    Epoch 22/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4044 - val_loss: 0.4049
    Epoch 23/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.4006 - val_loss: 0.3939
    Epoch 24/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3971 - val_loss: 0.3815
    Epoch 25/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3951 - val_loss: 0.3838
    Epoch 26/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3922 - val_loss: 0.3768
    Epoch 27/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3902 - val_loss: 0.3716
    Epoch 28/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3872 - val_loss: 0.3778
    Epoch 29/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3859 - val_loss: 0.3884
    Epoch 30/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3841 - val_loss: 0.3782
    Epoch 31/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3822 - val_loss: 0.3698
    Epoch 32/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3807 - val_loss: 0.3677
    Epoch 33/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3786 - val_loss: 0.3782
    Epoch 34/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3770 - val_loss: 0.3646
    Epoch 35/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3757 - val_loss: 0.3699
    Epoch 36/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3742 - val_loss: 0.3663
    Epoch 37/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3728 - val_loss: 0.3685
    Epoch 38/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3710 - val_loss: 0.3635
    Epoch 39/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3699 - val_loss: 0.3762
    Epoch 40/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3693 - val_loss: 0.3657
    Epoch 41/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3679 - val_loss: 0.3567
    Epoch 42/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3662 - val_loss: 0.3659
    Epoch 43/100
    7740/7740 [==============================] - 0s 37us/sample - loss: 0.3652 - val_loss: 0.3720
    Epoch 44/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3645 - val_loss: 0.3576
    Epoch 45/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3633 - val_loss: 0.3676
    Epoch 46/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3623 - val_loss: 0.3505
    Epoch 47/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3613 - val_loss: 0.3488
    Epoch 48/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3604 - val_loss: 0.3592
    Epoch 49/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3598 - val_loss: 0.3445
    Epoch 50/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3590 - val_loss: 0.3681
    Epoch 51/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3581 - val_loss: 0.3576
    Epoch 52/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3569 - val_loss: 0.3442
    Epoch 53/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3563 - val_loss: 0.3443
    Epoch 54/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3558 - val_loss: 0.3566
    Epoch 55/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3552 - val_loss: 0.3392
    Epoch 56/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3546 - val_loss: 0.3462
    Epoch 57/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3537 - val_loss: 0.3421
    Epoch 58/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3527 - val_loss: 0.3472
    Epoch 59/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3526 - val_loss: 0.3521
    Epoch 60/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3519 - val_loss: 0.3422
    Epoch 61/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3509 - val_loss: 0.3555
    Epoch 62/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3506 - val_loss: 0.3409
    Epoch 63/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3500 - val_loss: 0.3394
    Epoch 64/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3493 - val_loss: 0.3418
    Epoch 65/100
    7740/7740 [==============================] - 0s 36us/sample - loss: 0.3486 - val_loss: 0.3506
    3870/3870 [==============================] - 0s 14us/sample - loss: 0.3470
    7740/7740 [==============================] - 0s 15us/sample - loss: 0.3468
    [CV]  learning_rate=0.0012578066689117673, n_hidden=1, n_neurons=60, total=  18.6s


    [Parallel(n_jobs=1)]: Done  30 out of  30 | elapsed:  7.3min finished


    Train on 11610 samples, validate on 3870 samples
    Epoch 1/100
    11610/11610 [==============================] - 1s 46us/sample - loss: 0.8992 - val_loss: 2.7901
    Epoch 2/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.4828 - val_loss: 0.4578
    Epoch 3/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.4082 - val_loss: 0.3782
    Epoch 4/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3842 - val_loss: 0.4179
    Epoch 5/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3707 - val_loss: 0.3564
    Epoch 6/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3624 - val_loss: 0.3539
    Epoch 7/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3541 - val_loss: 0.3745
    Epoch 8/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3493 - val_loss: 0.4061
    Epoch 9/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3448 - val_loss: 0.3380
    Epoch 10/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3405 - val_loss: 0.3352
    Epoch 11/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3361 - val_loss: 0.4455
    Epoch 12/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3349 - val_loss: 0.6610
    Epoch 13/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3335 - val_loss: 0.4351
    Epoch 14/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3284 - val_loss: 0.3318
    Epoch 15/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3254 - val_loss: 0.3144
    Epoch 16/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3223 - val_loss: 0.4413
    Epoch 17/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3226 - val_loss: 0.3295
    Epoch 18/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3182 - val_loss: 0.4551
    Epoch 19/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3182 - val_loss: 0.7770
    Epoch 20/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3176 - val_loss: 0.8286
    Epoch 21/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3168 - val_loss: 0.4178
    Epoch 22/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3118 - val_loss: 0.3867
    Epoch 23/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3082 - val_loss: 0.3094
    Epoch 24/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.3063 - val_loss: 0.3018
    Epoch 25/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3046 - val_loss: 0.3086
    Epoch 26/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3035 - val_loss: 0.2955
    Epoch 27/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.3016 - val_loss: 0.2874
    Epoch 28/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2998 - val_loss: 0.3067
    Epoch 29/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2987 - val_loss: 0.2855
    Epoch 30/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2985 - val_loss: 0.4065
    Epoch 31/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2964 - val_loss: 0.4888
    Epoch 32/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2971 - val_loss: 0.6767
    Epoch 33/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.2964 - val_loss: 0.4657
    Epoch 34/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.2943 - val_loss: 0.3589
    Epoch 35/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2922 - val_loss: 0.3978
    Epoch 36/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2909 - val_loss: 0.6603
    Epoch 37/100
    11610/11610 [==============================] - 0s 36us/sample - loss: 0.2923 - val_loss: 0.4158
    Epoch 38/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2899 - val_loss: 0.3226
    Epoch 39/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2873 - val_loss: 0.2782
    Epoch 40/100
    11610/11610 [==============================] - 0s 36us/sample - loss: 0.2866 - val_loss: 0.3285
    Epoch 41/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2850 - val_loss: 0.4074
    Epoch 42/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2858 - val_loss: 0.3352
    Epoch 43/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2827 - val_loss: 0.2797
    Epoch 44/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.2824 - val_loss: 0.7323
    Epoch 45/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2838 - val_loss: 0.7834
    Epoch 46/100
    11610/11610 [==============================] - 0s 34us/sample - loss: 0.2888 - val_loss: 0.6376
    Epoch 47/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2844 - val_loss: 0.5744
    Epoch 48/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2819 - val_loss: 0.3071
    Epoch 49/100
    11610/11610 [==============================] - 0s 35us/sample - loss: 0.2790 - val_loss: 0.3484





    RandomizedSearchCV(cv=3, error_score='raise-deprecating',
              estimator=<tensorflow.python.keras.wrappers.scikit_learn.KerasRegressor object at 0x11213ed68>,
              fit_params=None, iid='warn', n_iter=10, n_jobs=None,
              param_distributions={'n_hidden': [0, 1, 2, 3], 'n_neurons': array([ 1,  2, ..., 98, 99]), 'learning_rate': <scipy.stats._distn_infrastructure.rv_frozen object at 0x1a3bcea4a8>},
              pre_dispatch='2*n_jobs', random_state=None, refit=True,
              return_train_score='warn', scoring=None, verbose=2)




```python
rnd_search_cv.best_params_
```




    {'learning_rate': 0.004038022680351922, 'n_hidden': 2, 'n_neurons': 69}




```python
rnd_search_cv.best_score_
```




    -0.3275334420977329




```python
rnd_search_cv.best_estimator_
```




    <tensorflow.python.keras.wrappers.scikit_learn.KerasRegressor at 0x1a3d960588>



Evaluate the best model found on the test set. You can either use the best estimator's `score()` method, or get its underlying Keras model *via* its `model` attribute, and call this model's `evaluate()` method. Note that the estimator returns the negative mean square error (it's a score, not a loss, so higher is better).


```python
rnd_search_cv.score(X_test_scaled, y_test)
```

    5160/5160 [==============================] - 0s 19us/sample - loss: 0.2968





    -0.2968322378604911




```python
model = rnd_search_cv.best_estimator_.model
model.evaluate(X_test_scaled, y_test)
```

    5160/5160 [==============================] - 0s 15us/sample - loss: 0.2968





    0.2968322378604911



Finally, save the best Keras model found. **Tip**: it is available via the best estimator's `model` attribute, and just need to call its `save()` method.


```python
model.save("my_fine_tuned_housing_model.h5")
```


```python

```


```python

```
