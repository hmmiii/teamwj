---
title: Flask 블루프린트(Blueprint)로 플라스크 각 화면을 모듈화하자
author: hmmi
date: 2023-11-10 16:50
categories: [기술스택,Flask]
tags: [project,Flask,미니루틴,항해99,Python,기술스택]
pin: true
toc: true
toc_sticky: true
---


항해 99 사전스터디에서 토이 프로젝트를 하면서, 깃 레포 관리를 맡게 되었다. python flask의 기본적인 강의만 수료한 뒤에 하게 된 토이프로젝트였는데, 스터디원은 나까지 포함해서 4명인데, app.py 하나에서 작업하려고 하니 버전 관리가 어려울 것 같다는 생각이 들었다.

```
Project
├── app.py
├── readme.md
├── static
└── templates
    └── index.html
```

원래 강의에서 배운 대로라면 이런 식의 디렉토리 구조를 가진다. 템플릿 html파일은 각자 작업한다고 해도, app.py에 많은 로직이 들어 있기 때문에 문제가 발생할 것 같아 각 화면을 모듈화 할 방법이 없을까 고민하다가 플라스크에 블루프린트라는 기능이 있다는 것을 알게됐다.

## 블루프린트로 화면 모듈화

우선 블루프린트를 사용하면 디렉토리 구조를 이렇게 변경할 수 있다.

```
Project
├── app
│   ├── __init__.py
│   ├── database.db
│   ├── login
│   │   └── views.py
│   ├── main
│   │   └── views.py
│   └── models.py
├── readme.md
├── run.py
├── static
└── templates
    ├── footer.html
    ├── header.html
    └── index.html
```

아직 프로젝트에 들어가지 않아, 로그인과 메인페이지만 있다는 가정 하에 디렉토리를 나눠두었다. 이렇게 하면 메인페이지 작업자와 로그인페이지 작업자는 각각의 파일에서 작업이 가능하다.

블루프린트의 장점은 다음과 같다. (gpt 답변)

1. **모듈화(Modularity):** 기능이나 모듈에 따라 코드를 나누어 각각의 블루프린트로 관리할 수 있습니다. 이로써 코드의 구조가 보다 체계적이 되고 유지보수가 용이해집니다.
2. **재사용성(Reusability):** 블루프린트는 여러 애플리케이션에서 재사용할 수 있습니다. 예를 들어, 인증 관련 기능을 포함한 블루프린트를 다른 프로젝트에서도 사용할 수 있습니다.
3. **라우팅(Routing):** 각 블루프린트는 독립적인 URL 접두사를 가질 수 있습니다. 이로써 서로 다른 블루프린트에서 동일한 엔드포인트를 사용할 수 있습니다.

## 블루프린트 흐름 요약

1. app/화면/views.py에서 화면 블루프린트 생성 (Blueprint)
2. app/models.py에서 db 정보 정의
3. app/\_\_init__.py
	- create_app 함수 정의
	- 블루프린트 등록
4. run.app에서 애플리케이션 실행

## 주요 파일

### app/main/views.py

```python
from flask import Blueprint, render_template

# 프로젝트 루트 디렉토리 경로
main_bp = Blueprint('main', __name__, template_folder='../../templates')

@main_bp.route('/')
def home():
    context = {
        'title': '미니 루틴'
    }
    return render_template('index.html', data=context)
```

### app/models.py

```python
from flask_sqlalchemy import SQLAlchemy
import os
db = SQLAlchemy()

# 직접 속성을 설정
db_path = os.path.join(os.path.abspath(os.path.dirname(__file__)), 'database.db')
db_uri = f'sqlite:///{db_path}'

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    pw = db.Column(db.String(120), nullable=False)

    def __repr__(self):
        return f'<User {self.username}>'
```

### app/\_\_init__.py

```python
from flask import Flask
from app.models import db, db_uri
from app.login.views import login_bp
from app.main.views import main_bp

def create_app():
    app = Flask(__name__)

    # 데이터베이스 URI 설정
    app.config['SQLALCHEMY_DATABASE_URI'] = db_uri

    # 데이터베이스 초기화
    db.init_app(app)

    with app.app_context():
        db.create_all()

    app.register_blueprint(login_bp)
    app.register_blueprint(main_bp)

    return app
```

### run.py

```python
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run()

```
