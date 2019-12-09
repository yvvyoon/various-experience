## Designing a RESTful API with Python and Flask

RESTful API로 구성된 서버의 특성

- Client-Server
    - 서비스를 제공하는 서버와 이를 소비 또는 사용하는 클라이언트로 분리된다.
- Stateless
    - 각 클라이언트의 요청은 서버가 그 요청을 처리하기 위해 필요로 하는 모든 정보를 포함하고 있다.
    - 서버는 각 클라이언트의 요청 정보를 저장할 수 없고, 다른 요청에 이를 사용할 수 없다.
- Cacheable
    - 서버는 각 요청이 캐시될 수 있는지 클라이언트에 표시해야 한다.
- Layered System
    - 클라이언트와 서버 간의 통신은 클라이언트가 다른 일을 할 필요 없이 중간에서 서버 대신 응답할 수 있도록 표준화되어야 한다.
- Uniform Interface
    - 클라이언트와 서버 간의 통신 방식은 동일해야 한다.
- Code on demand
    - 서버는 클라이언트에게 컨텍스트를 통해 실행가능한 코드 또는 스크립트를 제공할 수 있다.
    - 이 제약조건은 선택사항이다.

<br>

RESTful 웹 서비스의 핵심 개념은 URI로 표현되는, 리소스의 개념이다. 클라이언트는 HTTP 요청 메소드를 통해 URI로 요청을 보내고 그 결과로 리소스를 변경시킨다.

REST 디자인은 특정 데이터 포맷에 구애받지 않는다. URL 안에서의 쿼리 스트링일 수도 있고 JSON 형태일 수도 있다.

<br>

### 웹 서비스 설계

1. 서비스에 접속했을 때의 루트 URL을 설정한다.

    동일한 시스템 내에서 역할에 따라 이름을 통해 서비스를 구분할 수 있고 구 버전에 영향을 미치지 않고 새로운 기능을 추가할 수 있기 때문에 URL에 서비스의 이름과 버전을 추가해주는 것이 좋다.

```
http://[hostname]/todo/api/v1.0/
```

<br>

2. 각 기능에 사용할 HTTP 메소드 결정

| HTTP Method | URI                                             | Action          |
| :---------- | :---------------------------------------------- | :-------------- |
| GET         | http://[hostname]/todo/api/v1.0/tasks           | 할 일 목록 조회 |
| GET         | http://[hostname]/todo/api/v1.0/tasks/[task_id] | 할 일 조회      |
| POST        | http://[hostname]/todo/api/v1.0/tasks           | 할 일 생성      |
| PUT         | http://[hostname]/todo/api/v1.0/tasks/[task_id] | 할 일 수정      |
| DELETE      | http://[hostname]/todo/api/v1.0/tasks/[task_id] | 할 일 삭제      |

<br>

### 간단한 RESTful API 서버 만들기

Flask 프레임워크를 활용하여 파일 하나로 간단한 서버를 구동해볼 것이다.

`app.py`라는 파일을 생성해준다. 이제 각 기능 별로 적절한 HTTP 메소드를 사용해 코드를 구현한다.

- [GET] 할 일 목록 조회

```python
#!flask/bin/python
from flask import Flask, jsonify

app = Flask(__name__)

tasks = [
    {
        'id': 1,
        'title': u'식료품 사기',
        'description': u'우유, 치즈, 피자, 과일, 타이레놀',
        'done': False
    },
    {
        'id': 2,
        'title': u'파이썬 배우기',
        'description': u'괜찮은 파이썬 튜토리얼 찾기',
        'done': False
    },
]


@app.route('/todo/api/v1.0/tasks', methods=['GET'])
def get_tasks():
    return jsonify({'tasks': tasks})
```

<br>

- [GET] 특정 할 일 조회

```python
from flask import Flask, jsonify, abort

...

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['GET'])
def get_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]

    if len(task) == 0:
        abort(404)

    return jsonify({'task': task[0]})
```

<br>

- [POST] 새로운 할 일 등록

```python
from flask import Flask, jsonify, abort, request

...

@app.route('/todo/api/v1.0/tasks', methods=['POST'])
def create_task():
    if not request.json or not 'title' in request.json:
        abort(400)

    task = {
        'id': tasks[-1]['id'] + 1,
        'title': request.json['title'],
        'description': request.json.get('description', ''),
        'done': False,
    }

    tasks.append(task)

    return jsonify({'task': task}), 201
```

<br>

브라우저에서는 JSON 데이터로 접속이 어려우니 curl 커맨드로 직접 새로운 할 일을 등록하자.

```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"title": "빨래 하기"}' http://localhost:5000/todo/api/v1.0/tasks
```

<br>

추가된 할 일을 조회하자.

```
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
```

<br>

- 404 에러를 처리할 핸들러 메소드 정의

