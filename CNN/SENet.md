# SENet(Squeeze and excitation networks)

 작성일 2018-07-18 

|  In [Deep Learning ](https://jayhey.github.io/category/#/Deep%20Learning)

|  [2 Comments](https://jayhey.github.io/deep%20learning/2018/07/18/SENet/#comments)

2017 이미지넷 챌린지(ILSVRC 2017)에서 우승한 SENet입니다. 피쳐맵의 압축(squeeze)과 재조정(recalibration)을 통한 스케일이 핵심인 방법론입니다.

# Squeeze and excitation networks

알렉스넷이 나온 이후로 몇 년 동안은 ILSVRC에서 어떻게든 에러를 낮추기 위한 노력이 계속되었습니다. 그러나 레즈넷 이후로는 classification 분야에서는 네트워크 구조에 대한 관심이 많이 줄어들었습니다. 이번 포스트에서는 channel-wise feature response를 적절하게 조절해주는 **“Squeeze-and-Excitation(SE)”** 방법론을 제안한 SENet에 대하여 알아보도록 하겠습니다.

## Introduction

CNN 네트워크를 잘 살펴보면, convolution filter 하나하나가 이미지 또는 피쳐맵의 “local”을 학습합니다. 즉, local receptive field에 있는 정보들의 조합이라고 말할 수 있습니다. 이 조합들을 활성함수에 통과시켜 비선형적인 관계를 추론하고 pooling과 같은 방법으로 큰 피쳐들을 작게 만들어 한 번에 볼 수 있게 크게 만들기도 합니다. 이런 식으로 CNN은 global receptive field의 관계를 효율적으로 다루기 때문에 이미지 분류와 같은 분야에서는 인간을 뛰어넘는 성능을 낼 수 있습니다. 당연하게도 일반적인 CNN구조보다 효율적으로 피쳐들을 잘 다룰 수 있는 다양한 구조에 대한 연구가 있었고, 매년 ILSVRC에서 1등을 차지했습니다. 대표적으로는 GoogLeNet의 Inception module, ResNet의 Skip-connection, [DenseNet](https://jayhey.github.io/deep%20learning/2017/10/13/DenseNet_1/)의 Dense-connection 등 매년 기발한 네트워크로 우승을 차지한 네트워크들이 있습니다.

논문의 저자들은 **Squeeze-and-Excitation**라는 방법을 적용해서 ILSVRC 2017을 우승하게됩니다. 이 아이디어를 논문에 나온 영어로 간단히 써보면 다음과 같습니다.~~(사실 한국어로 써놓고 보니 더 말이 이상해서 영어로 바꿨습니다. 영어로 보는게 이해가 더 빠를듯 합니다)~~

- **Our goal is to improve the representational power of a network by explicitly modelling the interdependencies between the channels of its convolutional features.**

이를 위해서, 논문의 저자들은 squeeze and excitation이라는 아이디어를 제시합니다. 이를 가지고 기존 네트워크를 거친 피쳐맵들을 feature recalibration(재조정)할 수 있다고 합니다. 자세히 살펴보면, SE는각 피쳐맵에 대한 전체 정보를 요약하는 Squeeze operation, 이를 통해 각 피쳐맵의 중요도를 스케일해주는 excitation operation으로 이루어져 있습니다. 이렇게 하나의 덩어리를 SE block이라고 하며 이에 대해서는 아래에서 다시 자세히 말씀드리도록 하겠습니다.

저자들은 SE block의 장점이 다음과 같다고 말하고 있습니다.

1. 네트워크 어떤 곳이라도 바로 붙일 수 있습니다. VGG, GoogLeNet, ResNet 등 어느 네트워크에도 바로 부착이 가능합니다.
2. 파라미터의 증가량에 비해 모델 성능 향상도가 매우 큽니다. 이는 모델 복잡도(Model complexity)와 계산 복잡도(computational burden)이 크게 증가하지 않다는 장점이 있습니다.

이제 SE block에 대해 자세히 살펴보도록 하겠습니다.

## Squeeze-and-Excitation Blocks

[![Imgur](https://i.imgur.com/9UFjxDA.png)](https://i.imgur.com/9UFjxDA.png)

위 그림 Figure 1이 SE block의 전체적인 모습을 나타낸 그림입니다. 그림 내에 있는 수식은 다음과 같습니다.
$$
F_{tr}:X→U,X∈R^{H′×W′×C′},U∈R^{H×W×C}
$$


Ftr은 단순한 convolution 연산입니다. 그림에서도 확인할 수 있지만 콘볼루션 연산을 거치고 H, W, C 로 변화한 것을 확인할 수 있습니다. 나머지 수식은 다음과 같습니다.
$$
V=[v1,v2,...,vc]
$$
: 필터 집합 V와 각 필터의 파라미터. vc는 c번째 필터의 파라미터 
$$
U = [u_1,u_2,....,u_C]:F_{tr}
$$
을 통과한 결과물
$$
u_c = u_c * X = \sum_{x=1}^{C^\prime}v^S_c * X^S
$$

$$
vc=[v^1_c,v^2_c,...,v^{C′}_c]
$$


 는 2D spatial kernel 
$$
X=[x^s,x^2,...,x^{C′}]:
$$


처음엔 저도 수식보면서 천천히 살펴봤는데 결국 결론은 그냥 convolution 연산입니다. 논문의 수식을 그대로 옮기긴 했지만 수식이 복잡하신 분들은 쉽게 위에 그림을 일반적인 convolution 연산을 생각하시면 될 것 같습니다.

### Squeeze: Global Information Embedding

Squeeze operation은 말 그대로 짜내는 연산을 하게됩니다. 이는 다르게 말해서 각 채널들의 중요한 정보만 추출해서 가져가겠다는 뜻과 같습니다. Local receptive field가 매우 작은 네트워크 하위 부분에서는 이렇게 중요 정보 추출이라는 개념이 매우 중요합니다.

논문에서는 중요 정보를 추출하는 가장 일반적인 방법론 중 하나인 GAP(Global Average Pooling)을 사용합니다. GAP를 사용하면 global spatial information을 channel descriptor로 압축시킬 수 있다고 합니다. 수식으로 나타내면 아래와 같습니다.
$$
z_c=F_{sq}(u_c)=1/H×W∑i=1H∑j=1Wuc(i,j)zc=Fsq(uc)=1H×W∑i=1H∑j=1Wuc(i,j)
$$


수식을 쉽게 설명하자면 HH, WW, CC 크기의 피쳐맵들을 1, 1, CC크기로 만든 것입니다. zczc는 채널 CC 의 원소 중 하나가 됩니다. Fig 1의 squeeze 부분과 같이 보시면 이해가 더 쉽습니다. 논문의 저자들은 간단한 방식으로 정보 압축을 위해서 GAP를 사용했지만, 다른 방법론을 사용할 수 있다고 언급하고 있습니다.

### Excitation: Adaptive Recalibration

이제 중요한 정보들을 압축(Squeeze)했다면 재조정(Recalibration)을 해야 할 차례가 왔습니다. 이 과정을 Excitation operation이라고 하며, 채널 간 의존성(channel-wise dependencies)을 계산하게 됩니다. 논문에서는 Fully connected layer와 비선형 함수를 조절하는 것으로 간단하게 이를 계산합니다.

s=Fex(z,W)=σ(W2δ(W1z))s=Fex(z,W)=σ(W2δ(W1z))

δδ는 우리가 잘 알고있는 ReLU 함수이고 σσ는 시그모이드 함수입니다. W1W1과 W2W2는 각각 Fully connected layer를 뜻합니다. 여기서 주목해야 할 점은 reduction ratio rr 을 통해서 W1W1의 노드 수를 줄인다는 것입니다. 그리고 W2W2에서 다시 피쳐맵의 수 CC만큼 증가 시킵니다. 그림으로 간단히 그려보면 아래와 같습니다. 만약 피쳐맵의 수가 CC개(그림에선 8개)이고 rr값이 4라면 아래 그림처럼 FC layer가 생기게 됩니다.

[![Imgur](https://i.imgur.com/sMoKNuy.png)](https://i.imgur.com/sMoKNuy.png)

이렇게 모든 함수를 거쳐서 나온 값을 가지고 아래 수식처럼 기존 네트워크에 있던 GAP 이전의 CC개의 피쳐맵에 각각 곱해줍니다.

x̃ c=Fscale(uc,sc)=sc⋅ucx~c=Fscale(uc,sc)=sc⋅uc

X̃ =[x̃ 1,x̃ 2,...,x̃ C]X~=[x~1,x~2,...,x~C]

Fscale|(uc,sc)Fscale|(uc,sc)는 channel-wise multiplication이고 X̃ X~는 squeeze operation 이전 HH,WW,CC크기의 피쳐맵입니다. 결국 excitation operation을 거친 스케일 값 ss들이 모두 0과 1사이의 값을 가지기 때문에 채널들의 중요도에 따라 스케일 됩니다.

### Exemplars: SE-Inception and SE-ResNet

실제 적용 예시입니다. 논문 저자들은 VGGNet, GoogLeNet, ResNet에 SE block을 적용시켰습니다. VGGNet처럼 skip connection 등이 없는 통짜(?) 네트워크들은 바로 적용했고, Inception module과 skip connection이 있는 네트워크들은 아래와 같이 적용했습니다.

[![Imgur](https://i.imgur.com/ufAxbPN.png)](https://i.imgur.com/ufAxbPN.png)

코드로 나타내면 아래와 같습니다. [출처 : taki’s github](https://github.com/taki0112/SENet-Tensorflow)

```
def Squeeze_excitation_layer(self, input_x, out_dim, ratio, layer_name):
    with tf.name_scope(layer_name) :
        squeeze = Global_Average_Pooling(input_x)

        excitation = Fully_connected(squeeze, units=out_dim / ratio, layer_name=layer_name+'_fully_connected1')
        excitation = Relu(excitation)
        excitation = Fully_connected(excitation, units=out_dim, layer_name=layer_name+'_fully_connected2')
        excitation = Sigmoid(excitation)

        excitation = tf.reshape(excitation, [-1,1,1,out_dim])

        scale = input_x * excitation

        return scale
```

## Model and Computational Complexity

네트워크에 SE block을 적용한다면, 당연히 모델 복잡도와 퍼포먼스의 trade-off가 있기 마련입니다. 모델이 무거워지면 무거울수록 계산복잡도는 올라가고 대신 그만큼의 성능 향상을 기대할 수 있습니다. 논문에서는 SE block을 ResNet-50에 적용한 SE-ResNet-50 네트워크를 이용하여 계산 복잡도를 측정하였습니다. 기존 ResNet-50이 ~3.86 GFLOPs가 필요했다면 SE-ResNet-50은 ~3.87 GFLOPs가 필요했다고 합니다. 이렇게 0.26%밖에 증가하지 않았으면서도 모델 에러는 상당히 많이 떨어뜨렸습니다. 이는 squeeze, excitation, channel-wise scaling 모두 계산복잡도가 높지 않기 때문입니다.

SE block에서 계산복잡도에 영향을 미치는 하나의 변수는 reduction ratio rr입니다. 먼저 SE block을 적용했을 때 추가되는 파라미터의 수는 아래 수식과 같습니다.

2r∑s=1SNs⋅C2s2r∑s=1SNs⋅Cs2

S는 스테이지의 수, CsCs는 output channel의 차원, NsNs는 스테이지 SS에서 반복되는 블록의 갯수입니다. 논문에서는 SE block을 ResNet-50에 적용한 SE-ResNet-50 네트워크를 이용하여 rr값을 바꿔가면서 실험을 진행하였습니다. Table 7이 그 결과물이고 rr값을 16으로 셋팅했을 때 에러가 크게 줄어들거나 늘지 않는 최적의 성능을 보여주었다고 합니다. 2.5million개의 파라미터만 추가로 들어가는데 이는 약 10%정도만 증가한 수치입니다. 마지막 레이어의 SE block은 제거해도 성능에 큰 차이가 없었다고 합니다.

[![img](https://i.imgur.com/8Tr3YRa.png)](https://imgur.com/8Tr3YRa)

## Implementation

실험 셋팅은 다음과 같습니다.

- ImageNet data
- Random-size cropping to 224 x 224
- Random horizontal lflipping
- Normalize input image through mean channel subtraction
- Data balancing for mini batch sampling
- Momentum 0.9, batch size 1024
- Learning rate 0.6 -> x10 every 30 epochs
- 100 epochs
- Testing : center crop evaluation

[![Imgur](https://i.imgur.com/0630d2v.png)](https://i.imgur.com/0630d2v.png)

Table 2는 SE-ResNet의 구조와 모델 정확도를 비교한 수치입니다. 계산복잡도는 크게 증가하지 않으면서도 좋은 성능이 나오는 것을 확인할 수 있습니다. 여기서 주목할만한 점으로는 SE-ResNet-50이 single-crop top-5 validation error를 6.62% 기록했다는 것입니다. 이는 당연히 ResNet-50 (7.48%)를 뛰어넘은 수치이고 ResNet-101(6.52%)만큼 매우 깊은 신경망에 근사한 수치입니다. VGG-16, Inception처럼 non-residual 네트워크에 대한 결과도 마찬가지로 높은 좋아진 것을 확인할 수 있습니다.

[![Imgur](https://i.imgur.com/uR9YceG.png)](https://i.imgur.com/uR9YceG.png)

MobileNet과 ShuffleNet에도 실험을 진행하여 일반적인 네트워크 구조와는 조금 다른 네트워크 구조에도 잘 적용되는것을 증명하였습니다.

[![Imgur](https://i.imgur.com/pPPXfyS.png)](https://i.imgur.com/pPPXfyS.png)

또한 Scene classification 분야에서는 Places365 dataset을 이용하여 실험을 진행하였습니다. 역시나 좋은 성능을 보여줍니다. 마찬가지로 Object detection의 유명한 데이터셋인 COCO 데이터셋을 이용해서 실험ㅇ르 진행하여도 좋은 성능을 보여주고 있습니다. Faster R-CNN을 사용했고 역시나 ResNet-50 보다 좋은 성능을 보이고 있습니다.

### The role of excitation

Reduction ratio에 대한 최적 파라미터 탐색 실험은 Table 7을 통해서 진행하였습니다. 그렇다면 excitation operation이 정말로 feature recalibration을 제대로 하고있는지에 대한 실험은 어떻게 진행했을까요? 먼저 아래와 같은 4가지 클래스를 정해주었습니다.

[![img](https://i.imgur.com/oaUmTmG.png)](https://imgur.com/oaUmTmG)

그리고 위 클래스를 네트워크에 통과시켰을 때 excitation의 결과물인 각 채널 별 스케일 값 ss의 분포를 레이어마다 아래와 같이 그려보았습니다. SE_2_3은 2단계 stage의 3번 블럭에 해당하는 값들입니다.

[![Imgur](https://i.imgur.com/TxLUc34.png)](https://i.imgur.com/TxLUc34.png)

여기서 내린 결론은 다음과 같습니다.

- 네트워크 시작 부분일수록 거의 비슷한 분포를 따릅니다. 클래스가 어떻든 간에 excitation operation은 똑같은 스케일을 곱해줍니다. 이는 초반에는 중요한 feature channel들이 공유가 된다는 말과 같습니다.
- 네트워크의 깊이가 깊어질수록 분포가 달라지기 시작합니다. 클래스 별 분포가 다르면 클래스마다 피쳐의 특징적인 부분들을 전부 다르게 전달을 한다는 말입니다. 이는 SE block이 feature extraction, feature specialization을 잘하고 있다는 뜻입니다.
- 네트워크 마지막 부분(SE_5_2, SE_5_3)에서는 클래스 별 거의 똑같은 분포를 보여줍니다. 물론 SE_5_3에서의 값은 조금 다르긴 하지만 사실 클래스가 달라도 유사한 분포를 보여주면 recalibration을 제대로 하지 못한다고 볼 수 있습니다. 따라서 파라미터 축소를 위해서는 마지막 레이어의 SE block은 제거하는게 더 낫습니다.

## Conclusion

오랜만에 읽은 네트워크 구조 논문인데 굉장히 재밌게 읽었습니다. Squeeze와 Excitation이라는 연산작업으로 정보의 압축 & 재조정을 효율적으로 실시할 수 있게 하는 아이디어가 핵심입니다. 적은 연산량으로도 성능 향상을 상당히 기대할 수 있고 가장 좋은건 기존 네트워크에 바로 붙이기만 하면 적용이 가능하다는 것입니다.

> Reference
>
> - Hu, J., Shen, L., & Sun, G. (2017). Squeeze-and-excitation networks. arXiv preprint arXiv:1709.01507.
> - taki0112’s github : <https://github.com/taki0112/SENet-Tensorflow>