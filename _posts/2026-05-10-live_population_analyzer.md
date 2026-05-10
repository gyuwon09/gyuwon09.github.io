---
title: "YOLO기반 실시간 혼잡도 분석기"
date: 2026-05-10 22:22:34 +0900
categories: [program]
tags: [program, yolo, python, opencv, ai, school, app]
---

Opencv 및 NumPy와 YOLO 모델을 활용하여 제작한 혼잡도 분석 프로그램입니다.<br>

## 문제 인식
신갈고등학교의 약 880명의 학생들은 점심시간 급식을 먹기위해 급식실에 줄을 섭니다.
다만, 급식실은 수직적인 구조의 입구를 가지고 있어, 급식지도를 하는 학생회 학생들이 어려움을 겪고 있었습니다.

이러한 환경을 개편하기 위해 급식실 대기줄을 수치화하는 소프트웨어를 개발하게 되었습니다.

카메라와 노트북 및 라즈베리파이등을 활용하여 실제 급식실 환경에 설치할 예정입니다.

레포지토리 위치 : <a href="https://github.com/gyuwon09/live-population-analyzer">gyuwon09/live-population-analyzer</a>

## 기술 스택
- **Language**: Python
- **Computer Vision**: OpenCV
- **Object Detection & Tracking**: YOLOv8, ByteTrack
- **Detection Visualization**: Supervision
- **Numerical Processing**: NumPy

## 구현
YOLOv8과 ByteTrack을 활용하여 영상 내 사람 객체를 실시간으로 탐지하고 추적하는 인원 분석 시스템입니다.

비디오 프레임에서 사람 객체만 검출한 뒤, ROI(관심 영역) 내부의 인원 수를 계산하여 현재 공간의 혼잡도를 분석하도록 구현했습니다. 
또한 객체별 Tracking ID를 유지하여 동일 인물을 지속적으로 추적할 수 있도록 구성했습니다.

OpenCV를 이용해 Detection 결과와 상태 정보를 시각화했으며, FPS 측정을 통해 실시간 처리 성능도 함께 확인할 수 있도록 구현했습니다.
