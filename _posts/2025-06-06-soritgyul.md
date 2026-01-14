---
title: "청각 장애인을 위한 실시간 수업 자막 어플리케이션 '소리결(so-rit-gyul)'"
date: 2025-06-19 22:56:10
categories: [program]
tag: [program]
---

2025 청소년 SW동행 해커톤으로 청각장애학생 교육 프로그램 '소리결'(프로토타입 버전)은 실시간 음성을 자막으로 변환하고, 수업 내용을 요약하는 기능을 담은 프로그램입니다. Python과 Flet 라이브러리를 활용하여 윈도우 환경에서 실행할 수 있도록 제작되었습니다.

<a href="https://www.youtube.com/watch?v=6jKHWHUV3lI">인터페이스 시연 (Youtube)</a><br>
<a href="https://www.youtube.com/watch?v=Fvd0NEH8fKI">프로토타입 시연 (Youtube)</a>

## 구현
 이 프로젝트를 진행하면서 가장 핵심적인 기능은 STT 기능이였습니다. 이 아래 소스코드에서는 구글의 STT api가 활용되었는데,
 구글의 STT api는 실시간으로 자막 작성이 가능하며, 준수한 음성 인식 성능을 보여주는 api였습니다. 개발당시 가장 높은 성능을 보여주는 openai의 whisper모델을 도입하는 것 또한 고려되었지만, 기본적으로 음성 파일만 텍스트화가 가능했으며 음성 텍스트화에 소요되는 시간이 길어 활용이 어려웠던
 한계점이 존재하여 구글 api를 활용한 후, openai api를 활용하여 잘못 인식된 문자를 보정하거나 인식된 내용을 요약하는 기능을 추가하는 방향으로 진행하게되었습니다.

