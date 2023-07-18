# Object-detection

이전에 했던 image-classification 대회 도중에, 성능이 생각보다도 안나와서 이것저것 방법을 찾아보던 중, 이미지 분류가 어떻게 발전되어 왔는지에 대한 글을 접하게 된 적이 있었다. 가장 간단한게 CNN기반 이미지 분류였고, 그 다음에 객체 탐지라고 하는 기술이 뒤이어 생겨났다고 한다. 이전에 하던 대회에서는 사용할 수 없는 방법이긴 했지만, 나중에 공부하고 싶다는 생각이 들었었고, 오늘부터 공부를 해보기로 하였다. 아마 당분간은 이론에 대해서 먼저 공부를 하고, 그 뒤에 이론을 기반으로 한 실습을 할 것 같다.
* 06.24: 오늘은 selective search를 통해 후보 박스들을 구한 뒤, 이 후보 박스들을 실제 박스와의 비교를 통하여 IOU를 구하는 방법에 대한 이론 및 실습을 하였다. 

* 07.07: 여행을 다녀와서 학습을 하지 못하였다. 오늘부터 다시 object detection에 대해 학습을 해야겠다.
* 
* 07.10: object detection 실습을 하기 이전에, 이론부터 먼저 공부하기로 하였다. 가장 기본적인 R-CNN 모델에 대하여 학습을 하였는데, 확실히 초반 모델이라 그런지 효율이 매우 떨어지고 느릴 것 같다는 생각이 들었다. 구현 코드를 자세히 들여다보니, feature extraction, SVM (이 모델은 특이하게도 딥러닝 사이에 머신러닝을 끼워넣은 형태였다!) , bounding box regression의 학습에 이용되는 데이터들의 정의가 서로 제각각이라 학습 시키는 것도 매우 오래 걸릴 뿐더러, 이렇게되면 나중에 최적화 할때 더욱 번거로워 지는 단점도 있다. R-CNN의 가장 큰 단점이라한다면 무엇보다 이미지에서 selective search를 통해 2000개의 region을 제공 받은 뒤, 이 2000개의 region에 대하여 각각 CNN 모델을 통하여 feature extraction을 하게 되는데, 여기서 일단 정말 매우 느리고, 또 2000개의 region의 크기는 제각각인 반면, CNN에서의 마지막 층인 fully connected layer (fc layer)에 들어가는 input size는 항상 고정되어 있어야하기 때문에, 2000개의 region들을 강제로 모두 같은 사이즈로 만들어 주게 된다. 이러면 원본 이미지에서 변형되었기 때문에, 성능도 안 좋아질 수 있다는 생각이 들었다. 그래도 이 R-CNN 모델을 통해, 어떻게 단순히 이미지에서 특징맵을 추출을 통해 분류하는 것에서, 특정 오브젝트의 위치를 지정하는 것이 가능해졌는지 그 토대를 알 수 있게 되었다.
* 
* 07.11: 어제 R-CNN에 이어서, 오늘은 Overfeat 모델에 대하여 학습하였다. 일단 이 모델은 R-CNN에서 이어지는 fast R-CNN, faster R-CNN과는 약간 다른 구조인 것으로 알고 있다. 하지만, 이 모델에서의 시도 방법이 후의 SSD, YOLO등에 영향을 주었다고 하기에, 학습 해보기로 하였다. 이 Overfeat에서 가장 다른 부분이 있다면 바로 CNN에서의 마지막 층인 fc layer을 conv layer로 대체하였다는 것이다. fc layer의 경우 input size가 항상 고정적이여야 하기 때문에 region proposal 들을 전부 같은 사이즈로 변형해야 한다는 단점이 있었고, 이를 해결하기 위해 일반 conv layer로 변경한 것이다. 이렇게 되면 input size에 관계 없이 모델에 넣는게 가능해지는 대신, 피쳐맵  역시 사이즈가 제각각이게 되지만 공간적인 정보를 담고 있게 된다. 그 뒤에 특수한 풀링을 통해 9개의 spacial output을 생성하게 되고, 이 뒤의 절차는 R-CNN과 유사하게 bounding box regressor을 통해 영역을 산출한다. 

* 07.13: R-CNN에서의 가장 큰 문제점인 region proposal 각각 CNN 모델에 학습시켜야 한다는 과도한 연산량 문제의 해결책을 제시한 SPP-NET에 대해 학습하였다. SPP-NET은 나중에 학습하게될 fast R-CNN처럼 이미지 전체에 대하여 딱 한번만 CNN 네트워크를 통과하여 연산량을 줄이고, 이 CNN을 통하여 나온 feature map에 region proposal에서 제안된 부분만을 잘라내고, Spatial Pyramid Pooling이라는 과정을 거쳐 고정된 크기의 벡터로 변환하여 fc layer에 입력한다. 이 SPP-NET으로 연산량은 매우 줄었을 것으로 보이지만, 여전히 마지막 최종 분류단계에서는 SVM을 이용하기에 학습도 번거로울 뿐더러 최적화도 매우 어렵게 되는 문제는 여전히 그대로 남아있었다.

* 07.14: fast R-CNN은 SPP-NET처럼 전체 이미지를 CNN 네트워크를 통해 feature map을 추출하고, SPP와 유사한 방법으로 풀링을 통해 고정된 크기의 feature vector을 생성한다.  그 뒤는 R-CNN과는 다르게, SVM대신 softmax를 사용해 분류한다. 그 뒤, bounding box regressor을 통해 영역을 산출한다. fast R-CNN의 장점이라 한다면 SPP-NET과 마찬가지로 연산량이 줄었을 뿐더러, 머신러닝 기반인 SVM을 사용했기 때문에 최적화가 복잡하다는 단점을 딥러닝 기반의 softmax로 바꿈으로써 feature 추출, classification, bounding box regressor을 동시에 최적화가 가능해졌다는 것이다. 다만, 여전히 region proposal은 selective search를 사용하기 때문에, 후보 영역에 대해서 추출하는 것은 별개로 해주어야 한다는 단점이 여전하다.
* 07.15: faster R-CNN을 학습하였는데, 드디어 selective search 기반의 독립적인 region proposal 절차를 벗어나, 딥러닝 기반의 Region Proposal Network (RPN)을 사용함으로써 맨 처음부터 끝까지 하나의 네트워크에서 모든 알고리즘이 동작하는 구조로 진화하였다.
