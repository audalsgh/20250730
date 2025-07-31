# 28일차

## 1. NVIDIA api 키 발급후, 교수님의 peoplenet코드를 runpod에서 돌려보기
api키는 한번만 보여준다. 따로 복사해둘것.<br>
<img width="516" height="493" alt="image" src="https://github.com/user-attachments/assets/dd2bed39-66c9-4edf-b9f0-02be06d558fb" /><br>
runpod에서 pod를 생성한 후, 주피터 랩이 아닌 터미널 창을 열어 !명령어들을 실행해야 입력가능.<br>
주피터 랩 셀에선 입력이 안되는듯..<br>
<img width="2502" height="988" alt="image" src="https://github.com/user-attachments/assets/c5e0d56e-3168-401c-8cec-50802831f11b" /><br>
코드에서 api키, json파일, 0757485240724879, no-team, no-ace 선택하도록 되있어서 입력함.<br>
<img width="757" height="823" alt="image" src="https://github.com/user-attachments/assets/2bec9b5c-64a1-4bba-ba41-6eefe19d008a" /><br>
!명령어는 터미널에서 했고, import부분부터 실행해본 모습.<br>
결과영상을 다운받는 코드는 gpt를 통해 셀을 하나 더 추가했다.
<img width="1188" height="796" alt="image" src="https://github.com/user-attachments/assets/61a24f4c-fd68-4a4f-88a4-83f3b5351ad2" /><br>
결과영상이 1초도 안되게 짧은 이유는, 100프레임씩 건너뛰는 샘플링을 했기때문.

## 2. 앙상블 방식으로 (피플넷) + 트래픽넷 + YOLOv11 통합해보기
어제 피플넷까지만 진행해보았으니, 오늘은 트래픽넷과 YOLOv11까지 통합해보자.<br>
peoplenet 모델 파일을 직접 코랩에 업로드하는 코드를 사용했었는데, 구글 드라이브와 현재 리포지터리에 백업해두었다.<br>

**앙상블(ensemble)이란, 여러 개의 서로 다른 모델을 함께 사용해서 단일 모델보다 더 견고하고 정확한 예측을 얻는 기법.<br>
특히 객체 검출(Object Detection)에 쓰임**<br>

장점
- 모델 간 강·약점을 보완
- 드물게 검출되던 객체도 더 안정적으로 잡아냄
- 전체 성능(정확도·재현율) 향상

단점
- 추론 속도가 느려짐 (모델 수만큼 연산하므로)
- 구현 복잡도 상승

| 모델             | 주요 용도                  |
| -------------- | ---------------------- |
| **PeopleNet**  | 사람 검출에 특화              |
| **TrafficNet** | 교통 표지 · 신호 · 차량 검출에 특화 |
| **YOLOv11**    | 다양한 일반 객체 검출           |

우선순위 : PeopleNet→TrafficNet→YOLO 순. (사람이 제일 중요, 그다음이 신호 및 차량, 마지막이 일반 객체들)<br>
-> 우선순위가 높은 모델의 예측을 먼저 통과시키고 겹치는 박스를 제거<br>

세 모델(PeopleNet, TrafficNet, YOLOv11)각각의 검출 결과(박스, 클래스, confidence)를 모두 모아서, 이를 하나의 최종 결과로 합치는 단계가 필요.

## 코랩에서 피플넷 이후 부분을 만들고, 실행해보는 과정.
[코드 정리](0730_Peoplenet+Trafficnet+YOLOv11_ensemble.ipynb)

<img width="635" height="211" alt="image" src="https://github.com/user-attachments/assets/2a6f3d56-7a87-414d-b51d-89cc848a7890" />

Jetson‐Inference 저장소에서 제공하는 TrafficCamNet ONNX 파일들은 CPU/GPU(Colab)에서 바로 사용 가능한 순수 ONNX 형태가 아님.<br>
(실제로 내려받아보면 그래프 노드가 0이거나, TensorRT 전용 플러그인이 포함되있음.)<br>
따라서 Colab 상에서 “TrafficNet ONNX” 를 그대로 돌리는 건 현실적으로 불가능에 가깝고, 대신 ultralytics YOLO를 이용한 “교통 객체 필터링” 방식을 권장.

<img width="1289" height="719" alt="image" src="https://github.com/user-attachments/assets/e93a4f14-4e9a-4dcc-ad06-27aa48c9961c" /><br>
**결과영상 -> 피플넷 + YOLO 앙상블로 사람뿐만 아니라 일반적인 객체도 감지하는 모습!**