```python
from flask import Flask, jsonify, abort, request, make_response

...

@app.errorhandler(404)
def not_found(error):
  return make_response(jsonify({'error': 'Not found'}), 404)
```

<br>

- [PUT] 할 일 수정

```python
@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]

    if len(task) == 0:
        abort(404)

    if request.json != None:
        abort(40)

    if 'title' in request.json and type(request.json['title']) != unicode:
        abort(400)

    if 'description' in request.json and type(request.json['description']) is not unicode:
        abort(400)

    if 'done' in request.json and type(request.json['done']) is not bool:
        abort(400)

    task[0]['title'] = request.json.get('title', task[0]['title'])
    task[0]['description'] = request.json.get(
        'description', task[0]['description'])
    task[0]['done'] = request.json.get('done', task[0]['done'])
```

<br>

- [DELETE] 할 일 삭제

```python
@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]

    if len(task) == 0:
        abort(404)

    tasks.remove(task[0])

    return jsonify({'result': True})
```

<br>

`curl`로 수정, 삭제를 해보자.

```
$ curl -i -H "Content-Type: application/json" -X PUT -d '{"done": True}' http://localhost:5000/todo/api/v1.0/tasks/2
```

```
$ curl -i -H "Content-Type: application/json" -X DELETE http://localhost:5000/todo/api/v1.0/tasks/3
```

<br>

### 할 일의 ID를 URI로 대체

```python
...

@app.route('/todo/api/v1.0/tasks', methods=['GET'])
def get_tasks():
    # return jsonify({'tasks': tasks})
    # 리스트 내포를 사용
    return jsonify(({'tasks': [make_public_task(task) for task in tasks]}))

...
  
def make_public_task(task):
    new_task = {}

    for field in task:
        if field == 'id':
          	# url_for(): 라우팅이 설정된 함수에 대한 URL을 얻어오기 위한 함수
            new_task['uri'] = url_for('get_task', task_id=task['id'], _external=True)
        else:
            new_task[field] = task[field]

    return new_task
```

<br>

### 사용자 인증

이번 예제에서 데이터베이스를 사용하지 않고 간단하게 한 명의 사용자만 인증하는 기능을 구현해보자. 먼저 Flask의 `flask-httpauth` 익스텐션을 설치한다.

`pip install flask-httpauth`

```python
@auth.get_password
def get_password(username):
    if username == 'yoon':
        return 'python'

    return None


@auth.error_handler
def unauthorized():
    return make_response(jsonify({'error': 'Unauthorized access'}), 401)
```

<br>

`auth`에 관련된 데코레이터를 설정해주고, `get_task()` 메소드에 로그인되지 않은 사용자는 접근이 불가능하도록 하는 데코레이터도 설정해준다.

```python
@app.route('/todo/api/v1.0/tasks', methods=['GET'])
@auth.login_required
def get_tasks():
    return jsonify(({'tasks': [make_public_task(task) for task in tasks]}))
```

<br>

로그인하지 않고 URL을 요청하면 설정했던 에러 메시지가 출력된다.

```
$ curl -i http://127.0.0.1:5000/todo/api/v1.0/tasks
```

```json
// 20191209134422
// http://localhost:5000/api

{
  "error": "Not found"
}
```

<br>

ID와 비밀번호를 설정하여 curl을 날리면 정상적으로 할 일이 출력된다.

```
$ curl -u yoon:python -i http://127.0.0.1:5000/todo/api/v1.0/tasks
```

```json
// 20191209134556
// http://localhost:5000/todo/api/v1.0/tasks

{
  "tasks": [
    {
      "description": "우유, 치즈, 피자, 과일, 타이레놀",
      "done": false,
      "title": "식료품 사기",
      "uri": "http://localhost:5000/todo/api/v1.0/tasks/1"
    },
    {
      "description": "괜찮은 파이썬 튜토리얼 찾기",
      "done": false,
      "title": "파이썬 배우기",
      "uri": "http://localhost:5000/todo/api/v1.0/tasks/2"
    }
  ]
}
```

<br>

그러나, 웹 서비스를 개발할 때 인증 실패에 관한 에러 코드는 401보다 403을 권장하는 것으로 보인다.

<br>

> *401: Unauthorized*
>
> *403: Forbidden*

<br>

401을 사용하면 인증되지 않은 채 서비스에 접속하면 브라우저에서 로그인하라는 팝업을 띄운다. 보통 이런 팝업은 쓰지 않고 서비스 내에서 자체적인 로그인 로직을 태운다.

![image](https://user-images.githubusercontent.com/12066892/70408628-2f540300-1a8c-11ea-9f12-2c7f42852564.png)

403은 단순 에러만 뱉어내기 때문에 일반적으로 `403 Forbidden`으로 권장한다.

<br>

