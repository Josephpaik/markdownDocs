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

 