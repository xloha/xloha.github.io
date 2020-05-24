---

layout: post

title:  "(모듈) 연속된 숫자 찾아서 변환"

date:   2020-05-24 17:345:00 +0900

categories: java algorithm  

---



###### 알고리즘 공부 하면서 코드들을 모듈형식으로 만들면 기억도 쉽고, 여기저기 갔다 쓸 수 있을 것 같다는 생각을 가지고 있었는데 갑자기 포스팅 해봄,,,

<br>

# 연속된 숫자 찾고 변환

* 알고리즘 문제 중 팡팡게임(?) 터뜨리는 게임이 많이 나온다

* 연속된 char 또는 숫자를 찾길 원할 경우

* 가로 세로 3개 이상만 터뜨리길 원할 경우

  * BFS를 사용 시 - total이 3이 넘어갔는데 가로 세로 3이상이 아닌 곳이 터질 수 있음

   

```java
    // 3개 이상 연속된 숫자들을 찾아서 0으로 변환 
		int[] num = {8,2,3,4,4,5,5,5,3,8,6,9,9,9,9,1,5,5,5,1};
		
		for(int i = 0; i < num.length; i++) {
			int curNum = num[i];
			int total = 1;
			for(int j = i+1; j < num.length; j++) {
				if(curNum != num[j]) break;
				total++;
			}
			if(total >= 3) {
				for(int j = i; j < i+total; j++) {
					num[j] = 0;
				}
			}
		}

		
		for(int i = 0; i < num.length; i++) {
			System.out.print(num[i] + " ");
		}
```

