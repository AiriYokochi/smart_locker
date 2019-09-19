# スマートロッカーPOC
作成者：横地
### いきさつ
総務からスマートロッカーを用いた備品の貸し出しを行いたい
との要望があり、POCを作成する
### 業務概要
期間：2019年9月13日～19日(3営業日)
人数：1人

### ご要望
* 工数をかけずに物品の貸し出しを実現したい
* 取り扱い品
    * テプラ・**プロジェクター**
    * 切手もできればベスト
* 利用者ログを取得
* 予約者のみがロッカー開錠できる仕組み
* 先ずは日比谷・豊洲等で検証利用/良ければ全国展開
---
## 完成物
### 外見
![見た目](https://i.imgur.com/NiMuLH3.jpg)
左部分にラズベリーパイが入っている(開かない)
![](https://i.imgur.com/ozCCvNP.jpg)
### ロック機構
電流を流すことで上下に棒が移動する
ソレノイドラッチを用いて実装した
![](https://i.imgur.com/YTgFj5d.jpg)
![](https://i.imgur.com/6RYBHNN.jpg)


### 認証開始状態
認証が可能になるとLEDライトが青に光る
![](https://i.imgur.com/cyHZIMY.jpg)
### 認証成功
認証成功するとLEDライトが緑に光る
![](https://i.imgur.com/CuRjhyS.jpg)


### 認証失敗
認証失敗するとLEDライトが赤に光る

(例1)ロッカーIDが異なる場合
![](https://i.imgur.com/MRIZOCb.jpg)
(例2)利用可能時間より早い場合
![](https://i.imgur.com/2uctusF.jpg)
(例3)利用可能時間を過ぎている場合
![](https://i.imgur.com/ZJ6dqaU.jpg)
### 返却時
返却時には備品に張り付けられているQRコードを読み取らせる
![](https://i.imgur.com/FFJ2WEp.jpg)
返却時画面
![](https://i.imgur.com/Ss3cYdb.jpg)

---
## POC仕様
プロジェクタ1台を管理するスマートロッカーを作成する。
ロッカーの開け閉めはQRコードを用いて行う。


### 想定される利用の流れ(貸出時)
1. **利用者**が**総務備品貸し出しサイト**から備品貸出を依頼
    * 貸出依頼時に**ロッカー利用希望**、**利用時間の指定**を行う
2. **総務備品貸し出しサイト**から**QRコード**が発行され、
自動メールで**利用者**に送信される
3. **利用者**は指定した時間にスマートロッカーの
**QRコード読み取り部**に**発行されたQRコード**かざすことで
ロックを外し、備品を借りる

※今回はQRコードが発行された状態とする

### 想定される利用の流れ(返却時)
1. **利用者**は返却時にスマートロッカーの**QRコード読み取り部**に
**備品に貼られているQRコード**をかざすことでロックを外し、
備品を返却する

---
### ハード仕様
* ロッカー本体
    * サイズ：220[mm]×300[mm]×90[mm]
    * 段ボールで作成
* ロック機構
    * ソレノイド
* raspberry3
    * マイコン
    * 読み取ったQRコードの照会を行う
* USBカメラ
    * QRコードの読み取りに用いる
* フルカラーLEDライト
    * 認証状態を表す
* 電源
    * rasbperry Pi3、回路用に5V
    * ソレノイド動作用に24V
### つなぎ方
raspberry Piの17PinをHIにすると、
ソレノイドへの電源供給がされ鍵が開く

![回路図](https://i.imgur.com/FzaJTh5.jpg)

---

### ソフト仕様
* 環境：rasbian
* 言語：Python2.7
* パッケージ：OpenCV, gpiozero, zbar

### ファイル構成

-- QRCode.py
　　QRコードを読み込み登録されたものならばPINをHIにする

### QRコード
発行されるQRコードには以下の情報が付与されている
1. 利用可能日
2. 利用可能時間帯
3. 利用者名（name）
4. ロッカーIDと扉番号(locker)
5. パスワード(key)

2019年9月15日の14時～17時にロッカーID00の00番扉を
開けるQRコードは以下の情報を持つ

```例:2019091514001700?name=TaroTanaka?locker=00_00?key=secret```

![](https://i.imgur.com/k9pxOBE.png)

名前はユーザ設定、パスワードはシステムから発行される。
備品に張り付けられているQRコードはユーザ名・パスワード
ともにシステムから設定され、ロッカーの扉を常時解錠可能。

### プログラムの流れ

以下にプログラムの流れを示す。
```
1. 初期化
    1-1. LED初期化
    1-2. OpenCV初期化・エラー処理
    1-3. QRコード初期化
    1-4. 自分のlocker_IDと最大ドアの数を読み込む
2. 起動画面表示
3. QRコードを読み取るループ
    3-1. グレースケール/バイナリ化
    3-2. PILに変換
    3-3. データがあれば内容表示(チェック関数呼出)
    3-4. 読み取った画像を1/4サイズにしてウィンドウに表示
    3-5. 中止キー読み取り
4. チェック関数
    4-1. データが文字列か確認
        4-1-1. エラー処理、QRコード読み取りへ
    4-2. 利用者名の確認と表示
        4-2-1. エラー処理、QRコード読み取りへ
    4-3. ロッカーIDと扉番号の確認と表示
        4-3-1. エラー処理
    4-4. 備品に張り付けられたQRコード確認
        4-4-1. ロック解除関数呼び出し
    4-5. 利用時間の確認
        4-5-1. エラー処理、QRコード読み取りへ
        4-5-2. ロック解除関数呼び出し
5. ロック解除関数
    5-1. ロック解除画面表示
    5-2. 30秒間ロックを解除する
    5-3. プログラム終了
```
### 実プログラム
QRCode.py
```C++
#!/usr/bin/python
# -*- coding: utf-8 -*-
import cv2
import zbar
import PIL.Image
from gpiozero import LED
from time import sleep
import datetime
import sys

# 初期化_LED
Rled = LED(10)
Bled = LED(9)
Gled = LED(11)
door0 = LED(17)
door1 = LED(21)

this_locker_ID = "00"
this_locker_door = "2"

#initialize CV2
cap = cv2.VideoCapture(0)
cap.set(3, 800)  # Width
cap.set(4, 800)  # Heigh
cap.set(5, 15)   # FPS
if cap.isOpened() is False:
    raise("IO Error")

# 初期化_initialize QR_Code
scanner = zbar.ImageScanner()
scanner.parse_config('enable')

def led_on(color):
    if(color == "R"):
        #pass
        Rled.on()
    elif(color == "B"):
        #pass
        Bled.on()
    elif(color == "G"):
        #pass
        Gled.on()

def led_off(color):
    if(color == "R"):
        #pass
        Rled.off()
    elif(color == "B"):
        #pass
        Bled.off()
    elif(color == "G"):
        #pass
        Gled.off()


def open_locker(str_door):
    sleep(1)
    led_off("R")
    led_off("B")
    led_off("G")
    sleep(1)
    led_on("G")
    sleep(1)
    led_off("G")
    sleep(1)
    led_on("G")
    print(" +-+-+--------------------------------------+-+-+")
    print("        " + str_door + "番の扉を解錠します。")
    print("    備品を取り出したら扉を閉めてください。")
    print("              30sec後に施錠されます")
    print(" +-+-+--------------------------------------+-+-+")
    if( int(str_door) == 0 ):
        door0.on()
        pass
    elif( int(str_door) == 1):
        door0.on()
        pass
    
    sleep(30)
    led_off("G")
    led_off("R")
    sleep(1)
    led_on("B")
    sleep(1)
    door0.off()
    door1.off()
    sys.exit()


def check_open(str_data):
    # 開けて良いデータかチェックする
    # 返り値はtrue/false

    if( type(str_data) is not str ):
        print("E: invalid data: not string")
    str_data_array = str_data.split("?")
    if(len(str_data_array) is not 4):
        print("E: invalid data: size not 4")

    # 利用者名の確認
    tmp = str_data_array[1].split("=")
    user_name = tmp[1]
    print("OK: 利用者は" + str(user_name)+ "さんです")

    # ロッカー番号の確認
    tmp = str_data_array[2].split("=")
    locker_num = tmp[1].split("_")
    if( this_locker_ID != str(locker_num[0])):
        led_off("G")
        led_on("R")
        print("E: ロッカーが異なるようです。")
        print("   このロッカーのID："+ this_locker_ID + ", 指定ロッカーのID：" + str(locker_num[0]))
        sleep(3)
        led_off("R")
        sleep(1)
        led_on("B")
        return 1
    if( 0 <= int(locker_num[1]) and int(locker_num[1]) <= this_locker_door):
        print("OK: ロッカーの扉番号は"+ str(locker_num[1]) + "番です")
    else:
        led_off("G")
        led_on("R")
        print("E: 指定されたロッカーの扉番号が正しくないようです")
        sleep(3)
        led_off("R")
        sleep(1)
        led_on("B")
        return 1


    # マスターキー(いつでも開けることができる)かどうか確認
    tmp = str_data_array[3].split("=")
    password = tmp[1]
    if( user_name == "Admin" and password == "cscscs"):
        print(" +-+-+--------------------------------------+-+-+")
        print("       　　  返却ありがとうございます　　　　　　　　　")
        print(" +-+-+--------------------------------------+-+-+")
        open_locker(str(locker_num[1]))


    # 利用時間の確認
    date_from = datetime.datetime.strptime(str_data_array[0][0:12], "%Y%m%d%H%M")
    date_due = datetime.datetime.strptime(str_data_array[0][0:8] + str_data_array[0][12:16], "%Y%m%d%H%M")
    today = datetime.datetime.today()
    
    # for debug
    today = datetime.datetime(2019, 9, 15, 16, 0)

    if( date_from <= today and today <= date_due ):
        led_off("G")
        led_on("R")
        print("OK: 利用可能時間の範囲内です")
        open_locker(str(locker_num[1]))
    elif( date_from >= today ):
        led_off("G")
        led_on("R")
        print("E: invalid date time: 利用可能時間より早いです")
        sleep(3)
        led_off("R")
        sleep(1)
        led_on("B")
        return 1
    elif( today >= date_due ):
        led_off("G")
        led_on("R")
        print("E: invalid date time: 利用可能時間を過ぎています")
        sleep(3)
        led_off("R")
        sleep(1)
        led_on("B")
        return 1


def read_QR():
    while True:
        #input image
        ret, img = cap.read()
        img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        
        #grayscale
        img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

        #Binarization
        tresh = 100
        max_pixel = 255
        ret, img = cv2.threshold(img, tresh, max_pixel, cv2.THRESH_BINARY)

        #picture change PIL
        pil_img = PIL.Image.fromarray(img)
        width, height = pil_img.size
        raw = pil_img.tobytes()
        image = zbar.Image(width, height, 'Y800', raw)

        #result
        scanner.scan(image)
        for symbol in image:
            print("---------------------------------------------------------------")
            print(symbol.data)
            print("---------------------------------------------------------------")
            led_off("B")
            led_on("G")
            check_open(symbol.data)

         # 1/4サイズに縮小
        img = cv2.resize(img, (int(img.shape[1]/4), int(img.shape[0]/4)))
        # 加工なし画像を表示する
        cv2.imshow('Raw Frame', img)

        #finish flag
        k = cv2.waitKey(1)
        if k == 27: #ESC
            break

if __name__ == '__main__':
    led_on("B")
    print("----------------------")
    print("  * CR Code Reader *  ")
    print("----------------------")

    QR_data = read_QR()
    print(QR_data)
    cap.release()
    cv2.destroyAllWindows()
    print("OK: 正常終了")

```

### 全体テスト
行ったテストは以下の5項目である。

実施日：2019年9月19日11時
実施者：横地
項目　：5
結果　：全5項目でクリア

1. プログラム起動時に以下が実行される
    1-1. 起動画面の表示
    1-2. LEDが青に光る
    1-3. カメラ画像が表示される
2. 指定された時間帯内のQRコードをかざす
    2-1. 認証成功画面の表示
    2-2. LEDが緑に2回点滅
    2-3. ドアのロックが外れる
    2-4. 30秒後にロックがかかり、プログラム終了 
3. 指定された時間外のQRコードをかざす
    3-1(1). 認証失敗画面の表示(利用時間が早いです) 
    3-1(2). 認証失敗画面の表示(利用時間を過ぎています)
    3-3. LEDが赤に3秒間光る
    3-4. LEDが青に光り、QRコード読み取りモードに戻る
4. 備品に張り付けられたQRコードをかざす
    4-1. 返却画面・認証成功画面の表示
    4-2. LEDが緑に2回点滅
    4-3. ドアのロックが外れる
    4-4. 30秒後にロックががかりプログラム終了 
5. カメラをつながずにプログラムを実行する
    5-1. エラー表示してプログラム終了
    
#### テストに使用したQRコード
QRコード作成サイトにて作成。

1. 利用時間内
![](https://i.imgur.com/L2bKOJo.png)

2. 利用時間を過ぎている
![](https://i.imgur.com/In6hcuZ.png)

3. 利用時間より早い
![](https://i.imgur.com/U1EKKsc.png)

4. ロッカーIDが異なる
![](https://i.imgur.com/BCdAwpA.png)

5. ロッカーの扉1番
![](https://i.imgur.com/KAXOCvP.png)

6. 備品に張り付けられるQRコード
![](https://i.imgur.com/rK38qsx.png)

---
### 今後の開発案
* システム
    * ネットワークに接続しパスワード照会を行う
    * ワンタイムパスワードの発行
    * 利用者と貸出者を別にする
    * QRコードの暗号化
    * ロッカー拡張
    * UI/UXの作成
* ハード
    * 電流不足の解決(ソレノイド用)
    * ラズパイにヒートシンクをつける
    * ロッカー拡張
    * アクリルなどの耐久性に優れた素材での作成
    * 画面を小さくし、カメラとともにロッカーに組み込む
