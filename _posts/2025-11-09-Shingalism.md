---
title: "동아리부스속 작은 포토부스 [ Shingalism ]"
date: 2025-11-09 10:39:13
categories: [program]
tag: [program]
---

<b>학교 축제 부스 운영을 위해 제작한 셀프 포토부스 소프트웨어</b>

'포토이즘'이라는 셀프 사진 촬영 포토 부스 브랜드의 소프트웨어를 모티브로 제작된 소프트웨어입니다.
포토이즘 연예인 프레임에서 아이디어를 얻어 교사 7분의 프레임을 제작, 적용하여 자체적으로 촬영할 수 있도록 만들었습니다. 
pygame, opencv로 주요 기능을 구현하였으며, 
google api를 활용하여 자동으로 google drive에 촬영한 이미지가 전송되도록 구현하였습니다.
또한 촬영후 프린터로 직접 프린트되는 기능도 구현하였습니다.

레포지토리 위치 : <a href="https://github.com/gyuwon09/shingalism">gyuwon09/shingalism</a>

### 마무리 요약
이번 프로젝트는 '가은제'라는 고교 축제 부스를 위해 제작되었으며,
일주일간 187회 촬영, 250명 이상 사용 및 인기 동아리상 수상이라는 유의미한 결과를 얻게되었습니다.

## 기술 스택
- **Language**: Python 3.11  
- **Multimedia Processing**: pygame, OpenCV, Pillow  
- **Cloud Integration**: Google Drive API  
- **Platform**: Windows Desktop  

## 홈
프로그램의 시작을 알리는 화면입니다. 특별한 기능은 없기때문에 넘어가겠습니다.<br>
프로그램에서는 스페이스바를 눌러 넘어갈 수 있습니다.
<img src="/assets/img/main.png" alt="홈 화면 이미지">

## 프레임 선택
사진의 테두리가 될 프레임을 선택합니다. 일반프레임과 특별히 제작한 교사 7분의 프레임을 적용하였습니다.<br>
(공유된 레포지토리에는 초상권 때문에 제외하여 업로드하였습니다.)<br>

우측 원모양 이미지를 선택할 경우 테두리가 붉게 변하며 선택되고,<br>
선택된 이미지에 따라 좌측 프레임이 변경됩니다.

<img src="/assets/img/select frame.png" alt="프레임 선택 화면 이미지">

원형 이미지는 다음과 같은 병렬 구조로 저장됩니다. 또한 이미지 파일이 존재하지 않으면 검은 색으로 대체됩니다.

```
theme_styles = {
    0: {"color": (0, 0, 0), "image": None},
    1: {"color": (255, 255, 255), "image": None},
    2: {"color": (194, 230, 255), "image": r"source\theme2.png"},
    3: {"color": (0, 0, 0), "image": r"source\theme3.png"},
    4: {"color": (0, 0, 0), "image": r"source\theme4.png"},
    5: {"color": (0, 0, 0), "image": r"source\theme5.png"},
    6: {"color": (0, 0, 0), "image": r"source\theme6.png"},
    7: {"color": (0, 0, 0), "image": r"source\theme7.png"},
    8: {"color": (0, 0, 0), "image": r"source\theme8.png"},
    9: {"color": (0, 0, 0), "image": r"source\theme9.png"},
    10: {"color": (0, 0, 0), "image": r"source\theme10.png"},
    11: {"color": (0, 0, 0), "image": r"source\theme11.png"},
    12: {"color": (0, 0, 0), "image": r"source\theme12.png"},
    13: {"color": (0, 0, 0), "image": r"source\theme13.png"},
    14: {"color": (0, 0, 0), "image": r"source\theme14.png"},
    15: {"color": (0, 0, 0), "image": r"source\theme15.png"},
    16: {"color": (0, 0, 0), "image": r"source\theme16.png"},
    17: {"color": (0, 0, 0), "image": None}
    }
```

## 사진 촬영
사진 촬영은 opencv 라이브러리를 사용하여 PC에 연결된 카메라를 인식하여 촬영하도록 설정하였습니다.<br>
사진 촬영시 셔터음이 실행되며, pillow 라이브러리를 통해 프레임의 사진 영역에 맞게 3:2로 크롭됩니다.<br>
사진은 photo폴더에 각 촬영마다 날짜로 구분해서 생성된 폴더에 저장됩니다.

<b>폴더 생성</b>

```
file_name = datetime.now().strftime("%m_%Y_%H_%M_%S")
save_dir = f"photo/{file_name}"
os.makedirs(save_dir, exist_ok=True)
```

<b>사진 촬영</b>

