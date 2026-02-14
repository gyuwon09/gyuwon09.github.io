---
title: "청각 장애인을 위한 실시간 수업 자막 어플리케이션 '소리결(so-rit-gyul)'"
date: 2025-06-19 22:56:10
categories: [program]
tag: [program]
---

실시간 음성 자막 변환 및 수업 내용 요약 기능을 제공하는 교육 보조 프로그램<br>
Python + Flet 기반 Windows 데스크톱 애플리케이션

<a href="https://www.youtube.com/watch?v=6jKHWHUV3lI">인터페이스 시연 (Youtube)</a><br>
<a href="https://www.youtube.com/watch?v=Fvd0NEH8fKI">프로토타입 시연 (Youtube)</a>

## 프로젝트 개요

‘소리결’은 청각장애 학생의 수업 참여도를 높이기 위해 개발한 실시간 교육 보조 프로그램입니다.  
수업 중 발생하는 음성을 즉시 텍스트 자막으로 변환하고, 수업 종료 후 핵심 내용을 요약하여 학습 복습을 지원합니다.

Windows 환경에서 실행 가능하도록 <b>Python</b>과 <b>Flet</b> 라이브러리를 활용해 데스크톱 애플리케이션 형태로 구현했습니다.

## 기술 스택

- **Language**: Python 3.11
- **UI Framework**: Flet  
- **Speech-to-Text**: Google Cloud Speech-to-Text API  
- **Text Post-processing & Summarization**: OpenAI API  
- **Platform**: Windows Desktop  

## 핵심 기능
프로젝트의 핵심 기능은 <b>실시간 음성 인식(STT)</b>입니다.

초기 기획 단계에서 OpenAI의 Whisper 모델 도입을 검토하였으나,
구현 당시 해당 모델의 실시간 처리 기능의 한계가 존재하였습니다.<br>
따라서 해당 문제를 해결하기 위해 대체 모델로<br>
스트리밍 기반 처리가 가능하고 안정적인 성능을 제공하는 **Google Speech-to-Text API**를 최종 선택하였습니다.

다만 **Google Speech-to-Text API**의 성능이 Whisper모델에 비해 빈약한 점을 파악하였으며,<br>
이를 보완하기 위해 OpenAI API를 활용한 후처리 단계를 추가하여<br>
문맥 기반 오인식 문장 교정 및 반복 문장 제거 기능을 구현하였으며,<br>
더 나아가 수업 내용을 요약하는 기능까지 추가할 수 있었습니다.

## 마무리
이번 프로젝트를 통해
청각장애 학생의 학습 접근성이라는 사회 문제를 개선하며,
Python 기반 데스크톱 UI(Flet) 구현 경험 및 외부 API 연동 및 후처리 로직 설계를 경험하며
상용 AI 모델을 사용한 GUI 프로그램 개발의 전반적인 과정을 이해하고 개발자로써의 역량을 증진시킬 수 있었습니다.