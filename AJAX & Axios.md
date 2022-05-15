# XML REQUEST



브라우저 콘솔창에서

```
  const request = new XMLHttpRequest()
#1. 브라우저 탭 열가
  const URL = 'https://jsonplaceholder.typicode.com/todos/1'
#2. URL을 URL창에 입력
  request.open('GET', URL)
#3. 엔터
  request.send()
#4. 요청송신
  const todo = request.response
  console.log('end')
#5. 받아보기
```

# 어싱크로노스 자바스크립트

\# print('hi')

\# sleep(3)

\# print('bye')

파이썬은 하나의 과정이 끝나지 않으면 다음으로 넙어가지 않는다

자바스크립트의 특히 request.send()의 경우에는 끝나든 말든 그냥 다음으로 넘어간다! 그래서 이전 작업이 성립해야 이후가 나오는 경우에서, 그 이후 작업이 제대로 실행되지 않는다.

--------------



setTimeout(콜백함수, 시간숫자화, )시간이 지나면 이걸 해라

![image-20220515150506578](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515150506578.png)

그런데, 문제는 자바스크립트 콜스택은 그냥 다음으로 넘어가기 때문에, start end 둘다 뜬 상황에서 3초가 지나면 WEB API로 넘어간 after3seconds가 태스크 큐로 넘어가 대기중이게 되고 이벤트루프라고 하는 얘가 막는데, 콜스택이 비게 되면 실행된다는 것이다. 저기서 시간을 0이라고 해도 무조건 스타트 엔드보다 나중에 나온다.





![image-20220515152827572](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515152827572.png)

큐에서 올라가려면 콜스택이 비어있어야 한다.

Call Stack과 브라우저(Web Apis)는 상부상조하는 관계라는 거다.

# 콜백함수

다른 함수에 인자로 전달된 함수를 의미한다.

외부 함수 내에서 작업을 만들고 동기비동기호환

할당가능, 인자가능, 리턴가능!



비동기 로직, 특정 조건이 만족되면 함수를 실행해야 하기에 활용되는 개념이다.

# 프로미스

콜백함수 만으로 일을 순차적으로 진행하려면 필연적으로 콜백지옥이라는 상황이 펼쳐진다

1(234)2(34)3(3)4

이걸 그나마 좀 쉽게 하려면ㅇ....



.then(), .catch()

![image-20220515155306681](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515155306681.png)

각각의 .then블록은 서로 다륵 프로미스를 반환하고, 그렇다는 것은 .then을 여러 개 사용하여 연쇄적인 작업을 수행할 수 있음을 의미하고 또 결국 여러 비동기 작업을 차례대로 수행할 수 있다는 것을 의미(반환값 필수!). catch는 하나라도 틀리면 작동하게 된다.

![image-20220515155659086](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515155659086.png)



 

![image-20220515155905966](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515155905966.png)

# Axios

이 애를 어떻게 쓰냐

 

![image-20220515160225253](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515160225253.png)

이것은 기존의 방법



![image-20220515160332226](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515160332226.png)

```
```

promise를 활용하여, 이렇게 하는 것이 가능하다!

axios는 npm으로 설치하던가 cdn을 활용하던가 두 가지 방법이 있다.

```
axios.get("https://jsonplaceholder.typicode.com/todos/")
	.then(function(res) {
		console.log(res)
	}
#받아오는게 성공한다면 이거겠지
# res는 response의 줄임말입니다. function의 기능이 수행된 이후, 본문에서 수행될 것을 알려주는.... 그런 느낌입니다.
```

![image-20220515161447167](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515161447167.png)

이렇게 체인된다. 기억할 것.

![image-20220515161709726](C:\Users\administor\AppData\Roaming\Typora\typora-user-images\image-20220515161709726.png)

```
# this가 없으니 화살표함수로 표현해보자
responsePromise
	.then(res => res.data)
	.then(todo => todo.title)
	.then(title => console.log(title))
```

```javascript
# 배열에서 특정 데이터만 받아오려면
axios.get(URL)  // Promise 리턴
      // .then(res => res.data)
      // .then(todo => todo.title)
      // .then(title => console.log( title ))
      .then(res => {
        const todosArray = res.data
        const todo = todosArray.find(todo => todo.id ===10)
        return axios.get('${URL}${todo.id}')
     })
	  .then(res=> console.log(res.data))

      .catch(err => {
        if (err.response.status === 404) {
          alert('그딴건 없다.')
        }
      })
      .finally(() => console.log('어쨋든 끝!'))

```

# async & await