```
def take_picture(frame):
    global photo_count, countdown_index

    #촬영되는 소리 재생
    sound = pygame.mixer.Sound(r"source\camera.mp3")
    sound.play()

    #3:2 비율 중앙 크롭
    height, width, _ = frame.shape
    target_ratio = 3 / 2
    if width / height > target_ratio:
        new_width = int(height * target_ratio)
        x1 = (width - new_width) // 2
        frame_cropped = frame[:, x1:x1 + new_width]
    else:
        new_height = int(width / target_ratio)
        y1 = (height - new_height) // 2
        frame_cropped = frame[y1:y1 + new_height, :]

    filepath = f"{save_dir}/{photo_count}.jpg"
    cv2.imwrite(filepath, frame_cropped)
    captured_files.append(filepath)
    print(f"사진 {photo_count} 저장됨: {filepath}")
    photo_count += 1
    countdown_index = 0
```

<img src="/assets/img/take picture.png" alt="사진 촬영 화면 이미지">

[!]카메라가 비활성화 되어있어 OBS 가상 카메라 화면이 인식되었습니다.

## 사진 선택
촬영된 8개의 사진중 4개의 사진을 순서대로 선택하여 프레임에 적용합니다.<br>
선택된 사진은 테두리가 붉게 변하며, 우측 미리보기 프레임에 즉시 적용됩니다.<br>
선택된 사진은 앞서 생성한 사진 저장 폴더에 selected폴더를 추가하여 별도로 백업합니다.<br>
또한 리스트에 인덱스를 저장해서 최종 사진 제작에 활용됩니다.

<img src="/assets/img/select image.png" alt="사진 선택 화면 이미지">

[!]카메라가 비활성화 되어있어 OBS 가상 카메라 화면이 인식되었습니다.

## 종료 안내 화면
프로그램의 프로세스가 종료됐음을 알림과 동시에 최종 사진을 편집합니다.<br>
pillow 라이브러리를 활용하여 사진 편집 함수를 구현하였습니다.

pillow는 이미지를 좌상단을 기준으로 픽셀로 위치를 지정할 수 있으며 이를 기반으로 프레임에 사진을 붙혀넣었습니다.<br>
또한 아래 함수로 제작한 사진은 가로로 2개씩 배치하여 별도의 사진으로 제작하고, 이를 google api 또는 출력에 사용하였습니다.

```
def make_film(path1: str, path2: str, path3: str, path4: str, frame: str, index_num: int):
    #캔버스 사이즈
    canvas_width = 800
    canvas_height = 2400
    canvas = Image.new("RGBA", (canvas_width, canvas_height), (255, 255, 255, 255))

    path_list = [path1, path2, path3, path4]
    y_setting = [218, 750, 1282, 1814]  #Y좌표 기준

    for idx, path in enumerate(path_list):
        #이미지 열기 및 리사이즈
        img = Image.open(path).convert("RGBA")
        img = img.resize((624, 468), Image.LANCZOS)

        #붙일 위치 계산
        x = (canvas_width - 624) // 2
        y = y_setting[idx]
        
        canvas.paste(img, (x, y), mask=img)

    #프레임 이미지 붙이기
    frame_img = Image.open(frame).convert("RGBA")
    frame_img = frame_img.resize((canvas_width, canvas_height), Image.LANCZOS)
    canvas.paste(frame_img, (0, 0), mask=frame_img)

    #저장
    output_dir = "photo"
    os.makedirs(output_dir, exist_ok=True)

    output_file = os.path.join(output_dir, f"{index_num}.png")
    canvas.save(output_file, format="PNG")

    print(f"'{output_file}' 파일이 생성되었습니다.")
```

<img src="/assets/img/last.png" alt="종료 안내 화면 이미지">

## 마무리
이번 활동은 신갈고등학교 축제 "동아리 발표회"의 동아리 부스 운영을 위해 제작되었습니다.<br>
이번 활동을 통해 축제 기간 및 부스 추가 운영 기간 일주일간 187회 촬영이 이루어졌으며, 250명 이상의 학생이 촬영에 참여한 것으로 추정됩니다.
또한 현재 속한 동아리 '파고파고'가 우수동아리로 수상하는데 크게 기여할 수 있었습니다.

이번 활동은 기획, 구상, 제작 활동의 대부분을 담당하여 프로그램을 완성한 만큼 의미있는 활동이였던 것 같습니다.<br>
다만, 프로그램을 수정하는데 어려움을 겪었던 만큼 유지보수를 고려하여<br>
프로그램의 외적인 부분뿐 아니라 프로그램의 내적인 부분에서 역량의 부족을 느끼고,<br>
GUI 프로그래밍의 개발 역량을 증진시키는 계기로 활용할 수 있었습니다.