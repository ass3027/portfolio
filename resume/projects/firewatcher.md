# FireWatcher 산불감지 시스템

- 기간: 2024.12 ~ 2025.08
- 역할: AI 데이터 연계 및 영상 처리
- 직급: 연구원

<a id="video-reliability"></a>

## CCTV 영상 장애 대응 및 출력 안정성 개선

13개 지역에서 지역별 100대 이상의 CCTV를 운영하는 산불감지 시스템에서 영상 끊김이 발생했지만, 관측 체계가 없어 장애 확인과 조치가 지연됐습니다. DeepStream 오류 로그를 자동으로 수집하고 FastAPI로 영상 테스트 시스템을 구축했으며, FFmpeg를 최적화하고 송출 프로세스에 자동 재시작과 로깅을 적용했습니다. 영상 장애 확인과 조치에 걸린 시간을 80% 줄였고, 영상 출력 안정성을 85%에서 93%로 높였습니다.

<a id="metadata-streaming"></a>

## 실시간 분석 모듈 및 메타데이터 전송 구조 개선

산불감지 시스템의 DeepStream 분석 모듈이 단일 파일과 전역 변수에 의존해 변경이 어려웠습니다. 분석 로직을 클래스 단위로 분리하고 Kafka와 Avro 기반의 Producer 및 Consumer 모듈을 구현해 실시간 메타데이터 전송 구조를 재사용할 수 있게 했습니다.

## 기술 스택

Python, FastAPI, Kafka, Apache, Avro, FFmpeg, DeepStream
