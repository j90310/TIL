python -m venv venv

source venv/Scripts/activate

pip install django==3.2.12 djangorestframework django_extensions

pip freeze > requirements.txt

django-admin startproject djangoplace .

touch .gitignore

gitignore 작성

python manage.py startapp accounts

python manage.py startapp articles

settings에서 만든 앱 설치한 앱 추가, AUTH_USER_MODEL = 'accounts.User'



accounts앱 models에 확장

```
from django.db import models
from django.contrib.auth.models import AbstractUser


class User(AbstractUser):
    pass
```

articles앱 models에 확장

```
from django.db import models
from django.conf import settings

class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='articles')
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='like_articles')


class Comment(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='comments')
    article = models.ForeignKey(Article, on_delete=models.CASCADE, related_name='comments')
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

User가 article, comment을 1대 N으로 가지고 있을 거고

articler과 comment사이는 자기들끼리 1대 N

 ```
 # 프로젝트/urls.py
 path('api/v1/articles/'#이라는 이름의 요청이 들어오면, include('articles.urls'#로 포워딩하겠다.))
 
 path('api/v1/accounts/', include('accounts.ruls'))
 
 # 물론 두 앱 다에 urls.py를 만들어 준다!
 ```



```
accounts/urls.py
articles/urls.py
from django.urls import path
from . import views

app_name = 'accounts'#articles

urlpatterns = [
	
]

#서버를 돌리기 위해 필요한 작업이다
```

python manage.py makemigrations 를 통해 모델링한거 반영해줍시다

python manage.py createsuperuser를 통해 관리자계정을 만들자



모델은 어떤 데이터를 실행할지담당하고 시리얼라이저는 그 데이터는 어떻게 표현할지를 담당하고 검증을 담당한다.



serializers.py를 아티클 앱에 추가

생성, 조회, 수정을 할 때 얘내들을 쓸 건데 다양하게 만들자

아티클즈에 동명 폴더를 만들고 article.py와 comment.py라는 파일을 여기에 만들자

```
articles/serializers/article.py

from rest_framework import serializers
from django.contrib.auth import get_user_model

from ..models import Article
from .comment import CommentSerializer

User = get_user_model()


class ArticleSerializer(serializers.ModelSerializer):
    
    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = ('pk', 'username')
	#유저를 추가로 시리얼라이즈한다.(from django.contrib.auth import get_user_model)
    comments = CommentSerializer(many=True, read_only=True)
    user = UserSerializer(read_only=True)
    like_users = UserSerializer(read_only=True, many=True)
	코멘트는 밖에 있는 걸 그대로 들고 오고
    class Meta:
        model = Article
        fields = ('pk', 'user', 'title', 'content', 'comments', 'like_users')
	#필요한 요소들 삽입

# Article List Read
class ArticleListSerializer(serializers.ModelSerializer):
    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = ('pk', 'username')

    user = UserSerializer(read_only=True)
    # queryset annotate (views에서 채워줄것!)
    comment_count = serializers.IntegerField()
    like_count = serializers.IntegerField()

    class Meta:
        model = Article
        fields = ('pk', 'user', 'title', 'comment_count', 'like_count')
```

serializers.py의 양이 너무 커서 쪼갠건데



# CORS

동일 출처에서 불러온 스크립트만 받아주겠다.

동일 출처란 URL에서 scheme, protocol, host, port 부분이 전부 동일한 부분만을 의미한다.

![image-20220517132043986](Vue + API.assets/image-20220517132043986.png)

![image-20220517132410196](Vue + API.assets/image-20220517132410196.png)

완전히 같은 동일출처에서만 요청을 보낼 수 있다

서버는 똑바로 할 일 했는데 브라우저의 CORS라는 정책에 의해 막혔다는 거

동일 출처 정책 - 특정 출처에서 불러온 문서 또는 스크립트가 다른 출처에 의한 상호작용 금지



그러면 그 해결책은?
CORS header



