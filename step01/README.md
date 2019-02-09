# 도입
* 컴퓨터는 마법의 상자가 아니다. 모든 현상에는 이유가 있다. 문제가 있다면 잘못된 코드가 있기 때문에 그런 것이다. 
* 플로우 : 메모리에 적재되어 있는 명령이 순차적으로 ALU에 의해서 실행되는 과정. 아무도 개입할 수 없고 한번 적재된 명령은 쭉 실행된다. 싱크. 동기화 명령들이 이 플로우를 이루게 된다.
* 서브루틴 : 루틴은 플로우 그 자체를 의미한다. 명령어 세트가 루틴이다. 여러번 실행할 수 있는 명령어의 세트가 루틴인 것이다. 서브루틴의 특징은 자기를 호출했던 메인 플로우로 돌아간다는 것이다. 메인 플로우는 루틴 A를 만나 서브루틴이 복귀해야 하는 지점을 정해놓고 서브루틴으로 갔다가 정확히 그 지점으로 돌아간다. 

## 예제
```js
 const routineA = b => {
   const result = b * 2;
   console.log(result);
   return result;
 };
 const routineB = d => {
   const result = d * 3;
   console.log(result);
   return d;
 };
 // main flow
 const b = 10, d = 30;
 const a = routineA(b);
 console.log(a);
 const c= routineB(d);
 console.log(c);
```

* 이제껏 함수를 수학적으로만 접근했을 것이다. 이제부턴 함수가 아니라 플로우의 관점에서 접근해야 한다. 이 관점에서 함수는 Function이 아니라 Routine이다. 
* 메인 플로우와 루틴 사이엔 통신이 일어난다. 우리는 메인 플로우에서 서브루틴에게 값을 줄 땐 인자를 사용한다. 거꾸로 서브루틴에서 메인플로우에게 값을 돌려줄 땐 리턴을 사용한다.
* 자바스크립트는 리턴 없는 루틴이 없다. 리턴 값이 없으면 자동으로 리턴 값으로 undefined가 붙는다.
* 서브루틴도 하위 서브루틴을 가질 수 있다. 하위 서브 루틴의 입장에서 상위 서브루틴은 메인 플로우다. 이때 상위 서브루틴은 무조건 메모리에 킵 되어야 한다.

## 예제
```js
const routineA = arg => routineB(arg * 2);
const routineB = arg => arg * 3;

const b = 10;
const a = routineA(b);
```

* 이렇게 킵 한 각각의 메모리의 스냅샷을 콜 스택이라고 부른다. 

```js
const r1 = a => r2(a*2);
const r2 = a => r3(a*3);
const r3 = a => r4(a*4);
const r4 = a => r5(a*5);
const r5 = a => r6(a*6);
const r6 = a => a*5;
const b = 10;
const a = r1(b);
```

* 값은 메모리 상에서 전달할 때마다 복사되는 형태고, 참조는 메모리 상에서 공유된 객체를 가리키는 포인터만 전달되는 형식이다.
* 값이냐 참조냐 하는 건 언어가 정한다. 자바스크립트는 문자열을 값으로 보지만 자바에서는 참조로 본다. 값으로 보는 게 Primative Type(원시타입)이다. 얘네들 말고는 싹 다 참조다.  


## 예제
```js
const routine = a => a * 2;
const flow1 =_=> {
  const b = 10, d = 20;
  const a = routine(b);
  const c = routine(c);
  return a * c;
};
const flow2 =_=> {
  const b = 30, d = 40;
  const a = routine(b);
  const c = routine(c);
  return a + c;
};
flow1();
flow2();
```

* 위와 같이 값을 기반으로 하면 어디에서 뭐를 몇번 부르건 상관이 없다. 

## 예제1-1
```js
const routine = ref => ['a', 'b'].reduce((p,c) => {
  delete p[c];
  return p;
}, ref);
const ref = {a:3, b:4, c:5, d:6};
const a = routine(ref);
ref === a; // true
```

* 참조가 넘어오는 건 어쩔 수 없지만 그 참조는 아래처럼 읽기 전용으로만 사용하는 게 좋다. 

