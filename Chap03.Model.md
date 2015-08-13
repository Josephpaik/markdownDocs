#패턴 - 모델 믹스인Model mixin# 

**문제**  
개별 모델들은 같은 필드나 메서드를 중복적으로 가지는 경우가 있다

**해법**   
공통되는 필드와 메서드를 추출하여 다양한 재사용 가능한 모델 믹스인을 만든다   

```python
class Postable(models.Model):
	created = models.DateTimeField(auto_now_add=True)
	modified = models.DateTimeField(auto_now=True)
	message = models.TextField(max_length=500)

	class Meta:
		abstract = True
		
class Post(Postable):
	...
	
class Comment(Postable):
	...
```

Post와 Comment에는 모두 생성일자와 수정일자 그리고 메시지가 사용된다.

이 세가지 공통 필드를 Postable이라는 추상 클래스로 정의하여 상속하여 사용한다.
추상 클래스로 지정하려면 서브클래스 Meta에 ** abstract = True **를 지정한다.

사실, 생성일자와 수정일자만 있는 추상 클래스라면 timestamp 기능을 사용할 수 있다. 
이런 경우 보통 model mixin을 사용한다.

#모델 믹스인#

모델믹스인은 어떤 모델의 부모클래스에 추가하는 추상 클래스이다. 
파이썬은 다중상속을 지원하기 때문에 부모 클래스를 몇 개든 지정할 수 있다.

믹스인은 반듯반듯하고 쉽게 조합할 수 있어야 한다. 기본 클래스 록목에 믹스인을 넣고 나면 동작해야한다.
이런 측면에서 보면 상속보다는 조합(compositon)과 비슷하게 동작한다.

믹스인은 작을 수록 좋다. 한가지 동작만 잘수행하도록 해야한다.

앞에서 본 예제코드를 개선해보자.

```python
class TimeStampedModel(models.Model):
	created = models.DateTimeField(auto_now_add=True)
	modified = models.DateTimeField(auto_now =True)
	
	class Meta:
		abstract = True
		
class Postable(TimeStampedModel):
	message = models.TextField(max_length=500)
	...
	
	class Meta:
		abstract = True
		
class Post(Postable):
	...

class Comment(Postable):
	...
``` 
위의 코드를 보면 기본 클래스가 두 개가 되었고 기능이 명확하게 분리되었다.

#패턴 - user profiles#
공식적으로 추천하는 솔루션은 사용자 프로파일 모델을 생성하는 것이다. 
이럴 경우 사용자 모델과 프로파일 모델이 일대일 관계를 가져야 한다.
사용자 정보에 추가할 항목은 모두 이 프로파일모델에 저장한다.

```python
from django.db import models

class Profile(models.Model):
	user = models.OneToOneField(settings.AUTH_USER_MODEL, 
		primary_key = True)
```  

profile에 사용하는 모든 상세 필드는 null값 허용이거나 기본값을 부여하는 것을 추천하다.
그리고 프로필 인스턴스를 생성하는 동안 신호 처리기(signal handler)가 아무런 초기 변수도 전달하지 않도록 한다.

#신호(Signals)#
이상적인 상황이라면 사용자 모델 인스턴스가 만들어질 때마다 거기에 상응하는 사용자 프로필 인스턴스가 만들어져야한다.
이것은 보통 신호(signals)을 통하여 구현된다.
예를 들면 아래와 같은 신호처리기를 사용하는 사용자 모델의 **post_save** 신호를 듣고 처리하면 된다.

```python
# signals.py

from django.db.models.signals import post_save
	from django.dispatch import receiver
from django.conf import settings
from . import models

@receiver(post_save, sender=settings.AUTH_USER_MODEL)
def create_profile_handler(sender, instance, created, **kwargs):
	if not created:
		retrun
	# Create the profile object, only if it is newly created
	profile = models.Profile(user=instance)
	profile.save()
```

profile model은 사용자 인스턴스 외에 아무런 초기변수도 전달하지 않는다.

