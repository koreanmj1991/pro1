# LinePlus 사전과제 분석

데이터 전처리
데이터 전처리 과정에서 userId와 movieId를 각각 0부터 시작하는 수열에 대응시키는 딕셔너리로  만들었습니다. 이를 통해 분석에 쓰일 (행이 userId, 열이 movieId인) 행렬을 만드는데 필요한 과정을 간소화하였습니다. 모델 훈련에서는 임의로 생성된 인덱스를 사용하고 추후 테스트 데이터셋을 이용하여 에측 평점을 도출할 때 이 딕셔너리를 이용하여 다시 원래 userId와 movieId로 맵핑합니다.

모델 구현
주어진 논문에 기재된 모형을 MatrixFactorization이라는 모듈에 구현하였습니다. 절편항인 총평균평점, 각 유저와 영화에 각각해당하는 bias를 추가한 목적함수를 Stochastic Gradient Descent를 이용하여 최소화하였습니다. SGD를 이용한 optimization 과정은 해당 모듈의 gradient(), update()에 구현되어 있습니다. 모델 훈련은 모듈의 fit 함수를 이용하여 주어진 epoch에 따라 진행됩니다. 다만, 분석에 사용되는 행렬인 rating matrix R은 그 크기가 각각  데이터셋1은 (31332, 10647), 데이터셋2는 (133035, 22007)로 굉장히 큰 행렬이며, 또한 희소합니다. 따라서 매 epoch마다 target value (rating) 이 0보다 큰 값을 찾는 과정이 중복됨을 방지하기 위하여 모델 훈련 전에 0이 아닌 rating 값을 가지는 (i, j)의 위치를 미리 저장하는 과정을 추가하였습니다. 이는 모듈의 get_coordinates()에 구현되어 있습니다. 이렇게 생성된 좌표 데이터를 이용하여 모델 훈련 시 탐색 과정에 들어가는 자원을 최소화하였습니다.
모델 훈련은 데이터셋1과 데이터셋2 모두 동일한 파라미터 구성(잠재요인의 개수 f: 3, learning rate: 0.01, regularizing parmeter 람다: 0.01)으로 진행하였습니다. 그 결과, 데이터셋1은 초기 RMSE 0.928에서 0.72까지 약 22% 감소하였고, 데이터셋2는 0.905에서 0.772까지 약 15% 감소하였습니다. 각각 RMSE가 더이상 유의미하게 감소하지 않은 epoch까지 훈련을 진행하였습니다. 

Cold start problem
훈련이 된 모형을 통해 각각의 데이터 셋의 테스트 데이터를 이용하여 예측평점을 도출하였습니다. 예측평점을 계산할 때는 네 가지의 경우로 나누어 진행했습니다. 먼저, 테스트 데이터의 userId와 movieId가 훈련 데이터에도 둘 다 존재할 때는 훈련된 모형의 predict()를 이용하여 예측평점을 도출하였습니다. 어느 한 쪽이 존재하지 않는 경우에는 해당 유저 혹은 해당 영화의 평균 평점을 이용하여 그 값을 예측 평점으로 하였습니다. 둘 다 존재하지 않는 경우에는 총평점평균 mu를 그 값으로 하였습니다.

개선점
모델 훈련 시 최적의 파라미터 구성(잠재요인의 개수와 람다의 크기 등)을 교차검증을 통해 찾을 필요가 있습니다. 
Cold start problem을 해결하기 위한 계산 과정을 줄일 방안이 필요합니다. 유저 평균 평점과 영화 평균 평점을 미리 계산하여 저장해 놓는다면 예측 과정에서 소요되는 자원과 시간을 상당량 줄일 수 있습니다. 또한 단순히 평균을 예측에 사용하는 것보다 외부 데이터와 연동하여 각 유저의 장르별 펵균, 각 영화의 연령대별 평균 평점 등을 이용하는 것이 좀 더 정확한 예측 평점을 계산할 수 있을 것입니다.

