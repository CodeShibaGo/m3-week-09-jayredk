## 為什麼我們需要 WSGI？

WSGI(Web Server Gateway Interface) 是一個 Python 的規範 [PEP333](https://peps.python.org/pep-0333/)，

它定義了 Web Server 與 Web App / Framework 如何溝通

在 WSGI 出現前，Python Framework 要與 Web Server 溝通時，Python Framework 會透過各自創建的 API 進行溝通

此時，有很多不同的 Web Server 像是 Medusa、mod_python 等

Python Framework 不可能為所有的 Web Server 都開發一個分別支援的 API，也就是說，在這樣的環境下會讓 Python 開發者選擇框架時，受到 Web Server 的限制。

而 WSGI 出現後，以 WSGI 為基礎開發的 Web Server，開發者就可以在不影響既有的 Python app 下切換要使用的 Web Server

比如 gunicorn 用一用想換 uWSGI 之類的

結論：在如噴泉般爆發出來的 Web Server 中，定義一個規範，好讓大家以這個規範進行 Web Server 的開發，以後 Python framework 只要能介接 WSGI Server 即可

### Reference
- [An intro to WSGI and ASGI ](https://www.youtube.com/watch?v=5TYrPkugT5s)

## WSGI 跟 WSGI Server 有什麼不一樣？
WSGI 定出規範，WSGI Server 負責實現這個規範。

## ASGI 又是什麼？
ASGI(Asynchronous Server Gateway Interface)
如果 WSGI 是 Michael Jordan，ASGI 就是 Kobe Bryant

ASGI 是 WSGI 的接班人（ASGI 是 WSGI 的 superset），在 WSGI 的基礎上提供了非同步的機制，也就是在同一時間可以處理多個請求（透過切換的方式）

原先的 WSGI 只有支援同步（Synchronous）的機制
一次只能處理一個請求，所以如果請求需要花費較久的時間，WSGI server 在這段時間就會原地等待，無法做其他事

可以想成到麥當勞點餐，你點完餐之後要等你的餐出了之後，櫃檯人員才能為下一個人點餐
那在餐好了之前櫃檯人員就會呈現閒置狀態

為了解決這個問題就有非同步（Asynchronous）的機制產生
櫃檯人員點完一位客人的餐後可以馬上點下一個位客人的餐
這樣就能夠有效減少客人點餐到拿到餐點的時間

不過這不代表 ASGI 就好棒棒，WSGI 就很爛
還是需要看情況

### Reference

- [ASGI doc](https://asgi.readthedocs.io/en/latest/)
- [WSGI & ASGI Simplified](https://www.youtube.com/watch?v=LtpJup6vcS4)

## 說明 Gunicorn 是什麼？該如何使用它？
Gunicorn 是最流行的 WSGI HTTP server，簡單易用
主要處理前端 Web Server 發送的請求，並轉發給 Web App
最後將 Web App 回傳的結果返回給 Web Server

![Gunicorn 相關架構](https://miro.medium.com/v2/resize:fit:1400/1*i_9gtKakG2QjgwXdFDdwRA.jpeg)

### 快速使用
#### Step 1：安裝 Gunicorn
```shell
pip install gunicorn
```

#### Step 2：建立一個 WSGI entry point
wsgi.py
```python
from yourproject import app

if __name__ == "__main__":
    app.run()
```

> Note：這是一個在大型專案常見的 practice，不是必須。實踐的是關注點分離

#### Step 3：運行 Gunicorn
```shell
gunicorn --bind 0.0.0.0:5000 wsgi:app 
```
這段指令的意思是我要用 gunicorn 執行 wsgi 這個 module 的 app 並監聽 0.0.0.0:5000

0.0.0.0 的意思是開放可訪問的 IP 連線，不是實際的 IP

比如這台 server 有一個外網 IP，一個內網 IP

那外網的人如果想要訪問 gunicorn，只要知道外網 IP 並代上 5000 port 即可訪問

同樣的意思，內網的人只要訪問的到內網 IP 都可以連到 gunicorn

運行 gunicorn 時會有一些常見的參數設定，可以參考[文件](https://docs.gunicorn.org/en/latest/run.html#commonly-used-arguments)

詳細的參數以及預設值可以在[這裡](https://docs.gunicorn.org/en/latest/settings.html#settings)找到

這邊簡單列幾個參數
- -b / --bind

  設定監聽的介面以及 port 號，預設值是 127.0.0.1:8000。也就是只有本機能訪問

- -w / --workers

  設定有幾個 worker process 可以處理請求，預設值為 1，文件建議的數量是 2-4 x $(NUM_CORES)，NUM_CORES 指的是主機的 CPU 核心數

### 如何讓 Ubuntu 開機時自動執行 Gunicorn

#### Step 1：新增一個 systemd unit file
```shell
sudo nano /etc/systemd/system/myproject.service
```

#### Step 2：填入相關資訊
```
[Unit]
Description=Gunicorn instance to serve <your-project>
After=network.target

[Service]
User=<your-username-of-computer>
Group=www-data
WorkingDirectory=/home/<your-username-of-computer>/<your-project>
Environment="PATH=/home/<your-username-of-computer>/<your-project>/<your-virtualenv-folder>/bin"
ExecStart=/home/<your-username-of-computer>/<your-project>/<your-virtualenv-folder>/bin/gunicorn --workers 3 --bind unix:<your-project>.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```
記得要將

- `<your-username-of-computer>` 換成主機的使用者名稱

可以輸入 `whoami` 來查看

- `<your-project>` 換成你的專案資料夾名稱
- `<your-virtualenv-folder>` 換成你的虛擬環境資料夾名稱

systemd unit file 可以分成 3 塊，Unit、Service 以及 Install

Unit 主要是描述 metadata 和 dependencies

Service 定義 User、Group 和執行檔相關的路徑

Install 定義了哪一個 service 運行後再執行當前的 systemd unit file


### Reference
- [Gunicorn doc](https://gunicorn.org/)
- [Digital Ocean - How to configure Gunicorn](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-22-04#step-4-configuring-gunicorn)