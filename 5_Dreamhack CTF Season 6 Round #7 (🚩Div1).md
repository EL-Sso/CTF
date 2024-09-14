# Secure Image Storage

먼저 파일을 다운받으면 
![스크린샷 2024-09-14 050836](https://github.com/user-attachments/assets/94099991-7a30-4a44-b277-6be17e59e6f1)

대충 이렇게 4개의 파일을 볼 수 있는데 
flag.txt는

![image](https://github.com/user-attachments/assets/7e0c3fa1-efd1-4921-8861-e5dde888c1b5)

역시나 fake였다.
requirment는 구성 버전들이고 schema.sql은

![image](https://github.com/user-attachments/assets/b25850a4-304e-4b74-994c-3f5dde9f771d)

간단한 files 테이블의 구조를 볼 수 있다.
마지막으로 app.py를 들어가보면 

```
import sqlite3
from time import sleep
from hashlib import md5
from urllib import parse
from uuid import uuid4, UUID
from datetime import datetime
from selenium import webdriver
from os import urandom, getcwd, path, makedirs
from selenium.webdriver.chrome.service import Service
from flask import Flask, request, g, render_template, redirect, send_file, make_response

app = Flask(__name__)
app.secret_key = urandom(32)

try:
    FLAG = open('./flag.txt','r').read()
except:
    FLAG = 'DH{fake_flag}'

DATABASE = 'database.db'

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
        db.row_factory = sqlite3.Row
    return db

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

def init_db():
    with app.app_context():
        db = get_db()
        with app.open_resource('schema.sql', mode='r') as f:
            db.cursor().executescript(f.read())
        db.commit()

def get_data(query, args=(), one=False):
    cur = get_db().execute(query, args)
    rv = cur.fetchall()
    cur.close()
    return (rv[0] if rv else None) if one else rv

def insert_data(query, args=()):
    db = get_db()
    cur = db.cursor()
    cur.execute(query, args)
    db.commit()
    rowcount = cur.rowcount
    cur.close()
    return rowcount

def access_file(path, cookie={"name": "name", "value": "value"}):
    try:
        service = Service(executable_path="/chromedriver-linux64/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get(f"http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(f"http://127.0.0.1:8000/{path}")
        sleep(1)
    except Exception as e:
        print(e, flush=True)
        driver.quit()
        return False
    driver.quit()
    return True

@app.route('/', methods=['GET'])
def index():
    return redirect('/upload')

@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if request.method == 'GET':
        return render_template('upload.html')
    else:
        username = request.form['name'].strip()
        if len(username) > 30 :
            return render_template('upload.html', msg= 'The length of the username must be 30 characters or less')
        file = request.files['file']
        if file.filename == '':
            return redirect('/upload')
        
        if len(path.splitext(file.filename)[0]) > 30:
            return render_template('upload.html', msg= 'The length of the filename must be 30 characters or less')
        
        if ('.' in path.splitext(file.filename)[0]) or ('/' in path.splitext(file.filename)[0]):
            return render_template('upload.html', msg= 'Not allowed character')

        if path.splitext(file.filename)[1].lower() != '.png':
            return render_template('upload.html', msg= 'Only PNG files are allowed')
        
        if file.content_type != 'image/png':
            return render_template('upload.html', msg= 'Only PNG files are allowed')
        
        username = parse.quote(username, safe='')
        filename = username + '_' + parse.quote(path.splitext(file.filename)[0],safe='%')
        uuid = str(uuid4())
        base_path = getcwd()
        full_path = path.join(base_path+'/static/', uuid)
        makedirs(full_path)
        try:
            file.save(full_path + '/' + filename)
        except Exception as e:
            print(e, flush=True)
            return render_template('upload.html', msg= 'Something error')
        
        date = datetime.now().isoformat()
        encoded_data = (username+filename).encode('utf-8')
        md = md5()
        md.update(encoded_data)
        hash = md.hexdigest()
        insert_data('INSERT INTO files (username, filename, hash, uuid, date) values(?,?,?,?,?)', [username,filename,hash,uuid,date])
        return render_template('upload.html', msg='Upload success')


@app.route('/<username>', methods=['GET','POST'])
def storage(username):
    files=get_data('SELECT * FROM files WHERE filename LIKE ?',[username+'_%'])
    return render_template('storage.html', files=files)


@app.route('/static/<uuid>/<filename>')
def show(uuid, filename):
    uuid_obj = UUID(uuid, version=4)
    if str(uuid_obj) != uuid:
        return 'Not vaild UUID'
    file = get_data('SELECT * FROM files WHERE uuid=?',[uuid], one=True)
    if not file:
        return 'Directory does not exist'
    base_path = getcwd()
    if path.exists(base_path+f'/static/{uuid}/{filename}'):
        res = make_response(send_file(base_path+f'/static/{uuid}/{filename}'))
        # res.headers.add('X-Content-Type-Options', 'nosniff')
        res.headers.add('Content-Type', 'image/png')
        res.headers.add(f'X-Confirmed-{parse.unquote(filename)}', parse.unquote(file[1]) +'_' + file[3])
        for header in res.headers.to_wsgi_list():
            if 'Content-Type' in header[0]:
                if (not header[1].startswith('image/png')) and (not header[1].startswith('application/octet-stream')):
                    return 'Invaild Content-Type'
        return res
    else: 
        return 'File does not exist'


@app.route("/report", methods=["GET", "POST"])
def report():
    if request.method == "POST":
        path = request.form.get("path")
        if not path:
            return render_template("report.html", msg="fail")
        else:
            if access_file(path, cookie={"name": "flag", "value": FLAG}):
                return render_template("report.html", msg="Success")
            else:
                return render_template("report.html", msg="fail")
    else:
        return render_template("report.html")


if __name__ == "__main__":
    init_db()
    app.run(host='0.0.0.0', port=8000)
```
이런 코드를 볼 수 있다. 일단 해당 웹페이지를 보면

![image](https://github.com/user-attachments/assets/58439316-fae8-474f-8230-e3e3f47ec51a)

이렇게 id를 받고 파일을 업로드할 수 있는 것처럼 보인다.
일단 한번 업로드 해보겠다.

![image](https://github.com/user-attachments/assets/c13d5396-6203-4b17-9475-04709a4c04ee)

![image](https://github.com/user-attachments/assets/cac60442-938f-4c6d-a372-5145388fb9b1)

업로드한 후 경로에서 /<username> 즉, 내가 업로드할 때 입력했던 아이디를 넣으면

![image](https://github.com/user-attachments/assets/129ee463-e6e6-4781-a6ef-e6d37373ca2d)

업로드 내용이 저장되어있는 것을 볼 수 있는데 이걸 클릭해보면

![image](https://github.com/user-attachments/assets/a537ecc6-6360-47db-b9a8-0b336f00891b)


이렇게 /static 경로로 들어와서 파일이 존재하지 않는다고 뜬다.
다음 /report 경로로 들어가보면

![image](https://github.com/user-attachments/assets/344bee5c-00e4-4609-a16c-3bbc557e31e5)

path를 입력할 수 있는 창을 볼 수 있다.
아까 /static부분 경로를 여기다 넣고 report해보면
![image](https://github.com/user-attachments/assets/74f585ff-5d25-4493-bcb4-86df77cc977a)

코드를 보면 Success가 떴으니 flag.txt가 열려야되는 것 같은데 아무것도 안된다.
그래서 burp suite를 켜서

![image](https://github.com/user-attachments/assets/6bc80279-1d65-4fad-a701-654fa0311c44)

php로 웹쉘을 만든것을

![스크린샷 2024-09-14 204404](https://github.com/user-attachments/assets/72863759-7676-4e2e-aad1-5ac3e5b5113e)

png만 받으니까 

![스크린샷 2024-09-14 204520](https://github.com/user-attachments/assets/2f7b49e4-8572-4258-9be0-a6c172e9f343)

이렇게 업로드 시키면 

![스크린샷 2024-09-14 204615](https://github.com/user-attachments/assets/35e265d0-a900-41e1-95b0-eb58433c854d)

성공한다 이걸 이제 /flag 즉,username으로 들어가보면

![스크린샷 2024-09-14 204659](https://github.com/user-attachments/assets/14d75cf5-3ed9-40c6-b9c2-85c4e3fb2582)

이런식으로 뜨는데 report로 들어가서 path에 저 경로를 붙여봐도 success만 뜰 뿐 아무것도 없다..

/static에 아무것도 저장이 안되 걸 보니 아무래도 이쪽을 건들여야하는 것 같은데 어떤식으로 건들여야하는지 아직 감이 잘 안잡힌다.
