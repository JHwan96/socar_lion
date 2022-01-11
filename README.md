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
 - 1
 - DNN

## 4. 성능평가
 - MSE 활용


## 참고 논문
 - Traffic congestion prediction based on Estimated Time of Arrival
  https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0238200
 - Prediction of Traffic Congestion in Seoul by Deep Neural Network
  http://journal.kits.or.kr/journal/article.php?code=68020