## 예제1-2
```js
const routine = ({a, b, ...rest})=>rest;
const ref = {a:3, b:4, c:5, d:6};
const a = routine(ref);
ref !== a; // true 
```

## 예제2-1
```js
const routine = ref => {
  const local = ref;
  local.e = 7;
  return local;
};
const ref = {a:3, b:4, c:5, d:6};
const a = routine(ref);
ref === a; // true 
```

## 예제2-2
```js
const routine = ref => ({...ref, e:7});
const ref = {a:3, b:4, c:5, d:6};
const a = routine(ref);
ref !== a; // true 
```
* es6부턴 객체의 순서가 중요하다. 이전에는 해쉬맵의 형태였다면 지금은 링크드 맵이라고 보면 된다.


# COUPLING 
* 커플링은 의존을 양쪽에서 하고 있는 경우를 의미한다.
* 아래는 컨텐츠 커플링을 하고 있는 나쁜 예제이다.

## 예제1
```js
const A = class {
  constructor(v) {
    this.v = v;
  }
};
const B = class {
  constructor(a) {
    this.v = a.v;
  }
};
const b = new B(new A(3));
```

## 예제2
```js
const Common = class {
  constructor(v) {
    this.v = v;
  }
}
const A = class {
  constructor(c) {
    this.v = c.v;
  }
};
const B = class {
  constructor(c) {
    this.v = c.v;
  }
};
const a = new A(new Common(3)); 
const b = new B(new Common(3));
```

## 예제3
```js
const A = class {
  constructor (member) { this.v = member.name; }
};
const B = class {
  constructor(member) { this.v = member.age;}
};
fetch('/member').then(res=>res.json()).then(member => {
  const a = new A(member); 
  const b = new B(member);
});
```

* 위의 external 커플링은 나쁜 케이스다. 예를 들어 member.name이 아니라 member.username을 쓰기로 하면 A 클래스는 망가진다. 하지만 서버에서 json 데이터를 받아쓸 때 이는 어쩔 수 없는 부분이다. 그래서 강결합 된 나쁜 케이스지만 타 개발자들과 얘기를 해야 하는 필수불가결한 측면이 있다. 

```js
const A = class {
  process(flag, v) {
    switch(flag) {
      case 1: return this.run1(v);
      case 2: return this.run2(v);
      case 3: return this.run3(v);
    }
  }
}

const B = class {
  constructor(a) {
    this.a = a;
  }
  noop() { this.a.process(1); }
  echo(data) {
    this.a.process(2, data);
  }
};
const b = new B(new A());
b.noop();
b.echo('test');
```

* 위의 경우 점원과 소비자 간의 메뉴판이라는 인터페이스를 떠올려 보면 된다. 메뉴 1번 메뉴 2번 메뉴 3번의 규칙이 정해져 있는 것이다. 그러나 이렇게 정해진 인터페이스가 있다는 건 메뉴 번호에 따라 실행시키는 switch 코드를 바꿀 수 없다는 걸 의미한다. 메뉴의 번호가 강결합된 것이다. 이걸 컨트롤 커플링이라고 한다.

```js
const A = class {
  add(data) {
    data.count++;
  }
}

const B = class {
  constructor(counter) {
    this.counter = counter;
    this.data = {a:1, count:0};
  }
  count() {
    this.counter.add(this.data);
  }
};
const b = new B(new A());
b.count();
b.count();
```

* count 키를 cnt로 바꾸면 모든 게 다 깨진다. 되도록이면 필요한 데까지만 데이터를 넘겨줘야 한다. 데이터의 카운터만 건너줬어야 하는데 더 넓은 범위를 넘겨주면 의존성이 생겨서 문제가 발생하는 것이다. 이를 스탬프 커플링이라고 한다.

```js
const A = class {
  add(count) {
    return count + 1;
  }
}

const B = class {
  constructor(counter) {
    this.counter = counter;
    this.data = {a:1, count:0};
  }
  count() {
    this.data.count = 
    this.counter.add(this.data.count);
  }
};
const b = new B(new A());
b.count();
b.count();
```

* 위의 예제는 데이터 커플링의 예제다. 