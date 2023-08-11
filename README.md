# python_django
python_django


pip install django

django-admin startproject mysite

py manage.py startapp welcome

welcome 
admin.py : 設定資料庫呈現的模式，之後會跟models溝通
models.py : 建構你的資料庫型態
tests.py : 這是拿來檢查商業邏輯的地方，也就是用來測試你的邏輯是否有遺漏，這裡我們沒有要討論太多這方面的議題，但是你要記住，寫測試是一件相當重要的事情，千萬不能小看使用者的潛能
views.py : 相信你還沒有忘記，在 Day1我們這一位擔任控制者(controller)的身分，沒錯! 它就是寫商業邏輯的地方，它會跟urls.py做呼應，並將所需傳達給前端
urls.py : 它擔任著橋樑的角色，讓views.py與相對的網站做對應。蛤? 你說怎麼沒見到 urls.py，因為我們要自己建阿^^，在這裡建議可以把內部 ithome的urls.py複製貼過來就好，但是內容要修改!! 待會再來說明修改的部分
apps.py : 這裡你只要先了解，這是用來區別app的一個檔案即可
init.py : 相信大家都還沒有忘記^^，就是告訴Python這資料夾是個套件
migrations : 這資料夾裡面存放的內容，記錄著models裡面所創建的資料庫型態，