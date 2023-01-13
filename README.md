# MNIST_FCN_DNN

## 본 Repository는 패스트캠퍼스의 딥러닝 유치원 강좌를 참고로 제작되었습니다.

## 1. 문제 정의
본 Repository는 MNIST의 숫자 손글씨 데이터(0~9,28*28 이미지 데이터)를 **FCN 모델(Fully-Connected Layer Model)**에  학습시켜 숫자 손글씨 데이터가 입력으로 들어왔을 때 해당 데이터가 실제 어떤 숫자를 나타내는지를 분류하는 분류 모델을 생성합니다.
### MNIST 데이터 구성
- 학습 데이터 : 60,000개
- 테스트 데이터 : 10,000개
### 학습 데이터 Split
- Train data : Validation data = 8 : 2
- Train data = 48,000개
- Validation data = 12,000개 

## 2. 모델 구성
### 입력계층 
입력계층은 28x28 손글씨 이미지 데이터를 한줄로 펼쳐놓은 28x28=784 차원입니다.
### Block
모델의 각 은닉 계층은 Block으로 구성되어 있으며 Block 내부 구조는 다음과 같습니다.
- linear transformation : nn.Linear
- activation function : nn.LeakyReLU
- Regularization : BatchNormalization or Dropout
입력으로 모델의 계층의수(n_layers)가 들어오면 MNIST 숫자 데이터의 입력의 크기(784)에서 최종 출력계층의 크기(10) 사이 등차 간격으로 은닉 계층의 뉴런의 개수가 정해집니다. 
### 출력 계층
출력 계층은 10차원으로 0 ~ 9 각 클래스별 LogSoftmax 결과를 출력합니다.
### Loss function = NLLLoss() (Negative Log Likelihood Loss)

## 3. 코드 구성
### model.py
- Class Block 
````
class Block(nn.Module):
    def __init__(self,
                input_size,
                output_size,
                use_batch_norm=True,
                dropout_p=.4):

        self.input_size=input_size
        self.output_size=output_size
        self.use_batch_norm = use_batch_norm
        self.dropout_p=dropout_p

        super().__init__()

        def get_regularizer(use_batch_norm,size):
            return nn.BatchNorm1d(size) if use_batch_norm else nn.Dropout(dropout_p)

        self.block = nn.Sequential(
            nn.Linear(input_size,output_size),
            nn.LeakyReLU(),
            get_regularizer(use_batch_norm,output_size)
        )

    
    def forward(self,x):
        # |x| = (batch_size,input_size)
        y = self.block(x)
        # |y| = (batch_size,output_size)

        return y

````

- class ImageClassifier
````
class ImageClassifier(nn.Module):
    def __init__(self,
                input_size,
                output_size,
                hidden_sizes=[500,400,300,200,100],
                use_batch_norm=True,
                dropout_p=.3):

        super().__init__()

        assert len(hidden_sizes) > 0 , "you need to specify hidden layers"

        last_hidden_size = input_size
        blocks=[]

        for hidden_size in hidden_sizes:
            blocks+=[Block(
                last_hidden_size,
                hidden_size,
                use_batch_norm,
                dropout_p
                )]
            last_hidden_size=hidden_size

        self.layers = nn.Sequential(
            *blocks,
            nn.Linear(last_hidden_size,output_size),
            nn.LogSoftmax(dim=-1)
        )

    def forward(self,x):
        # |x| = (batch_size,input_size)
        y = self.layers(x)
        # |y| = (batch_size,output_size)
        return y
````






 