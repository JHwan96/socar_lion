# 쏘카x멋직 부트캠프

# 팀명 : high go
# 주제 : 타다 데이터와 공공데이터를 활용한 타다 ATA 정확도 높이는 모델

## 1. 아이디어 주제 및 배경

- 타다 데이터와 공공 데이터를 활용한 타다 도착시간 예측 정확도 높이는 모델
- 기존 주제는 '타다 데이터와 공공 데이터를 활용한 교통량 예측 모델' 이였으나,
- 모델링 후 최종 예측 결과가 0.0001 ~ 0.0003 구간의 MSE 값 도출.
- label leak 를 의심하여 데이터 전처리를 다시 진행 후, 데이터 훈련을 했지만, 정확도가 터무니없이 높은 예측 결과 도출.
  - 문제점
   1. label을 CI(Congestion Index) 활용
    실제 이동거리와 평균속도, 여유로울때의 속도의 데이터가 없어서 유추한 값으로 계산함으로써, 정확한 CI값을 못구함.
   2. 본 해커톤에서 활용한 평균속도와 여유로울때의 속도는 외국 데이터 기반의 근사값을 활용. 

- 위와 같은 문제와, 정확도가 너무 뛰어난 MSE값 도출로 인하여, 주제변경
- '공공데이터를 활용한 타다 ATA 값의 정확도 높이는 모델' 로 주제 변경


## 2. 데이터
### 2-1. 수집 데이터
 - 쏘카 제공 타다 ATA 데이터
 - 일별 차량통행 속도 및 날씨, 최고온도, 최저온도
  https://topis.seoul.go.kr/refRoom/openRefRoom_1_3.do
 - 2019년 휴일 정보
  https://www.data.go.kr/data/15012690/openapi.do
 - 일별 시도, 시군구별 교통사고
  http://taas.koroad.or.kr/sta/acs/exs/typical.do?menuId=WEB_KMP_OVT_UAS_PDS
 - 자치구 단위 서울 생활 인구(일별 기준)
  https://www.data.go.kr/data/15083611/fileData.do
  
### 2-2. 데이터 전처리
 - 수집한 데이터들을 KEY 값에 매핑
 - 결측지 제거 : feature값을 0으로 변환 후 실제 데이터와 비교했을대, 큰 왜곡이 없어 결측값 행 삭제
 - 범주형 데이터를 구간 나눔
 - feature engineering : api-eta 와 속도를 활용하여 CI 근사값 도출
 - 이상치 제거 : box plot를 활용하여 이상치 제거
 - standard scailing

## 3. 활용 모델
### 3-1. 머신러닝
  - 교통사고 건수 데이터 활용한 RandomForestRegressor 모델  
    GridSearch 통한 hyperparameter parameter가 max_depth=6, max_features=13, n_estimators=1000 일 때,   
    MSE: 8.3938, RMSE: 2.8972로 가장 성능이 좋았다.   
   
  - 교통사고 확률 데이터 활용한 RandomForestRegressor 모델
    GridSearch 통한 hyperparameter 기본 모델이 hyperparameter 한 모델 보다 성능이 더 좋았다.
    MSE: 8.3981, RMSE: 2.8980
 
 => 결론 : 교통사고 건수와 교통사고 확률 데이터를 RandomForestRegressor에 학습 시켰을 때 성능면에서는 큰 차이가 없었다.
           feature importance를 보면 두 데이터 모두 api_eta의 중요도가 가장 높았고,
           그 다음으로 pickup 위치의 위도, 경도와 driver 위치의 위도, 경도 순이었다.
 
 ### 3-2. DNN regressor model 이용한 ATA예측
  * 머신러닝의 결과를 봤을 때 교통사고 건수와 교통사고 확률 데이터의 모델 간 성능 차이가 거의 없었기 때문에 교통사고 건수 데이터를 기반으로 딥러닝 모델을 생성.
  - 모델1. linear model로 3 layers.
    activation function은 relu, optimizer는 RMSprop, batch 하나당 10개 샘플 사용, 총 epoch 수 1000개, 성능평가로 mse와 mae 선택.
    결과: 테스트 셋 성능 mae: 2.1333 - mse: 9.1143
          train data의 경우 mse 값 줄더드는 반면에 validation data의 경우 오히려 증가하는 추세를 보여 좋은 모델은 아니라고 판단됨.
  
  - 모델2. 첫번째 모델과 마찬가지로 교통사고건수 데이터를 사용
    train과 test 8:2로 분리
    dnn regressor model 4 layers.
    activation function은 relu, input shape은 train 데이터 셋의 column 수 15개, batch size는 32개, epoch 수 200, learning_rate는 0.001 지정, optimizer는 Adam, loss 평가는 mse 선택
    결과: train의 경우 loss가 감소하는 추세인 반면에 validation의 경우 loss가 작게 감소, test set loss: 16.4813
    
 ### 3-3. K-Fold
  - 교통사고 건수 데이터로 k-fold 진행  
    성능 평가 지표 rmse
    사용한 모델: Linear Regression, Decision Tree Regressor, Random Forest Regressor, GradientBoostingRegressor, HistGradientBoostingRegressor
    가장 성능이 좋았던 모델은 GradientBoostingRegressor, rmse 값 0.89
  
  - 교통사고 확률 데이터로 k-fold 진행  
    평가지표 rmse
    사용한 모델: Linear Regression, Decision Tree Regressor, Random Forest Regressor, GradientBoostingRegressor, HistGradientBoostingRegressor
    가장 성능이 좋았던 모델은 GradientBoostingRegressor, rmse 값 0.89
    hyperparameter tuning해도 최저 rmse 값은 0.89
    
  => 결론: 교통사고 건수 데이터를 포함한 경우, 최고 성능을 보인 모델은 GradientBoostingRegressor였고 rmse 값 2.89.
           교통사고 확률 데이터를 포함한 경우, 최고 성능을 보인 모델은 동일하게 GradientBoostingRegressor였고 rmse 값 2.89.
           교통사고 건수 데이터와 교통사고 확률 모두 GradientBoostingRegressor가 rmse 값 2.89로 제일 좋은 성능을 보였음.

## 4. 참고 논문
 - Traffic congestion prediction based on Estimated Time of Arrival
  https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0238200
 - Prediction of Traffic Congestion in Seoul by Deep Neural Network
  http://journal.kits.or.kr/journal/article.php?code=68020
