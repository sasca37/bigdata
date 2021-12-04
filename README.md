빅데이터분석기사 실기 준비
- 데이터 수집 작업
- 데이터 전처리 작업
- 데이터모형 구축작업
- 데이터모형 평가작업

library 모음
-caret : 혼동행렬 (confusionMatrix)
- rm(list = ls()) : 현재 R Studio에 기록된 데이터 모두 삭제

## 진흥원 예시 
```R
# 출력을 원할 경우 print() 함수 활용
# 예시) print(df.head())

# setwd(), getwd() 등 작업 폴더 설정 불필요
# 파일 경로 상 내부 드라이브 경로(C: 등) 접근 불가

X_test = read.csv('data/X_test.csv') 
X_train = read.csv('data/X_train.csv') 
y_train = read.csv('data/y_train.csv')

# 사용자 코딩
set.seed(1234)
summary(X_test)  # 환불금액 결측값 존재 
str(X_test) 
summary(X_train) 
summary(y_train)
str(y_train)

X_test$환불금액 <- ifelse(is.na(X_test$환불금액), 0, X_test$환불금액)
X_train$환불금액 <- ifelse(is.na(X_train$환불금액), 0, X_test$환불금액)

# Scaling
library(caret)
model <- preProcess(X_train[,-1], method='range')
x_t <- predict(model, X_train)
x_v <- predict(model, X_test)

# 로지스틱 
lg_model <- glm(y_train$gender ~.-cust_id, binomial, x_t)
lg_pred <- predict(lg_model, x_t, type='response')
lg_pred2 <- ifelse(lg_pred>=0.5, 1,0)
print(head(lg_pred2))
caret::confusionMatrix(as.factor(lg_pred2), as.factor(y_train$gender)) #0.6714 
library(ModelMetrics)
ModelMetrics::auc(y_train$gender, lg_pred) #0.6942 

#svm 
library(e1071)
svm_model <- svm(y_train$gender~.-cust_id, x_t)
svm_pred <- predict(svm_model, x_t, type='response')
ModelMetrics::auc(y_train$gender, svm_pred) #0.6936 
svm_pred2 <- ifelse(svm_pred>=0.5, 1,0)
caret::confusionMatrix(as.factor(svm_pred2), as.factor(y_train$gender)) #0.6409

#randomForest 
library(randomForest)
rf_model <- randomForest(y_train$gender~.-cust_id, x_t, ntree=300)
rf_pred <- predict(rf_model, x_t, type='response')
ModelMetrics::auc(y_train$gender, rf_pred) #0.9999
rf_pred2<- ifelse(rf_pred>=0.5, 1,0)
caret::confusionMatrix(as.factor(rf_pred2), as.factor(y_train$gender)) #0.9986

# 랜덤포레스트 선정 
result <- as.data.frame(cbind(x_v$cust_id, rf_pred))
names(result) <- c('cust_id', 'gender')
write.csv(result, '123451231.csv', row.names=F)
# 답안 제출 참고
# 아래 코드 변수명과 수험번호를 개인별로 변경하여 활용
# write.csv(변수명,'003000000.csv',row.names=F) 


```
