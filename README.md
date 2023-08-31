# Object-detection

이전에 했던 image-classification 대회 도중에, 성능이 생각보다도 안나와서 이것저것 방법을 찾아보던 중 이미지 분류가 어떻게 발전되어 왔는지에 대한 글을 접하게 된 적이 있었다. 가장 간단한게 CNN기반 이미지 분류였고, 그 다음에 객체 탐지라고 하는 기술이 뒤이어 생겨났다고 한다. 이전에 하던 대회에서는 사용할 수 없는 방법이긴 했지만, 나중에 공부하고 싶다는 생각이 들었었고, 오늘부터 공부를 해보기로 하였다. 아마 당분간은 이론에 대해서 먼저 공부를 하고, 그 뒤에 이론을 기반으로 한 실습을 할 것 같다.
* 06.24: 오늘은 selective search를 통해 후보 박스들을 구한 뒤, 이 후보 박스들을 실제 박스와의 비교를 통하여 IOU를 구하는 방법에 대한 이론 및 실습을 하였다. 

* 07.07: 여행을 다녀와서 학습을 하지 못하였다. 오늘부터 다시 object detection에 대해 학습을 해야겠다.
* 07.10: object detection 실습을 하기 이전에, 이론부터 먼저 공부하기로 하였다. 가장 기본적인 R-CNN 모델에 대하여 학습을 하였는데, 확실히 초반 모델이라 그런지 효율이 매우 떨어지고 느릴 것 같다는 생각이 들었다. 구현 코드를 자세히 들여다보니, feature extraction, SVM (이 모델은 특이하게도 딥러닝 사이에 머신러닝을 끼워넣은 형태였다!) , bounding box regression의 학습에 이용되는 데이터들의 정의가 서로 제각각이라 학습 시키는 것도 매우 오래 걸릴 뿐더러, 이렇게되면 나중에 최적화 할때 더욱 번거로워 지는 단점도 있다. R-CNN의 가장 큰 단점이라한다면 무엇보다 이미지에서 selective search를 통해 2000개의 region을 제공 받은 뒤, 이 2000개의 region에 대하여 각각 CNN 모델을 통하여 feature extraction을 하게 되는데, 여기서 일단 정말 매우 느리고, 또 2000개의 region의 크기는 제각각인 반면, CNN에서의 마지막 층인 fully connected layer (fc layer)에 들어가는 input size는 항상 고정되어 있어야하기 때문에, 2000개의 region들을 강제로 모두 같은 사이즈로 만들어 주게 된다. 이러면 원본 이미지에서 변형되었기 때문에, 성능도 안 좋아질 수 있다는 생각이 들었다. 그래도 이 R-CNN 모델을 통해, 어떻게 단순히 이미지에서 특징맵을 추출을 통해 분류하는 것에서, 특정 오브젝트의 위치를 지정하는 것이 가능해졌는지 그 토대를 알 수 있게 되었다.
* 07.11: 어제 R-CNN에 이어서, 오늘은 Overfeat 모델에 대하여 학습하였다. 일단 이 모델은 R-CNN에서 이어지는 fast R-CNN, faster R-CNN과는 약간 다른 구조인 것으로 알고 있다. 하지만, 이 모델에서의 시도 방법이 후의 SSD, YOLO등에 영향을 주었다고 하기에, 학습 해보기로 하였다. 이 Overfeat에서 가장 다른 부분이 있다면 바로 CNN에서의 마지막 층인 fc layer을 conv layer로 대체하였다는 것이다. fc layer의 경우 input size가 항상 고정적이여야 하기 때문에 region proposal 들을 전부 같은 사이즈로 변형해야 한다는 단점이 있었고, 이를 해결하기 위해 일반 conv layer로 변경한 것이다. 이렇게 되면 input size에 관계 없이 모델에 넣는게 가능해지는 대신, 피쳐맵  역시 사이즈가 제각각이게 되지만 공간적인 정보를 담고 있게 된다. 그 뒤에 특수한 풀링을 통해 9개의 spacial output을 생성하게 되고, 이 뒤의 절차는 R-CNN 과 유사하게 bounding box regressor을 통해 영역을 산출한다. 

