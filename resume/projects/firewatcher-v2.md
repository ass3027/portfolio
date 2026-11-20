# FireWatcher 산불감지 시스템

- 기간: 2024.12 ~ 2025.08
- 역할: AI 데이터 연계 및 영상 처리
- 직급: 연구원

<a id="video-reliability"></a>

## CCTV 영상 장애 진단과 복구 자동화

13개 지역에서 지역별 100대 이상의 CCTV를 운영하는 산불감지 시스템에 FastAPI 기반 영상 테스트 도구를 구축했습니다. DeepStream 오류 로그를 자동 수집해 개발자가 아니어도 영상과 오류를 확인할 수 있게 했습니다. FFmpeg 설정을 조정하고, 송출 프로세스에 오류 로깅과 자동 재시작을 적용했습니다.

<a id="metadata-streaming"></a>

## 실시간 영상 메타데이터 전송

산불감지 시스템의 DeepStream 분석 로직을 클래스 단위로 분리해 단일 파일과 전역 변수에 대한 의존을 줄였습니다. 영상 분석 결과를 Avro 스키마로 정의하고, 이를 Kafka로 보내고 읽는 Producer와 Consumer 모듈을 구현했습니다.

## 기술 스택

Python, FastAPI, Kafka, Avro, FFmpeg, DeepStream
