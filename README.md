# 音階検出プログラム

このプログラムは、マイクから入力された音声を解析し、音階を表示するものです。以下の手順に従って、プログラムを実行してください。

## 必要な環境

- Python 3.x
- Visual Studio Code (VSCode)
- 必要なPythonライブラリ

## インストール手順

### 1. Pythonのインストール

1. Pythonの公式サイトからPythonをダウンロードしてインストールします。
2. インストール時に「Add Python to PATH」にチェックを入れてください。

### 2. Visual Studio Code (VSCode) のインストール

1. VSCodeの公式サイトからVSCodeをダウンロードしてインストールします。

### 3. 必要なPythonライブラリのインストール

1. VSCodeを開きます。
2. ターミナルを開きます（`Ctrl + @`を押します）。
3. 以下のコマンドを入力して、必要なライブラリをインストールします：

```sh
pip install matplotlib pyaudio numpy opencv-python pillow
```

### 4. プログラムの実行

1. VSCodeで新しいPythonファイル（例：`main.py`）を作成します。
2. 以下のコードをコピーして、`main.py`に貼り付けます：

```python
import matplotlib.pyplot as plt
import pyaudio as pa
import numpy as np
import cv2
from PIL import Image, ImageFont, ImageDraw

RATE=44100
BUFFER_SIZE=16384

HEIGHT=300
WIDTH=400
SCALE=['ラ', 'ラ#', 'シ', 'ド', 'ド#', 'レ', 'レ#', 'ミ', 'ファ', 'ファ#', 'ソ', 'ソ#']

#ストリーム準備
audio = pa.PyAudio()
stream = audio.open(rate=RATE, channels=1, format=pa.paInt16, input=True, frames_per_buffer=BUFFER_SIZE)

#波形プロット用バッファ
data_buffer = np.zeros(BUFFER_SIZE*16, int)

#プロット描画
fig, (fft_fig, wave_fig) = plt.subplots(2, 1)

try:
  while True:
    #ストリームからデータ取得
    audio_data=stream.read(BUFFER_SIZE)
    data=np.frombuffer(audio_data,dtype='int16')

    #ハミング窓関数
    windowed_data = data * np.hamming(len(data))

    #FFT計算
    fd = np.fft.fft(windowed_data)
    fft_data = np.abs(fd[:BUFFER_SIZE//2])
    freq=np.fft.fftfreq(BUFFER_SIZE, d=1/RATE)

    #スペクトルで最大成分取得
    val=freq[np.argmax(fft_data)]
    offset = 0.5 if val >= 440 else -0.5
    scale_num=int(np.log2((val/440.0)**12)+offset)%len(SCALE)

    #描画準備
    canvas = np.full((HEIGHT,WIDTH,3),255,np.uint8)

    #日本語描画
    #フォントへのpath指定(フォントは無料配布サイトなどでダウンロードしpathを指定)
    font = ImageFont.truetype("path/to/XXXX.ttf", 120)
    canvas = Image.fromarray(canvas)
    draw = ImageDraw.Draw(canvas)
    draw.text((20, 100), SCALE[scale_num], font=font, fill=(0, 0, 0, 0))
    canvas = np.array(canvas)

    #判定結果描画
    cv2.imshow('sample',canvas)

    #プロット
    data_buffer = np.append(data_buffer[BUFFER_SIZE:],data)
    wave_fig.plot(data_buffer)
    fft_fig.plot(freq[:BUFFER_SIZE//20],fft_data[:BUFFER_SIZE//20])
    wave_fig.set_ylim(-10000,10000)
    plt.pause(0.0001)
    fft_fig.cla()
    wave_fig.cla()

    # 'q'キーが押されたらループを終了
    if cv2.waitKey(1) & 0xFF == ord('q'):
      break

except KeyboardInterrupt:
  pass

finally:
  #終了処理
  stream.stop_stream()
  stream.close()
  audio.terminate()
  cv2.destroyAllWindows()
```

3. `main.py`を保存します。
4. ターミナルで以下のコマンドを入力して、プログラムを実行します：

```sh
python main.py
```

## 注意事項

- プログラムを実行する際には、マイクが正しく接続されていることを確認してください。
- プログラムを終了するには、Figureタブで`q`を押した後、ターミナルで`Ctrl + C`を押してください。