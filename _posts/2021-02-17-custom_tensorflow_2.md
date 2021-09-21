---
title: tf.keras 커스텀 하기
author: Gukwon Koo
categories: [ML, TF]
tags: [tensorflow]
pin: false
math: true
comments: true
date: 2021-02-17 21:26:00 +0900
---

![](https://miro.medium.com/max/4928/1*-QTg-_71YF0SVshMEaKZ_g.png)

최근에 python2 + tensorflow 1.x로 작성된 추천 시스템 레거시 코드를 유지 보수 및 개선하는 업무를 진행하고 있습니다. 기존 코드는 tensorflow 1.x 버전으로 짜여져 있어서 API의 통일성이 부족했고, 오픈 소스 코드에 기반하여 상황 마다 필요한 컴포넌트를 추가 하다보니 코드의 일관성도 많이 저해된 상태였습니다. 따라서 유지 보수의 용이성을 확보하기 위해 tensorflow를 2.0 버전으로 코드를 변환해야겠다는 결심을 하게 되었습니다. tensorflow 공식 문서에서는 tensorflow 2.0 버전의 특징이 다음의 네 가지로 요약되어 있습니다. 가독성이 향상되고, 디버깅도 편해질 것 같군요!

- [API Cleanup](https://www.tensorflow.org/guide/effective_tf2#api_cleanup)
- [Eager execution](https://www.tensorflow.org/guide/effective_tf2#eager_execution)
- [No more globals](https://www.tensorflow.org/guide/effective_tf2#no_more_globals)
- [Functions, not session](https://www.tensorflow.org/guide/effective_tf2#functions_not_sessions)

<br>

특히 저는 깔끔하면서도 정형화된 형태로 tf 코드를 작성하고, 필요하다면 low-level로 layer를 커스터마이징할 수 있어야 했으므로 tensorflow 2.0 + keras layer 조합으로 코드를 작성하였습니다. 본 글에서는 이와 관련된 내용을 공부하면서 얻은 지식을 다음의 내용을 중심으로 정리해보겠습니다.

- custom layer class
- custom loss class
- custom model class
- custom training loop

<br>

# Layer Class

---

keras layer를 만들 때, subclassing[^1]을 활용하여 클래스 형태로 만들 수 있습니다. 이와 같은 방식에는 두가지 장점이 있다고 생각하는데요. 

- 메서드 이름이나 메서드가 받는 argument 등에 대한 정형화된 형태가 존재하고, 이에 맞춰 코드를 작성해야 하기 때문에 **가독성이 향상**됩니다.
- 한편으로 layer에서 이루어지는 computation 로직은 직접 low-level로 작성할 수 있기 때문에 **커스터마이징**이 가능합니다.

<br>

layer class는 weight와 computation의 결합으로 표현할 수 있는데요. 텐서플로우 [공식 문서](https://www.tensorflow.org/guide/keras/custom_layers_and_models)에서는 layer class에 대해 다음과 같이 설명하고 있습니다. 한마디로 정리하면 weight와 computation 과정이 결합된 object라고 말할 수 있겠습니다.

>One of the central abstraction in Keras is the `Layer` class. A layer encapsulates both a state (the layer's "weights") and a transformation from inputs to outputs (a "call", the layer's forward pass).

<br>

[이 문서](https://www.tensorflow.org/guide/keras/custom_layers_and_models#best_practice_deferring_weight_creation_until_the_shape_of_the_inputs_is_known)에서는 custom layer class를 작성하는 **best practice**를 다음과 같이 제안하고 있습니다.

```python
class Linear(tf.keras.layers.Layer):
    def __init__(self, units=32):
        super(Linear, self).__init__()
        self.units = units

    def build(self, input_shape):
        self.w = self.add_weight(
            shape=(input_shape[-1], self.units),
            initializer="random_normal",
            trainable=True,
        )
        self.b = self.add_weight(
            shape=(self.units,), initializer="random_normal", trainable=True
        )

    def call(self, inputs):
        return tf.matmul(inputs, self.w) + self.b
```

- **\_\_init\_\_()**	

  해당 layer에서 활용하는 hyperparameter 등을 선언합니다.

- **build()**

  해당 layer에서 활용하는 trainable/non-trainable weights와 관련된 로직을 작성합니다. 특히 build 메서드에 weights 초기화 로직을 구현하면 해당 레이어의 인풋의 차원을 정확하게 알지 못하더라도 lazy 하게 작동하게 할 수 있다는 장점이 있습니다. build 메서드에 선언된 weights는 call 메서드가 처음 호출될 때 생성됩니다. 

- **call()**

  해당 layer의 computation과 관련된 로직을 작성합니다. 인풋을 받아 원하는 형태의 아웃풋을 리턴하도록 하면 되겠습니다. 

<br>

그러나 딥러닝 코드를 짜다보면 같은 computation 로직을 반복 사용해야할 경우가 많죠. 예를 들어 위 예시처럼 Linear(Dense) layer는 정말 많이 사용하는데요. 내가 만든 custom layer class 혹은 keras layer class 여러 개를 묶어서 다시 레이어 클래스를 만드려면 어떻게 해야할까요? [공식 문서](https://www.tensorflow.org/guide/keras/custom_layers_and_models#layers_are_recursively_composable)에서는 아래와 같은 방법을 제안합니다.

```python
class MLPBlock(tf.keras.layers.Layer):
    def __init__(self):
        super(MLPBlock, self).__init__()
        self.linear_1 = Linear(32)
        self.linear_2 = Linear(32)
        self.linear_3 = Linear(1)

    def call(self, inputs):
        x = self.linear_1(inputs)
        x = tf.nn.relu(x)
        x = self.linear_2(x)
        x = tf.nn.relu(x)
        return self.linear_3(x)
```

첫번째 예시와 거의 비슷한 형태인데요. 공통점은 computation 로직이 `call()` 메서드로 처리된다는 점입니다. 그러나 `build()` 메서드에 weight를 직접 선언하지 않고, layer class를 `__init__()` 메서드에 선언한다는 것이 차이점이네요. 이는 공식적으로 권장하는 방법입니다. 그 이유는 outer layer (e.g. MLPBlock)는 inner layer (e.g. Linear)의 weight를 자동으로 추적할수 있기 때문이라고 합니다. 이 부분은 공식 문서의 설명이 조금 부실한데요. outer layer class에서 `call()` 메서드가 처음으로 호출되면, computation 과정에 참여하는 inner layer들의 `call()` 메서드도 역시 처음으로 호출되게 됩니다. 이때 inner layer의 `build()` 메서드에 선언된 weight들이 lazy하게 생성됩니다. outer layer는 inner layer의 weight를 추적할 수 있도록 디자인 되어 있기 때문에 이와 같은 로직 작성이 가능한 것입니다.

<br>

# Loss Class

---

custom loss function을 만드는 방법은 크게 세 가지로 나눌 수 있습니다. 

- simple loss function
- nested loss function
- loss class

<br>

먼저 세가지 방식은 공통적으로 **loss를 계산하는 메서드가 `y_true`, `y_pred`라는 단 두가지 인자**만 받을 수 있다는 단점이 존재합니다. 물론 각 방법의 특징에 따라 추가적인 인자들을 활용할 수 있는 방안이 마련되어 있지만, 실제로 loss를 계산하는 메서드는 무조건 위의 두가지 인자만 전달받을 수 있습니다. 

사실 케라스 built-in function인 `model.fit()`를 활용하실 계획이 없으시다면, low-level로 loss 함수를 작성하셔도 관계없습니다. 그러나 `model.fit()`을 사용하실 계획이시라면 반드시 정해진 형태에 맞춰 코드를 작성해주셔야 합니다. `model.fit()` 메서드 내부에 loss와 관련된 부분은 `y_true`, `y_pred` 두 가지 인자만을 활용하도록 디자인 되어 있기 때문입니다.

<br>

## 2.1 &nbsp; Simple Loss Function

[공식 문서](https://keras.io/api/losses/#creating-custom-losses)에서 제안하는 가장 기본적인 custom keras loss 함수 작성 방법입니다.

```python
def my_loss_fn(y_true, y_pred):
    squared_difference = tf.square(y_true - y_pred)
    return tf.reduce_mean(squared_difference, axis=-1)  # Note the `axis=-1`
```

- `y_true`: ground_truth 값을 전달합니다.
- `y_pred`: 모델의 예측 값을 전달합니다.

이 방식의 가장 큰 문제는 `y_true`나 `y_pred` 이외의 값을 전혀 활용할 수 없다는 것입니다. 만일 loss 함수에 추가적인 파라미터가 요구된다면 이러한 방식의 loss 함수는 사용할 수 없습니다. 다음에 설명드릴 nested loss function이나 loss class 방식을 참고해주세요.

<br>

## 2.2 &nbsp; Nested Loss Function

기본적인 loss 함수의 단점을 보완하고 추가적인 파라미터를 전달하기 위해서 다음과 같은 방법으로 custom loss 함수를 작성할 수 있습니다.

```python
def my_loss_fn(threshold):
    def inner_fn(y_true, y_pred):
        squared_difference = tf.square(y_true - y_pred) * threshold
        return tf.reduce_mean(squared_difference, axis=-1)
    return inner_fn
```

nested loss function에는 파이썬 클로저(closure)[^2]의 개념이 활용 되었습니다. 클로저에 대한 자세한 내용은 [이 곳](https://shoark7.github.io/programming/python/closure-in-python)을 참고해주세요. 결론적으로 말씀드리면, threshold라는 변수는 inner_fn의 입장에서는 nonlocal 변수인데요. 클로저는 nonlocal 변수인 threshold에 담긴 값을 기억하고 참조할 수 있습니다. 따라서 squared_difference를 계산할 때, inner_fn 함수의 네임 스페이스 바깥 영역에서 선언된 threshold 변수를 참조할 수 있는 것입니다. nested loss 함수는 기본적인 loss 함수에 비해 조금 더 자유롭지만, 저는 개인적으로 클래스를 이용해서 조금 더 깔끔하게 함수를 작성하실 것을 추천드립니다.

<br>

## 2.3 &nbsp; Loss Class

개인적으로 custom loss를 작성할 때 가장 선호하는 방식입니다. 가장 가독성이 좋으면서도 외부 파라미터도 활용할 수 있기 때문입니다. 생성자 `__init__()`에 필요한 외부 파라미터를 선언해두고 `call()` 메서드에 실제 loss를 계산하는 로직을 작성하시면 됩니다.

```python
class CustomLoss(tf.keras.losses.Loss):
    def __init__(self, param1, param2, ...):
        super(CustomLoss, self).__init__()
        self.param1 = param1
        self.param2 = param2

    def call(self, y_true, y_pred):
        loss = ...
        return loss
```

<br>

loss class 방식은 아래와 같은 keras high-level api와 쉽게 결합 가능합니다.

```python
loss_fn = CustomLoss(param1, param2)
...
model.compile(optimizer=..., loss=loss_fn)
model.fit()
```

<br>

그러나 세 가지 방법 모두 여전히 `call()` 메서드에 `y_true`, `y_pred` 두 가지 인자만 전달할 수 있다는 단점은 존재하는데요. `y_true` 값이 여러 개일 경우에 문제가 될 수 있습니다. 예를 들어 negative sampling loss를 계산하는 상황을 가정해보겠습니다. 이 경우 `call()` 메서드에 `y_true_positive`, `y_true_negative`, `y_pred` 와 같이 인자를 3개를 전달해야 할겁니다.

인자를 3개를 전달하기 위한 여러 방법을 찾아보았는데, 제가 내린 결론은 `model.fit()` 메서드를 사용할 것이라면 인자를 3개 이상 전달할 수 있는 방법은 없다는 것입니다. 그래서 저는 우회적으로 `y_true_positive`와 `y_true_negative`를 reshape 및 concat 하여 `y_true` 인자로 전달한 후 `call()` method 내부에서 slice 및 reshape를 통해 negative sampling loss를 계산하는 식으로 로직을 작성했습니다. 혹시 더 좋은 방안이 있다면 제보 부탁드립니다!

<br>

# Model Class

---

tf.keras에서 커스텀 모델을 만드는 방법은 크게 3가지입니다. 

- [Sequential API](https://www.tensorflow.org/guide/keras/sequential_model)
- [Functional API](https://www.tensorflow.org/guide/keras/functional)
- [Model Class (via subclassing)](https://www.tensorflow.org/guide/keras/custom_layers_and_models#the_model_class)

<br>

세가지 방법 중 가장 low-level로 custom 할 수 있는 model class를 위주로 설명드리려고 합니다. [공식 문서](https://www.tensorflow.org/guide/keras/custom_layers_and_models#the_model_class)에서 제안한 예시를 통해 설명드리겠습니다.

<br>

## 3.1 &nbsp; Simple Model Class

```python
class ResNet(tf.keras.Model):

    def __init__(self, num_classes=1000):
        super(ResNet, self).__init__()
        self.block_1 = ResNetBlock()
        self.block_2 = ResNetBlock()
        self.global_pool = layers.GlobalAveragePooling2D()
        self.classifier = Dense(num_classes)

    def call(self, inputs):
        x = self.block_1(inputs)
        x = self.block_2(x)
        x = self.global_pool(x)
        return self.classifier(x)
```

- **tf.keras.Model 상속**

  tf.keras.Model의 상속을 통해서 `model.fit()`, `model.compile()`, `model.predict()` 등의 keras built-in 함수를 자연스럽게 사용할 수 있게 되는데요. [여기](https://github.com/tensorflow/tensorflow/blob/v2.4.1/tensorflow/python/keras/engine/training.py#L138-L2675)을 참고 하시면 `tf.keras.Model` 클래스가 `fit()`, `compile()` 등의 메서드를 가지고 있는 것을 볼 수 있습니다.

- **\_\_init\_\_()**

  hyperparameter나 layer class를 정의하는 메서드입니다.

- **call()**

  model의 computation이 일어나는 부분입니다. argument로 inputs를 받아서 custom 원하는 output을 계산하도록 로직을 작성하면 되겠습니다.

<br>

## 3.2 &nbsp; End-to-End Model

model class 방식의 가장 큰 장점은 여러 layer class를 조합하여 하나의 큰 모델로 만들 수 있다는 점입니다. 많은 분들께서 결국에는 이와 비슷한 형태로 model class를 만드실 것으로 생각합니다. [공식 문서]()의 예제를 조금 간소화하여 설명드릴게요. 아래의 예시는 *encode layer class*와 *decoder layer class*를 선언한 뒤에 *VariationalAutoEncoder model class*로 두 레이어를 조합하는 예시를 보여줍니다. 이와 같이 직접 만드신 layer class나 케라서 built-in layer class를 활용하여 원하시는 모델을 레고 조립하듯이 쌓아 나가시면 되겠습니다.

```python
class Encoder(tf.keras.layers.Layer):
    def __init__(self, latent_dim=32, ...):
        super(Encoder, self).__init__(...)
        ...
				
    def call(self, inputs):
        z_mean = ...
        z_log_var = ...
        z = ...
        return z_mean, z_log_var, z

class Decoder(tf.keras.layers.Layer):
    def __init__(self, intermediate_dim=64, ...):
        super(Decoder, self).__init__(...)
        ...

    def call(self, inputs):
        x = ...
        y = ...
        return y

class VariationalAutoEncoder(tf.keras.Model):
    def __init__(self, original_dim, ...):
        super(VariationalAutoEncoder, self).__init__(...)
        self.encoder = Encoder(...)
        self.decoder = Decoder(...)

    def call(self, inputs):
        x = self.encoder(...)
        y = self.decoder(...)
        ...
        return output
```

<br>

# Custom Training Loop

---

training loop를 작성하는 방법은 크게 두가지입니다.

- [for loop를 활용한 low-level 수준의 코드 작성](https://www.tensorflow.org/guide/keras/writing_a_training_loop_from_scratch#using_the_gradienttape_a_first_end-to-end_example)
- [model class에 train_step() 함수를 오버라이딩](https://www.tensorflow.org/guide/keras/customizing_what_happens_in_fit#a_first_simple_example)[^3]하여 작성

<br>

첫번째 방식은 코드를 작성하는 사람의 입맛에 맞게 매우 자유롭게 코드를 작성할 수 있다는 장점이 있지만, 그에 비례해서 코드 작성 과정에서 human error가 발생할 가능성이 높아집니다. 저는 자유롭게 코드를 작성할 수 있으면서도 `model.fit()`의 편리한 장점을 취하여 human error를 최소화 할 수 있는 두번째 방식으로 custom training loop를 작성하는 방법을 설명해드리고자 합니다.

<br>

섹션3에서 언급한 model class에 `train_step()` 메서드를 활용하면 됩니다. 코드는 다음과 같은 방식으로 작성하시면 됩니다. 보다 자세한 내용은 [공식 문서](https://www.tensorflow.org/guide/keras/customizing_what_happens_in_fit#a_first_simple_example)를 참조해주세요.

- tf.keras.Model을 상속 받아 model class를 만듭니다.
- model class에 `train_step()` 메서드를 작성합니다. `model.fit()`는 이 메서드를 활용하여 학습이 진행되도록 디자인 되어 있습니다.
- `tf.GradientTape` API는 [자동 미분(automatic differentiation)](https://www.tensorflow.org/guide/autodiff#automatic_differentiation_and_gradients) 기능을 제공합니다. 따라서 우리는 `train_step()` 메서드의 로직 작성 순서만 고려하면 됩니다.

```python
class CustomModel(tf.keras.Model):
    def __init__(self):
        ...
    def call(self):
        ...
    def train_step(self, data):
        # Unpack the data. Its structure depends on your model and on what you pass to `fit()`.
        x, y = data

        with tf.GradientTape() as tape:
            # Forward pass
            y_pred = self(x, training=True)  
            # Compute the loss value (the loss function is configured in `compile()`)
            loss = self.compiled_loss(y, y_pred, regularization_losses=self.losses)

        # Compute gradients
        trainable_vars = self.trainable_variables
        gradients = tape.gradient(loss, trainable_vars)
        # Update weights
        self.optimizer.apply_gradients(zip(gradients, trainable_vars))
        # Update metrics (includes the metric that tracks the loss)
        self.compiled_metrics.update_state(y, y_pred)
        # Return a dict mapping metric names to current value
        return {m.name: m.result() for m in self.metrics}
```

train_step() 메서드는 다음과 같은 순서로 진행됩니다.

- **Step1: computation 기록**

  `tf.GradientTape()` 컨텍스트 내부의 연산은 추적되어서 이후 자동 미분이 가능하게 됩니다.

- **Step2: forward pass** 

  model class의 `call()` 메서드를 활용해서 모델의 예측값을 계산합니다.

- **Step3: compute loss**

  `model.compile()` 메서드에 전달한 loss 함수로 loss가 계산됩니다.

- **Step4: compute gradients**

  후진 방식 자동 미분(reverse mode differentiation)을 사용해 기록된 연산의 gradient를 계산합니다.

- **Step5: update weights**

  계산된 gradient와 `model.compile()` 메서드에 전달한 optimizer를 활용해 weight를 업데이트 합니다.

- **Step6: update metrics**

  현재 step에서의 metrics를 계산합니다.

- **Step7: return metric dictionary**

  training step 마다 계산된 metrics를 progress bar에 출력하기 위해 해당 딕셔너리를 return 합니다.

<br>

# Conclusion

tf.keras를 커스텀 하는 방법은 매우 다양하고 알아야 되는 개념도 많다는 것을 알게 되었습니다. 특히 저는 keras built-in function들을 최대한 활용할 수 있는 방안을 고민하면서 많은 시행착오를 겪었고 그 결과 제가 체득한 가장 최선의 시나리오(?)를 정리하였습니다. 다만 최대한 쉽게 작성하려고는 했으나, 애초에 알아야할 개념들이 많이 있어서 글 자체의 난이도 조절에는 실패해버린 것 같습니다...

업무를 어느 정도 일단락 하고 지난 일들을 돌이켜 보면 keras built-in function을 활용한다는 것이 양날이 검이 될 수도 있다는 생각이 들었습니다. 딥러닝 코드에서 어느 정도 정형화된 부분은 keras built-in function을 활용하면 human error를 줄일 수 있는 것은 확실한 장점이라고 생각합니다. 그러나 정말 모든 것을 low-level로 하고 싶은신 분들에게는 어느 정도 규격화된 틀이 있다는 것이 답답하게 느껴질 수도 있을 것 같습니다. 특히 pytorch를 사용하셨던 분들에게는 더욱 크게 다가올 것 같네요.

아 그리고 이 글에서 담지 못한 이야기들도 많이 있는데요. metric로 커스터마이징이 가능하고, 만들어진 모델을 tf.serving container를 활용해서 API 서버를 구축하시려면 tf.graph에 대한 개념 이해도 필요합니다. 추후에 기회가 되면 이 부분도 다뤄보겠습니다.

# Reference

---

- [Simple custom layer example: Antirectifier](https://keras.io/examples/keras_recipes/antirectifier/)
- [직접 케라스 레이어 만들기](https://keras.io/ko/layers/writing-your-own-keras-layers/)
- [Training and evaluation with the built-in methods](https://www.tensorflow.org/guide/keras/train_and_evaluate)
- [Making new Layers and Models via subclassing](https://www.tensorflow.org/guide/keras/custom_layers_and_models)
- [Writing a training loop from scratch](https://www.tensorflow.org/guide/keras/writing_a_training_loop_from_scratch)
- [Customize what happens in Model.fit](https://www.tensorflow.org/guide/keras/customizing_what_happens_in_fit)
- [Customizing what happens in fit()](https://keras.io/guides/customizing_what_happens_in_fit/)
- [tensorflow custom loss](https://junstar92.tistory.com/144?category=905975)
- [Losses (keras.io)](https://keras.io/api/losses/#creating-custom-losses)
- [The Sequential model](https://www.tensorflow.org/guide/keras/sequential_model)
- [Introduction to gradients and automatic differentiation](https://www.tensorflow.org/guide/autodiff)
- [파이썬 강좌 번외편. 클로저(Closure)](https://blog.hexabrain.net/347)
- [클로져(Closure) 이해하기](https://whatisthenext.tistory.com/112?category=761276)
- [Python의 Closure에 대해 알아보자](https://shoark7.github.io/programming/python/closure-in-python)

<br>

# footnote

---

[^1]: 서브클래싱(subclassing)이란 객체 지향 프로그래밍(OOP)에 등장하는 개념으로서 [구현되어 있는 클래스를 상속하는 것](https://epicdevsold.tistory.com/177)을 말합니다.
[^2]: [클로저(closure)란 자신을 둘러싼 네임 스페이스에 존재하는 변수를 기억할 수 있는 함수](https://shoark7.github.io/programming/python/closure-in-python)를 뜻합니다. 대표적으로 파이썬 데코레이터는 클로저의 개념을 활용한 것입니다.
[^3]: 오버라이딩(overriding)이란 부모 클래스의 메서드를 자식 클래스에서 재정의하는 것을 말합니다. 자식 클래스의 메서드를 작성할 때 부모 클래스의 메서드와 이름은 같지만 로직을 다르게 하고 싶을 때 사용합니다.