* 07.13: R-CNN에서의 가장 큰 문제점인 region proposal 각각 CNN 모델에 학습시켜야 한다는 과도한 연산량 문제의 해결책을 제시한 SPP-NET에 대해 학습하였다. SPP-NET은 나중에 학습하게될 fast R-CNN처럼 이미지 전체에 대하여 딱 한번만 CNN 네트워크를 통과하여 연산량을 줄이고, 이 CNN을 통하여 나온 feature map에 region proposal에서 제안된 부분만을 잘라내고, Spatial Pyramid Pooling이라는 과정을 거쳐 고정된 크기의 벡터로 변환하여 fc layer에 입력한다. 이 SPP-NET으로 연산량은 매우 줄었을 것으로 보이지만, 여전히 마지막 최종 분류단계에서는 SVM을 이용하기에 학습도 번거로울 뿐더러 최적화도 매우 어렵게 되는 문제는 여전히 그대로 남아있었다.

* 07.14: fast R-CNN은 SPP-NET처럼 전체 이미지를 CNN 네트워크를 통해 feature map을 추출하고, SPP와 유사한 방법으로 풀링을 통해 고정된 크기의 feature vector을 생성한다.  그 뒤는 R-CNN과는 다르게, SVM대신 softmax를 사용해 분류한다. 그 뒤, bounding box regressor을 통해 영역을 산출한다. fast R-CNN의 장점이라 한다면 SPP-NET과 마찬가지로 연산량이 줄었을 뿐더러, 머신러닝 기반인 SVM을 사용했기 때문에 최적화가 복잡하다는 단점을 딥러닝 기반의 softmax로 바꿈으로써 feature 추출, classification, bounding box regressor을 동시에 최적화가 가능해졌다는 것이다. 다만, 여전히 region proposal은 selective search를 사용하기 때문에, 후보 영역에 대해서 추출하는 것은 별개로 해주어야 한다는 단점이 여전하다.
* 07.15: faster R-CNN을 학습하였는데, 드디어 selective search 기반의 독립적인 region proposal 절차를 벗어나, 딥러닝 기반의 Region Proposal Network (RPN)을 사용함으로써 맨 처음부터 끝까지 하나의 네트워크에서 모든 알고리즘이 동작하는 구조로 진화하였다.
* 07.18: R-CNN부터 faster R-CNN까지 이어지는, object가 있을 법한 위치의 후보(proposals) 들을 뽑아내는 단계와 이후 실제로 object가 있는지를 Classification과 정확한 바운딩 박스를 구하는 Regression을 수행하는 단계가 분리되어 있는 two-stage detector에 대한 학습을 마쳤다. 객체의 검출과 분류, 그리고 바운딩 박스 regression을 한 번에 하는 one-stage detector의 시초격인 YOLO-V1에 대하여 학습을 하였다. YOLO-V1은 이미지를 지정한 grid로 나누고, 각 grid cell이 한번에 bounding box와 class 정보라는 2가지 정답을 도출하도록 만들어, localization과 classification을 하나의 문제로 정의하여 동시에 모든 task가 수행되도록 설계되었다. 7x7개의 각 grid cell마다 30개의 결과값이 나오는데, 20가지 클래스에 대한 확률과 바운딩 박스 2개에 대한 각각의 confidence score과 b-box좌표가 산출된다. (20+ (1+4)*2).
YOLO-V1은 faster-RCNN보다 훨씬 더 빠르지만, detection 성능이 그렇게 좋지 못하였다. 각 grid cell마다 class를 구분하는 방식이기 때문에, 한 grid cell에 여러개의 class가 동시에 존재하는 경우 정확하게 작동되지 않기 때문이다.
* 07.20: OpenCV를 이용하여 faster-RCNN의 모델을 불러와 이미지와 영상 속에 있는 class들을 분류하는 실습을 하였다. 불러온 모델은 COCO 데이터셋으로 학습을 하였으며, 80가지의 class에 대하여 faster-RCNN (Back-Bone : Resnet50) 을 통하여 학습이 되어있는 상태이다. 
* 07.22: Pytorch 기반의 Object Detection 오픈소스 라이브러리인 MMDetection을 사용해보았다. Config기반의 딥러닝 프레임워크는 처음 사용해보는지라, 아직 config가 어떻게 이루어져 있는지에 대해서만 간략하게 학습하였다.
* 07.24: config기반의 프레임워크를 사용하기 전에, 먼저 one-stage detector에 대해서 좀 더 자세히 학습을 하기로 하였다. 이전에 학습하였던 YOLO-V1에 대해 좀 더 자세하게 학습하였다.
* 07.25: 단순히 다른 사람들이 짜놓은 모델들을 가져오는 것이 아닌, 직접 모델을 짜보고 싶다는 생각이 들어 YOLO-V1을 직접 해보기로 하였다. 오늘은 YOLO-V1에서 darknet을 이용하여 feature map을 추출하는 코드를 작성하였는데, 이 darknet도 직접 구현하였다.
* 07.26: YOLO-V1의 LOSS를 구하는 부분을 구현해 보았는데, loss가 크게 3부분으로 나뉘어 있어 상당히 복잡했다. 그래도 이전에 논문을 먼저 학습했었기에, 이해하는 것에는 큰 문제가 있지는 않았다.
* 07.29: YOLO-V1의 데이터셋을 받고 형식을 설정하는 dataset을 구현하였다. 이 부분에서 가장 번거로웠던 것이 좌표 설정이였다. YOLO에서는 다른 모델들 (x,y,w,h), (x1,x2,y1,y2)과는 다르게 (centerX,centerY,w,h)로 bounding box 설정을 해야하기 때문에, 변환하는 과정이 번거로웠다. 그리고 utils에 있는 내용들은 (NMS, IOU 등)은 이전에 이미 했었기에, 파일을 불러왔다. 그러고 전체 파일을 실행해본 결과, 아무 이상없이 돌아가는 것을 확인했고, confidence score가 0.9를 넘어갈 때 모델을 자동으로 세이브 하도록 설정을 해 놓았다.
* 08.02: YOLO-V1의 구현을 완료했으니, 다음 단계로 넘어가 보기로 하였다. YOLO-V1의 다음에 나온 one-stage detector 모델인 SSD (Single Shot Detector)에 대하셔 학습을 해 보았는데, YOLO-V1에서 가장 큰 문제점이라고 생각하였던 고정된 피쳐맵 크기 (7x7)에서는 다양한 크기의 오브젝트를 탐지하는 것이 어렵다는 점을 보완한 모델이다. 이전에 학습하였던 FPN처럼 (정확히는 FPN과는 다르다. FPN은 각 서로 다른 크기의 feature map에 upsampling과 element-wise addition을 통하여 합쳐서 사용하지만, SSD에서는 크기가 다른 feature맵을 각자 conv연산에 통과시킨다.) , 이미지 분류 모델 (VGGNET)을 통과할때마다 생기는 크기가 서로 다른 feature맵들마다 3x3 conv연산을 적용하여 YOLO-V1처럼 class일 확률들과 bounding box의 좌표를 구한다. 여기서 의문이였던 점은, YOLO-V1에서는 클래스가 20개이므로 class confidence를 20개로 나누어 구하였는데, SSD에서는 배경일 확률도 포함하여 21개로 구한다는 것이다. 좀 더 자세히 알아보아야 할 것 같다.
  (가중치를 구하는 공식이 달랐기 때문에 그런 것 같다! YOLO-V1을 구현할 때의 LOSS 함수를 생각해보면, 물체가 없이 배경인 경우가 대부분이므로, 배경이 loss에 영향을 덜 미치도록 가중치를 0.5로 설정해주었다)
