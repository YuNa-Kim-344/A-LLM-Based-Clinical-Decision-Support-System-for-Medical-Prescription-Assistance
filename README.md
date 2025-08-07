# A-LLM-Based-Clinical-Decision-Support-System-for-Medical-Prescription-Assistance

해당 연구는 2025 전자공학회 하계학술대회에서 포스터 발표되었으며, 자세한 내용은 첨부된 논문을 통해 확인하실 수 있습니다.

---

### < Abstract >

  This study proposes an LLLM-based prescription support system that recommends appropriate drug lists based on patient diagnoses to assist clinicians amid the ongoing shortage of medical staff. By applying FA-LoRA, we reduced the number of trainable parameters by approximately 93.5% while maintaining performance, and enhanced efficiency through FlashAttention. Using Llama3.1-8B-Instruct in a single-node setup, we evaluated the model via human assessment and GPT-4, confirming its accuracy and appropriateness. Future work includes extending the system with federated learning and differential privacy to protect sensitive medical data.
 
---

### 연구 개요
본 연구는 의료진 부족 문제에 도움이 되고자 환자 개인의 정보와 진단에 따른 적절한 약물 목록을 추천하는 LLM 기반 처방 지원 시스템을 제안한다. 
- 모델: Llama3.1-8B-Instruct
- 효율화: FA-LoRA로 파라미터 수 약 93.5% 감소 + FlashAttention 적용
- 학습 환경: RTX 6000 Ada 48GB, LR=1e-3, Batch Size=4, LoRA Rank=16/32


### 평가 방식
모델 평가를 위해 Human Evaluation과 LLM Evaluation 두가지를 정확도와 적합성으로 병행하여 보다 객관적이고 효율적인 평가를 진행한다.

정확도는 높은 객관성을 보일 수 있으나, 시간이 오래 걸리며, 의료 전문가가 참여해야 하는 경우 비용적인 문제가 발생한다. 또한 적절한 약물을 제시했더라도 정답 데이터셋에 없는 약물일 경우 부정확하게 평가되어 정확도가 과소평가 되는 한계가 존재한다. 이러한 한계를 보완하고자 기존 평가 방식에 LLM Evaluation 방식을 병행한다.

LLM Evaluation에 활용할 LLM으로는 Hugging Face 에서 제공하는 의료 질문 답변 Leaderboard에서 약 83%로 높은 성능을 보인 GPT-4를[12] 활용한다. 또한, 현진 의사들과 동일한 환자 사례를 기반으로 진단을 수행한 연구에서, 진단의 정확도와 근거 제시 측면에서 의사보다 높은 점수를 기록하는 사례가 존재하여, 최종적으로 GPT-4를 선정하였다.

- 정확도 (Human Evaluation) : 정답 약물과 일치하는 비율
- 적합성 (LLM Evaluation) : GPT-4가 판단한 진단에 적절한 약물 비율

### 실험 

|  rank  | steps | 출력률 | 정확도 | 적합성 |
|:------:|:-----:|:------:|:------:|:------:|
|  16    | 1950  |  60%   | 13.0%  | 81.1%  |
|        | 2925  |  45%   | 22.9%  | 79.9%  |
|        | 3900  |  55%   | 19.7%  | 81.1%  |
|        | 4870  |  55%   | 15.7%  | 85.5%  |
|  32    | 1950  |  55%   | 20.7%  | 98.2%  |
|        | 2925  |  55%   | 18.5%  | 87.5%  |
|        | 3900  |  50%   | 17.4%  | 85.1%  |
|        | 4870  |  55%   | 21.6%  | 91.0%  |

실험 결과, rank값에 관계없이 출력률은 약 55%로 응답을 출력하였다. 정확도 측면에서는 rank16 에서 steps 2925일 때가 가장 높은 정확도를 보였지만, 평균적으로는 rank32에서 더 높은 정확도가 나타났다. 이는 rank32가 전체적으로 더 안정적이고 좋은 성능을 보이고 있음을 의미하며, 적합성 역시 rank 32에서 더 높은 값을 기록했다.
정확도와 적합성 간의 차이가 상당히 두드러진 진다. 이는 동일 성분의 다양한 약물을 추천하는 경향이 있으나, 정답 데이터셋은 하나의 약물만 처방 했기 때문이다. 예를 들어, 통증을 완화시키기 위한 동일한 성분의 약물을 여러 개 추천하는 경우가 다수 존재한다. 하지만 실제 정답 데이터셋에는 그 중 하나의 약물만 처방되었기 때문에, 모델이 추천한 여러 약물 중 하나만 정답으로 간주되며, 이로 인해 정확도가 낮게 측정되는 경향이 있다.
또한, 현재 사용 중인 Llama3.1-8B-Instruct 모델이 의료 지식에 대한 사전 학습이 어느 정도 되어 있어, 학습되지 않은 약물임에도 진단에 적합하다고 판단되면 이를 추천하는 경향이 있다고 판단된다. 이는 모델이 실제로 진단에 적절한 약물을 추천하더라도, 정답 데이터셋에는 포함되지 않게 때문에 정확도가 낮게 나오는 원인 중 하나이다.
결론적으로 정확도도 중요한 부분이지만 본 프로젝트에서 적합성이 필요한 이유는, 모델이 의료진을 대체하는 것이 아닌 보조적인 역할을 한다는 점이다. 모델이 제공하는 진단에 적합성 높은 약물 추천은 의료진이 최종 결정을 내리기 위한 참고자료로 활용될 수 있다. 따라서, 모델이 제공하는 약물이 진단에 적합하지 않다면 그 추천이 무효화되겠지만, 모델이 제공하는 약물이 적합성 기준에서 좋은 평가를 받았다면, 의료진이 그 추천을 참고하여 더 나은 판단을 내릴 수 있도록 돕는 보조적인 역할을 하게 될 것으로 기대한다.

### 향후 계획
- 연합학습(Federated Learning) 구조로 확장하여 다기관에 적용하여 데이터셋을 공유하지 않고도 학습가능.
- 차등프라이버시 등의 적용으로 의료 데이터셋의 개인정보 보호 강화.
- 실제 의료 현장에서 의료진에게 신뢰 가능한 AI 보조 시스템 구축을 목표로 한다.
