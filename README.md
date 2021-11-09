# 정보보호 Readme

## HTTP request 로그들을 기계학습을 통해 학습시켜서 정상 로그와 비정상 로그를 분류하는 알고리즘

### 개발 환경

---

- Google colab - Python 3

## 프로젝트 설명

### parsing 함수

---

- HTTP request 로그는 GET method, POST method, PUT method로 나눌 수 있다.
    - GET method는 url 부분에 query 메시지가 들어가고 POST나 PUT method일 경우엔 query에 들어갈 내용이 payload에 들어가게 된다.
        
    ![image 8U27B1](https://user-images.githubusercontent.com/54922639/140898685-2e9fea5d-0dce-47f8-9985-df1b3595405b.png)

        
- 신민식, 권태경 저자의 "정상 사용자로 위장한 웹 공격 탐지 목적의 사용자 행위 분석 기법" 논문에 따르면 HTTP request 로그의 6개 영역인 url, host, Content-Length, Content-Type, Cookie, payload에서 url, payload, host 정보 외에는 공격자가 위장할 수 있는 영역이다. 따라서 이 세 개의 영역 중 유의미한 값을 갖는 url과 payload를 호출해서 진행했다.
- url에서 다시 path와 query로 나누고 query 안에서도 & 구분자를 통해 하나하나의 query를 space로 구분해 추출했다. 또한 payload도 query와 같은 형식으로 하나하나의 query를 추출했다.
- url path와 query, payload 추출시 필요하지 않은 문자가 들어가지 않도록 문자열 슬라이싱을 통해 method 타입을 나누어 진행했다.
- 좀 더 자세한 정보는 코드의 주석을 참고.

### vectorize 함수

---

- `tf = TfidfVectorizer(min_df=0.0, analyzer="char", sublinear_tf=True, ngram_range=(3,3))`
    - min_df: 최소 빈도값을 설정하는 파라미터이다.
    - analyzer: 학습단위를 결정하는 파라미터이다. 각각 word, char의 두가지 옵션이 있다
    - sublinear_tf: TF(Term- Frequency,단어빈도) 값의 스무딩 여부를 결정하는 파라미터이다. outliers(이상치)를 완만하게 처리해주는 역할을 한다.
    - ngram_range: 단어의 묶음을 정하는 파라미터이다. ngram_range=(1,1)은 단어의 묶음을 1개부터 1개까지 설정하라는 의미이므로 단어사전에는 1개짜리 단어묶음만 존재하게된다. 따라서  ngram_range =(3,3)은 3개짜리의 단어묶음만 존재하게된다.
- analyzer에 word, char과 ngram_range에 (1,1), (3,3), (5,5)를 여러 조합으로 시도해본 결과 char, (3,3)이 근소한 차이로 좀 더 빠르고 정확도 높다고 여겨져 char과 (3,3)을 사용하기로 결정했다.

### train 함수 ****

---

- **Random Forest**: ensemble machine learning 모델의 한 종류로 여러 개의 decision tree 부분집합을 형성(Bagging)하여  overfitting을 예방하는 머신러닝 모델이다.
- 여러 개의 Training Data를 생성해 각 데이터마다 개별 Decision Tree를 구축→(Bagging)
- 랜덤 포레스트는 텍스트 데이터 같이 매우 차원이 높고 희소한 데이터에는 잘 작동이 안된다. 또한 랜덤 포레스트는 선형 모델보다 많은 메모리를 사용하며 훈련과 예측이 느려진다.
- 부트스트랩을 사용하는데 부트스트랩이란 여러개의 데이터 세트를 중첩되게 분리하는 것이다
- 부트스트랩은 통계학에서 여러개의 작은 데이터 세트를 임의로 만들어 개별 평균의 분포도를 측정하는 것이 목적이다. 따라서 이진 분류를 할 때는 비슷비슷한 트리로만 나오니 다양하지도 않고 랜덤 포레스트의 장점을 살리기 힘들다.

![Untitled](https://user-images.githubusercontent.com/54922639/140898902-6d6a5bce-fdb9-4b86-860f-087d952fbf82.png)


Source: [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2015/06/tuning-random-forest-model/)

- **Linear SVC(Linear Support Vector Classifier)**: 데이터 집합이 주어졌을 때 분류하는 비확률적 이진 선형분류모델로 만들어진 분류 모델은 경계로 표현되는데 SVM알고리즘은 그 중 가장 큰 폭을 가진 경계를 찾는 알고리즘이다.
- SVM은 데이터가 P차원에서 주어졌을때 이 데이터를 P-1차원의 초평면(hyperplane)으로 분류해 준다. 이 때 데이터를 분류할 수 있는 여러 초평면 중 각 클래스의 데이터 점들 간의 거리가 가장 먼 초평면을 선정하는데 이 초평면을 최대-마진 초평면(maximum-margin hyperplane)이라고 하고 이 때 사용된 선형 분류기를 최대-마진 분류기(maximum margin classifier)라고 한다.
- 초평면에서 마진(margin)은 각 서포트 벡터와 초평면 사이의 거리를 의미하는데 이 때 초평면의 법선 벡터 $w$에 대해서 $w⋅x-b=0$을 만족하는 점 $x$의 집합이 된다. 서포트 벡터 머신은 초평면 내에서 마진의 최대값을 구해준다.
    
    ![Untitled 1](https://user-images.githubusercontent.com/54922639/140899083-6db8d86b-5c0c-4c90-991c-3978de685ab6.png)


    

---

### Random Forest가 아닌 Linear SVM을 사용한 이유

- Linear SVM가 이진 분류 모델이기 때문에 주어진 데이터셋인 CSIC를 분류하기에 더 적합하다고 판단해서 기본 base code에서 주어진 Random Forest대신 Linear SVM를 사용하게 되었다.

### 결과

---

- Accuracy : 약 98~99 %
- F1 score :  약 98~99 %
- train set에서 고유한 단어가 약 3만 개가 나오고 로그 하나당 약 2천 ~ 약 3만 차원이 나온다.

기술하지 않은 dataset, test 함수는 Base code와 동일합니다.