## 소스코드
```
import flet as ft
from flet import *
import speech_recognition as sr
import openai
import threading
import subprocess
import tkinter as tk
from tkinter import messagebox

mic_graphic = True

def show_warning(message):
    root = tk.Tk()
    root.withdraw()  # 메인 윈도우 숨김
    root.attributes("-topmost", True)  # 항상 위로 설정

    messagebox.showwarning("경고", message, parent=root)

    root.destroy()

#UI
class MyApp:
    def __init__(self):
        self.page = None
        self.ui_text = Text(value="", color=Colors.BLACK, weight=FontWeight.BOLD,size=30)
        self.current_texts = ""
        self.lock = threading.Lock()
        API_KEY = "API_KEY(프로토타입이므로 환경변수를 사용하지 않았음.)"
        openai.api_key = API_KEY
        self.model = "gpt-3.5-turbo"
        self.voice_button_type = True
        self.voice_record = True
        self.voice_button_label = Text(value="녹음 시작하기", color=Colors.BLACK, weight=FontWeight.BOLD)
        self.listening = None
        self.func_pause = False
        self.summary_text = Text(value="", color=Colors.BLACK, weight=FontWeight.BOLD,size=22)
        self.summary_text_ui = Row(
            [
                Container(
                    bgcolor=Colors.BLUE_50,
                    margin=20,
                    padding=10,
                    expand=True,
                    content=Container(
                        content=self.summary_text,
                        padding = 10
                    )
                )
            ], expand=True)
        self.wave_img = Row(
            [
                Image(
                    "wave.png",
                    height=200,
                    fit=ImageFit.FILL,
                    expand=True,
                )
            ],
            expand=True,
            spacing=0
        )

        global mic_graphic
        if mic_graphic:
            visual_graph = threading.Thread(target=self.sub_visual_program)
            visual_graph.daemon = True
            visual_graph.start()
        else:
            print("마이크 그래픽이 비활성화 되어있습니다.")

        self.recognizer = sr.Recognizer()

        mic_names = sr.Microphone.list_microphone_names()
        print("사용 가능한 마이크 목록:")
        for i, name in enumerate(mic_names):
            print(f"{i}: {name}")

        default_mic_index = 1

        self.activate_mic = []
        try:
            self.mic = sr.Microphone(device_index=default_mic_index)
            print(f"활성화된 기본 마이크: {mic_names[default_mic_index]}")
        except OSError:
            print("⚠️ 기본 입력 장치를 감지할 수 없습니다. 시스템 설정을 확인해주세요.")
            self.mic = None
        except:
            print("마이크가 설정되지 않았습니다.")

    def voice_record_start(self,e):
        self.func_pause = True
        if self.voice_button_type:
            try:
                self.voice_button_label.value = "배경소음 측정중"
                self.page.update()
                with self.mic as source:
                    print("배경 소음 측정 중입니다. 마이크를 사용하지 마십시요")
                    self.recognizer.adjust_for_ambient_noise(source, duration=3)
            except Exception as ex:
                print(f"[오류] ambient noise 설정 실패: {ex}")
                show_warning("배경 소음 측정 중 문제가 발생했습니다.")
                self.voice_button_label.value = "녹음 시작하기"
                self.voice_button_type = True
                self.func_pause = False
                self.page.update()
                return
            self.listening = self.recognizer.listen_in_background(
                self.mic, self.callback, phrase_time_limit=15
            )
            print("\n[ 음성 인식이 시작되었습니다. ]")
            self.voice_record = True
            self.voice_button_type = False
            self.voice_button_label.value = "녹음 끝내기"
            threading.Thread(target=self.while_true, daemon=True).start()
            self.page.update()
        else:
            self.voice_button_label.value = "녹음본 보정중"
            self.page.update()
            if self.listening is not None:
                self.listening()  # 반환된 stop() 호출!
                self.listening = None
            print("[ 음성 인식이 종료되었습니다. ]\n")
            self.voice_record = False
            self.voice_button_type = True
            self.voice_button_label.value = "녹음 시작하기"
            self.text_clean()
            self.func_pause = False
            self.page.update()


    def text_clean(self):
        if not self.ui_text.value == "":
            print("\n[ 인식된 음성 정리하는 중 ]")
            filtered_text = self.gpt_request(self.ui_text.value)
            if filtered_text is not False:
                print(filtered_text)
                self.ui_text.value = filtered_text
                self.page.update()
            else:
                print("gpt api가 응답하지 않습니다. token이 정확이 입력되었는지 확인해주세요.")
                show_warning("음성 필터링 기능이 작동하지 않습니다. token값이 정확히 입력되었는지 확인하세요.")
            print("\n[ 종료됨 ]")
        else:
            show_warning("텍스트가 인식되지 않았습니다.")

    def text_summary(self,e):
        if not self.func_pause:
            self.func_pause = True
            if not self.ui_text.value == "":
                filtered_text = self.gpt_request_summary(self.ui_text.value)
                if filtered_text is not False or filtered_text == '오류가 발생했습니다':
                    print("[ 텍스트 요약됨 ]")
                    self.summary_text.value = filtered_text
                    self.page.remove(self.wave_img)
                    try:
                        self.page.remove(self.summary_text_ui)
                        self.page.add(self.summary_text_ui)
                        self.page.add(self.wave_img)
                    except:
                        self.page.add(self.summary_text_ui)
                        self.page.add(self.wave_img)
                elif filtered_text is False:
                    print("gpt api가 응답하지 않습니다. token이 정확이 입력되었는지 확인해주세요.")
                    show_warning("음성 요약기능이 작동하지 않습니다. token값이 정확히 입력되었는지 확인하세요.")
                else:
                    show_warning("음성 요약기능이 작동하지 않습니다. 분량이 너무 짧지 않은지 확인하세요")
            else:
                show_warning("요약할 텍스트가 감지되지 않았습니다.")
            self.func_pause = False
        else:
            show_warning("음성 인식이 종료된 뒤 이용해주십시요.")

    def refresh(self,e):
        if not self.func_pause:
            self.ui_text.value = ""
            self.current_texts = ""
            self.page.update()


    def main(self,page: Page):
        self.page = page
        self.page.title = "수업 듣기"
        self.page.scroll = ScrollMode.AUTO
        self.page.fonts = {
            "Pretendard": "https://github.com/orioncactus/pretendard/tree/main/packages/pretendard/dist/public/variable/PretendardVariable.ttf"
        }
        self.page.bgcolor = Colors.WHITE
        self.ui_text.font_family="Pretendard"
        self.voice_button_label.font_family = "Pretendard"
        self.summary_text.font_family="Pretendard"
        self.page.appbar = AppBar(
            leading=IconButton(Icons.ARROW_BACK_IOS_NEW_OUTLINED),
            title=Text("2025-06-15 4교시",font_family="Pretendard",weight=FontWeight.BOLD),
            bgcolor="#EBF0FF",
            surface_tint_color=None
        )
        self.page.add(
            Container(height=5)
        )
        self.page.add(
            Row(
                [
                    Container(width=20),
                    ElevatedButton(
                        width=200,
                        height=50,
                        style=ft.ButtonStyle(
                            side=ft.BorderSide(1, Colors.BLACK),
                            shape=ft.RoundedRectangleBorder(radius=10),
                        ),
                        content=Row(
                            [
                                Icon(Icons.MIC_EXTERNAL_ON_OUTLINED, color=Colors.BLACK),
                                self.voice_button_label,
                             ],
                            alignment=ft.MainAxisAlignment.CENTER,
                            vertical_alignment=ft.CrossAxisAlignment.CENTER
                        ),
                        on_click=self.voice_record_start
                    ),
                    Container(width=5),
                    ElevatedButton(
                        width=200,
                        height=50,
                        style=ft.ButtonStyle(
                            side=ft.BorderSide(1, Colors.BLACK),
                            shape=ft.RoundedRectangleBorder(radius=10),
                        ),
                        content=Row(
                            [
                                Icon(Icons.INSERT_CHART_OUTLINED, color=Colors.BLACK),
                                Text("내용 요약하기",color=Colors.BLACK,weight=FontWeight.BOLD,font_family=("Pretendard")),
                            ],
                            alignment=ft.MainAxisAlignment.CENTER,
                            vertical_alignment=ft.CrossAxisAlignment.CENTER
                        ),
                        on_click=self.text_summary
                    ),
                    Container(width=5),
                    ElevatedButton(
                        width=200,
                        height=50,
                        style=ft.ButtonStyle(
                            side=ft.BorderSide(1, Colors.BLACK),
                            shape=ft.RoundedRectangleBorder(radius=10),
                        ),
                        content=Row(
                            [
                                Icon(Icons.REFRESH_OUTLINED, color=Colors.BLACK),
                                Text("수업 초기화", color=Colors.BLACK, weight=FontWeight.BOLD, font_family=("Pretendard")),
                            ],
                            alignment=ft.MainAxisAlignment.CENTER,
                            vertical_alignment=ft.CrossAxisAlignment.CENTER
                        ),
                        on_click=self.refresh
                    ),
                ]
            )
        )

        self.page.add(
            Container(content=Row(
                [
                    Container(width=20),
                    Container(
                        height=self.page.height,
                        content=Icon(Icons.PERSON_OUTLINED, color=Colors.BLACK, size=40),
                        alignment=alignment.top_center,
                        padding=10,
                    ),
                    Container(
                        padding=10,
                        height=self.page.height,
                        expand=True,
                        content=self.ui_text,
                        alignment=alignment.top_left,
                    ),
                    Container(width=20)
                ],expand=True,)
            )
        )

        self.page.add(self.wave_img)




    #시각화 프로그램 실행
    def sub_visual_program(self):
        try:
            subprocess.run("mic.exe")
        except:
            print("Error[ 시각화 프로그램 ]")

    def gpt_request(self, query):
        try:
            messages = [{
                "role": "system",
                "content": """
너는 음성 인식 결과를 분석하고, 잘못 인식된 단어나 문장을 사람이 실제로 말했을 가능성이 높은 문장으로 복원하는 역할을 수행하는 전문가야.

입력되는 텍스트는 사람의 발화 내용을 음성 인식 엔진이 인식한 결과로, 다음과 같은 오류가 포함될 수 있어:
- 단어의 일부가 잘못 변형됨 (예: '성태' → '상태')
- 문법이 어색하거나 일부 단어가 빠짐
- 유사한 소리를 잘못 인식한 경우 (예: '공부하겠습니다' → '공부했습니다' 또는 '공부 해석니다')

너는 사용자가 실제로 말하려던 의도를 가장 자연스럽고 정확한 문장으로 추측하여 수정해야 해.

**주의사항:**
- 입력 문장을 크게 바꾸지 말고, 오직 의미 보정 및 오류 수정을 수행해.
- 말의 순서를 바꾸거나 내용을 추가하지 마.
- 한 문장씩만 수정하고, 출력에는 오직 **수정된 문장 하나만** 보여줘. 설명이나 여분의 텍스트는 포함하지 마.
- 복원이 불가능하면 "복원 실패"라고만 출력해.
"""
            }, {
                "role": "user",
                "content": query
            }]

            response = openai.chat.completions.create(model=self.model, messages=messages)
            answer = response.choices[0].message.content
            return answer
        except:
            print("Error[gpt_request]")
            return False

    def gpt_request_summary(self,query):
        try:
            messages = [{
                "role": "system",
                "content": """너는 입력되는 글을 분석하고 이론적인 내용을 중심으로 요약하는 편집자야.
        프로그램이 출력하는 글을 분석하고 글의 이론적인 부분에 문제가 되지 않도록 요약해야해. 글은 진행중인 수업 스크립트임을 고려하며 작성하도록해.
        이론과 필요없는 부분은 제거해도 좋아. 수업 스크립트인 이 글을 수업 내용 중심으로 요약하는게 핵심이야. 다만 글에 등장하지 않았던 이론이나
        개념을 추가하지는 말도록해. 이 글을 요약하는 것이 너의 역할이지, 말을 덧붙히는 것이 너의 역할이 아니야.
        수정을 완료한 뒤에는 아무런 말없이 그저 수정 완료한 글을 출력해.
        너는 만약 성공적으로 글을 수정하면 3,000달러의 팁을 받게 될거야.
        너는 출력하는 텍스트에 내가 시킨 일에 관한 텍스트 외엔 너가 하는 말을 전혀 포함시킬 수 없어.
        만약 너의 권한 밖이거나 수행할 수 없다면, '오류가 발생했습니다'를 출력해."""
            }, {
                "role": "user",
                "content": query
            }]

            response = openai.chat.completions.create(model=self.model, messages=messages)
            answer = response.choices[0].message.content
            return answer
        except:
            print("Error[gpt_request]")
            return False


    def callback(self,recognizer, audio):
        try:
            text = recognizer.recognize_google(audio, language='ko-KR')
            with self.lock:
                self.current_texts += text
                print(text)
        except sr.UnknownValueError:
            pass
        except sr.RequestError as e:
            print(f"API ERROR : {e}")
            show_warning("음성 인식 기능에 문제가 발생했습니다. google api가 정상적으로 작동하는지 확인해주세요.")


    def while_true(self):
        while self.voice_record:
            with self.lock:
                if self.current_texts != "":
                    self.ui_text.value = f"{self.ui_text.value} {self.current_texts}"
                    self.page.update()
                    self.current_texts = ""

myapp = MyApp()
app(target=myapp.main)
```