* 08.03: 2 stage-detector중 하나인 R-FCN에 대하여 학습을 하였다. 전체적으로 faster R-CNN의 단점들을 보완하고, 학습 속도와 정확도를 늘린 모델이라는 느낌이 강하였다. 이 모델을 학습하면서 배운 가장 중요한 점중 하나는, Translation invariance  Dilemma이다. 단순히 이미지 분류를 할 경우를 생각해본다면, 같은 물체라도 이미지 변형 같은 이유로 인하여 위치가 변경되더라도 동일한 물체이므로 출력값이 같게 나와야 (Translation invariance) 좋은 모델이다. 하지만 물체의 위치를 파악하는 것이 목표인 object detection 모델의 경우에는 같은 물체라도 다른 위치이므로 출력값이 다르게 나와야 (Translation Variance) 좋은 모델이라고 할 수 있다. 기존 faster R-CNN의 경우 피쳐맵 추출을 위한 backbone network로 VGG16을 detection 모델로 넘기게 되는데, backbone을 통과한 피쳐맵은 위치 정보가 소실된 채로 넘어오기 때문에, 이 위치 정보가 부족한 피쳐맵을 detection에 넘기게 되면 적절하게 학습이 이루어지지 않는다. 이를 Translation invariance dilemma라고 하며, R-FCN은 이 딜레마를 해결하기 위하여, RPN을 통하여 넘어온 ROI에 위치 정보를 encode하기 위해 ROI를 3x3 grid로 나누어준다. (약간 Overfeat 모델이 연상되었다.)
* 08.04: YOLO-V2에 대하여 학습을 하였다. YOLO-V1에 batch normalization을 추가하였고, YOLO-V1에서 문제점 중 하나인 초기 bounding box의 좌표 (0~1사이 랜덤으로 정함)로부터 실제 좌표를 처음부터 예측해 나가야한다는 점을 개선하기 위하여 faster-RCNN에서 쓰였던 anchor box를 도입하였다. 하지만 단순히 9개의 정해진 비율의 anchor box를 도입하는 것이 아니라, k-means clustering을 통하여 적절한 수의 k를 찾아 k개의 앵커박스를 사용하였다. (논문상 5개). 또 특이한 점은, 최종 feature맵을 산출하는 과정이다. 이전에 SSD를 학습할 때 보았던 것처럼, 최종 피쳐맵의 크기가 고정(7x7)된 상황에서는 다양한 크기의 물체를 탐지하는 것이 어렵다. YOLO-V2에서는 입력 이미지의 크기를 2배로 늘렸기 때문에 (448x448, 하지만 최종 feature map을 홀수로 나오게 하기 위하여 416x416을 인풋으로 사용), 원래 YOLO-V1에서 사용한 Darknet을 그대로 사용할 경우 13x13의 피쳐맵이 나오게 되지만, 마지막 Pooling을 수행하기 전에 26x26x512의 feature map을 4개로 분할하여 (13x13x512 4개) 결합하여 13x13x2048 사이즈의 feature map을 얻고 원래의 결과인 13x13x1028인 피쳐맵과 합쳐 13x13x3072 크기의 feature map을 생성한다. 이렇게 되면 feature map을 분할하는 과정을 통하여 보다 작은 객체에 대한 정보를 함축하게 된다. 최종적으로는 conv layer를 통과하여 13x13x125 (13x13x 5*(20+5))의 feature map을 생성하게 된다.
* 08.06: RetinaNet에 대하여 학습을 하였다. RetinaNet 학습 과정중 가장 반가웠던 것은 Focal Loss였다. 이전에 데이콘의 결함 이미지 분류 대회에서 이미지별로 불균형이 매우 심해 f1-score가 매우 안좋게 나오는 점을 해결하기 위하여 방법을 찾던 중, Focal Loss를 사용하면 성능이 더 높아진다는 글을 보았다. 그때 당시 이 focal loss를 weighted loss function을 약간 변형한 정도라고 이해하고 넘어갔었다. 이미지에서 물체가 차지하는 비중은 매우 적으므로, 대부분은 배경일 것이다. 배경이 대부분을 차지하므로 배경을 분류하는 것은 매우 쉬울 것이고 이러한 easy example이 많으면 기존의 Loss인 Cross Entropy Loss의 경우 이 easy example들로 인한 loss들이 매우 많이 더해지게 되어 결국 보기 드문 class를 압도해버려 제대로 작동하지 않게 된다. 이를 해결하기 위해 기존 cross entropy에 가중치를 추가하는데, 확률이 높은 쪽의 loss를 훨씬 더 작게하도록 하여 easy example들이 전체 loss에서 차지하는 비율을 낮춘다. 이전 대회에서 이 focal loss를 사용시 성능이 향상되었던 것도, 각 결함별로 이미지 개수의 차이가 매우 심해서 (가장 많은 이미지는 1400개, 가장 적은 이미지는 3개) 분류가 쉬운 결함의 loss가 대부분을 차지해 버리기 때문에 가장 적은 결함에 대한 loss가 무시되는 결과를 방지할 수 있었기 때문인 것 같다.
* 08.08: YOLO-V3에 대한 학습 및 OPENCV를 이용하여 모델을 불러온 뒤 적용을 해 보았다. 지난번에 YOLO-V1을 직접 구현할 때는 YOLO의 모델, 손실함수 등등을 직접 구현하였고, 이를 학습하는데 매우 오랜 시간이 걸렸었기에 이번에는 모델과 가중치를 외부에서 받아오기로 하였다. (YOLO-V3도 나중에 직접 구현해 볼 생각이다) YOLO-V3는 YOLO-V2까지와는 다르게 여러 사이즈의 feature map을 이용한다. 앞선 SSD에서 사용되었던 방법과 똑같다. feature map 사이즈는 13x13,26x26,52x52이고, 각 그리드별로 3개의 bounding box가 있다. 각 bounding box에는 좌표(4) + confidence score(1) + class score(80) 총 85개의 값이 들어있다. 이 뒤는 다른 모델들처럼 NMS를 통하여 겹치는 box들을 제거한다.
* 08.10: Mask R-CNN에 대해 학습하려는 도중, faster R-CNN에 대해 기억이 잘 나지 않는 것을 반성하며 다시 학습하고 이번에는 직접 구현된 코드들을 리뷰해보기로 하였다. 이전에도 느꼈던 것이지만 2 stage-detector는 RPN등 region proposal network와, classifier에 쓰이는 데이터의 비율을 각각 다르게 정해줘야 하기 때문에, 매우 번거로운 작업처럼 느껴졌다. 확실히 코드를 직접 들여다보니 좀 더 확실하고 자세하게 알 수 있게 되었다는 생각이 들었다. 이전에는 어느정도로만 이해하고 넘어갔던 과정들도 이제는 확실히 이해를 한 것 같아 보람이 있게 느껴졌다.
* 08.12: faster R-CNN에 대하여 자세하게 이해를 하였으니 다시 Mask R-CNN 학습을 하였다. Mask R-CNN에서 특징적인 부분중 하나는 ROI-Pooling 대신에 ROI-Align을 사용한다는 점이였다. 이전에 ROI-Pooling에 대해 자세하게 알고싶어 해외 사이트를 찾아보던 중에, ROI-Pooling에 대해서 상세하게 적어놓은 것과 동시에 ROI-Align이라는 것에 대해서도 쓰여있었다. 그때 당시는 ROI-Pooling에 대해서만 이해하고 넘어갔지만, 이제는 ROI-Align에 대해서도 들여다 보았다.
ROI Pooling의 가장 큰 문제점은 이미지의 부분 손실이 많다는 점이다. 먼저 RPN을 통해 얻은 region proposal을 feature map에 투영할때도 비율로 나눌때 발생하는 소수점으로 인하여 소수점을 제거하는 과정에서 기존 region proposal보다 더 작은 크기의 feature map을 얻게 되고, 또한 이 feature map을 또 고정된 크기 (faster R-CNN의 경우 7x7)로 맞춰줄때 또 다시 소수점으로 인한 손실이 발생하게 된다. 물론 이러한 손실이 매우 크다고 하면 그건 아니기 때문에 기존 object detection 모델 성능에는 크게 영향을 미치지 못한다. 하지만 Mask-RCNN의 경우 이미지 내에 존재하는 모든 객체를 탐지하는 동시에 각각의 경우(instance)를 정확하게 픽셀 단위로 분류하는 task인 instance segmentation task에서 주로 사용되는 모델이기에 영향이 크므로 ROI-Align이라는 새로운 방식을 적용한 것이다. ROI-Align은 소수점을 버리지 않고 정확하게 투영시킨 뒤, 일정한 크기의 feature map으로 변환시킬 때에도 각 셀이 차지하는 비율을 소수점까지 정확하게 이용하여 ROI Pooling을 진행한다. 또한 Mask-RCNN은 다양한 크기의 feature map을 사용하기 위해 FPN까지 이용하였다. faster-RCNN에 매우 여러가지 기능을 덧붙였기에 성능은 비교가 안될정도로 좋아졌지만, faster-RCNN도 느린편에 속하는 모델이였는데 Mask-RCNN은 더 느려졌기 때문에 실제 생활속 사용 용도로는 적합하지 않다는 생각이 들었다.
* 08.14: 가장 보편적인 모델인 YOLO-V3의 pretrained된 모델을 이용하여 inference를 해 보았다. 추후에는 직접 object detect를 해보고 싶은 이미지들을 직접 데이터셋을 만들어 해볼 예정이다.
* 08.16: EfficientDet모델을 학습하기로 하였는데, 그 전에 baseline인 EfficientNet에 대하여 학습을 하기로 하였다. 이전에 데이콘 이미지 분류 대회에 참가를 해 보았을 때, 그때 당시의 baseline 모델이 efficientnet-b0 였었고 이 모델을 그대로 사용하였던 것이 기억이 나는데 그때는 Resnet과 같은 다른 pretrained 모델들과의 차이점은 모른 채 그냥 사용했었다. 이번에 학습을 하고 난 뒤에는 명확하게 다른 모델이라는 것을 느꼈다.
이미지의 특징을 추출하기 위한 Conv-layer은 성능을 향상시키기 위하여 여러가지 방법을 사용하였다. 깊이를 늘리거나 (Depth Scaling. Resnet에서 Resnet 50과 Resnet 101처럼 깊이를 늘린 것), 각 layer에 있는 unit의 수를 늘리거나 (Width Scaling. GoogleNet부터 등장한 1x1 convolution을 통해 파라미터 수도 줄이고 Width도 늘려 비선형성을 증가시켰다), 네트워크의 입력으로 받는 이미지의 해상도, 즉 input size를 늘렸다 (Resolution Scaling).  EfficientNet은 가장 최적의 with/depth/resolution 조합을 찾아 (Compound Scaling) 더욱 빠르면서도 정확도가 높아지게 하였다.
이러한 EfficientNet을 BackBone으로 사용하는 EfficientDet은 Neck에 기존의 FPN대신 더 발전시킨 BiFPN을 사용한다. 기존의 FPN은 한방향(Top down)으로만 정보의 흐름이 제한되었다면, Bi-FPN은 Bottom up Path도 추가하고 불필요한 Node들을 제거하여 속도를 높이고, 서로 다른 resolution의 feature map끼리의 연산 시 가중치를 주어 정확도도 높인다.
* 08.17: EfficientNet의 Bi-FPN에 대하여 한가지 의문이 생겨서 조사를 해 보았다. FPN보다도 훨씬 단계가 깊어지고 복잡해진 것처럼 보이는데, 아무리 불필요한 노드들을 약간씩 제거해 주었다고는 해도 결과 표를 보니 연산량이 훨씬 적게 나온 것이다. 이유를 찾아보니 이 Bi-FPN 연산을 할 때 Depth-Wise Convolution을 이용했다고 한다. 이전에 MobileNet의 논문을 지나가듯이 본 적 있었는데, 그때 본 용어인 것 같아 다시 자세하게 보기로 하였다.
* 08.18: MobileNet의 주요 맹점은 Conv-Net 학습시의 파라미터를 줄이는 것이다. Resnet에서 나온 것처럼 모델을 깊이 할수록 한계는 있지만 성능이 좋아지게 되지만, 반대로 연산량은 기하급수적으로 늘어나게 되고, 이를 해결하기 위한 새로운 방법이 Depth-Wise Convolution이다. 기존의 CNN을 생각해보면 Input의 Channel수가 필터의 높이가 되어, KxKxC 사이즈의 필터를 Output의 사이즈 (M)으로 적용하는 방식이였다면, Depth-Wise Convolution은 KxK 사이즈의 필터를 적용하여 C개의 feature map을 생성한 뒤, 1x1 Convolution을 이용하여 (1x1xCxM) M개로 feature map을 늘리는 방식을 이용한다. 이렇게 되면 같은 결과값이 나오지만 (중간에 1x1 Conv를 이용하였기에 비선형성이 더 증가했을지도..?) 연산량은 매우 줄어드는 결과가 나오게 된다. (K^2*C*M vs K^2*+C*M). 이걸 이용하였기 때문에 EfficientNet에서 Bi-FPN을 사용했을 때 FPN보다도 연산량이 줄어드는 신기한 결과가 나왔던 것이다.
* 08.20: 한가지 EfficientDet에 대하여 오해를 하였다. EfficientDet의 Compound Scaling이란 EfficientNet에서 사용한 Compound Scaling을 시사하는줄 알았고, EfficientDet은 이 EfficientNet을 BackBone으로 사용한 Object Detection Model이라고 생각했었다. 하지만 자세히 학습을 해 보니, EfficientDet은 또 다른 Compound Scaling을 하고 있었다. Backbone Network, Bi-FPN Network, Prediction Network, Image Resolution 총 4가지의 Scale에 대하여 Compound Scaling을 통해 적합한 조합을 찾는 모델이였다.
* 08.22: EfficientDet을 Tensorflow Hub를 통해 불러온 뒤, Inference를 해 보았다. 이전 모델들부터 같은 이미지와 영상을 inference 해보는 중인데, 모델이 좋아질수록 성능이 확연히 좋아지는게 느껴진다. 
예비군(28~31)으로 인하여 당분간 정지일듯 하다...
복귀
