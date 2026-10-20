## 각 지표

AI를 개발하며, 다양한 지표와 자료를 직접 눈으로 확인해야 한다.
이번 프로젝트를 진행하면서 어떤 지표를 보면 될까?
먼저 각 지표의 의미에 대해 알아보자.

- Precision (정밀도) : 모델이 답한 것 중에서 실제 정답 데이터에 대한 비율
- Recall (재현율) : 전체 정답 중 올바르게 예측한 데이터 비율
- Precision - Recall (PR) 그래프
    ⇒ Precision과 Recall 관계를 그래프로 나타낸 것이다. 이는 threshold에 dependent하므로 trade off 관계에 있다.
    ⇒ AUPRC는 그래프 아래 면적을 의미한다. 그래프가 크면 예측률이 높고, 놓치는 오류도 없다는 뜻이다.

- AUROC (Area Under the ROC Curve)
    <details>
    <summary>ROC Curve</summary>

    거짓 양성률 vs 진짜 양성률을 그린 그래프이다.

    - 완벽 모델 : 1.0
    - 부족 모델 : 0.5

    ⇒ 숫자가 높음에 따라 구별을 잘 함

    </details>

    - AUPRC (Area Under the Precision-Recall Curve)
        - 예측된 문제 중에 진짜 문제가 있는 비율이 중요한 경우
            - 문제가 드문 경우에 중요한 지표이다