앞의 예에서는 signal code를 초기화 해주는 부분이 어디에도 없다.
**__init__.py** 패키지를 생성하여 초기값을 지정해 줄 수 있다.
작성할 app의 __init__.py 파일 안에 아래 처럼 설정한 다음
```python
ProfileConfig:
	default_app_config = "profiles.apps.ProfileConfig"   
```
 app.py안에 ProfileConfig메서드를 AppConfig의 서브클래스로 정의하고 **ready**메서드로 signal을 설정한다.
 
 ```python
 # app.py
 fromd django.apps import AppConfig
 
 class ProfileConfig(AppConfig):
	 name = "profile"
	 verbose_name = 'User Profiles'
	 
	 def ready(self):
		 from . import signals
 ``` 
 이렇게 signal을 설정하면 새로운 사용자를 생성할 때도 user.profile에 접근하여 Profile 객체를 얻을수 있다. 
 
 
 #Admin#  
 
 **UserAdmin**을 아래와 같이 커스터마이징한다.
 
 ```python
# admin.py
	from django.contrib import admin
	from .models import Profile
	from django.contrib.auth.models import User
	
	class UserProfileInline(admin.StackedInline):
		model = Profile
		
	class UserAdmin(admin.UserAdmin):
		inlines = [UserProfileInline]
		
	admin.site.unregister(User)
	admin.site.register(User, UserAdmin) 
 ```
 
 
 #여러 가지 프로파일 타입 Multiple profile types#
 
 사용자에 따라 서로 다른 프로파일을 여러개 사용해야하는 경우 Aggregate profile을 추천한다.
 슈퍼히어로 타입과 일반인 타입 프로필을 사용하는 경우는 아래와 같이 쓸수있다.
 
 ```python
class BaseProfile(models.Model):
	USER_TYPES = (
		(0, 'Ordinary'),
		(1, 'SuperHero'),
	)
	user = models.OneToOneField(settings.AUTH_USER_MODEL, primary_key=True)
	user_type = models.IntegerField(max_length=1, null=True, choices=USER_TYPES)
	bio = models.CharField(max_length=200, blank=True, null=True)

	def __str__(self)
		return "{}: {:.20}". format(self.user, self.bio or "")
		
	class Meta:
		abstract = True
		
		
class SuperHeroProfile(models.Model):
	origin = models.CharField(max_length=100, blank=True, null=True)
	
	class Meta:
		abstract = True
		
		
class OrdinaryProfile(models.Model):
	address = models.CharField(max_length=200, blank=True, null=True)
	
	class Meta:
		abstract = True


class Profile(SuperHeroProfile, OrdinaryProfile, BaseProfile):
	pass
	
 ``` 
 
#패턴 - 서비스 객체#
 **문제**
 
 >모델들이 커지면 감당 할 수 없게 될수 있다. 한 모델이 하나 이상의 일을 하게 되면 
 >테스트하고 유지관리하는게 힘들어진다.
 
 **해법**
 
 >연관된 일련의 메서드들을 특화된 **Service** 객체로 간추려낸다.

#구체적인 문제점#
'뚱뚱한 모델, 날씬한 뷰'가 장고 초보자를 위한 일반적인 격언이다. 
이상적인 경우라면, 뷰에는 프리젠테이션 로직외에는 아무것도 담겨있지 않아야 한다.
 
하지만, 시간이 좀 지나면 갖다 둘곳이 마땅치 않은 코드는 죄다 모델로 들어온다.
모델은 금새 코드들의 야적장이 되어버린다.

Service 객체를 사용할 때라는 단초는 다음과 같다:

> 1. Interactions with external services, for example, checking whether the
> user is eligible to get a SuperHero profile with a web service.
> 2. Helper tasks that do not deal with the database, for example, generating
> a short URL or random captcha for a user.
> 3. Involves a short-lived object without a database state, for example,
> creating a JSON response for an AJAX call.
> 4. Long-running tasks involving multiple instances such as Celery tasks.  
 
 ~구체적인 솔루션은 생략~
 
#패턴 - 프로퍼티 필드#
**문제** 

>모델은 메서드로 실행되는 속성(attributes)을 가진다. 하지만 이런 속성은 데이터베이스에 
>자리잡고 있으면 안된다.

**해법**

>이런 메서드에는 프로퍼티 데코레이터를 사용한다.
  
~ 상세항목 생략 ~

#패턴- 커스텀 모델 관리자#
**문제**

> 어떤 쿼리문들은 코드 여기 저기서 반복적으로 사용된다.

**해법**
> 공통된 쿼리에 의미있는 이름을 부여하는 일반 관리자를 정의한다.
~ 상세항목 생략 ~


 
 
 
 