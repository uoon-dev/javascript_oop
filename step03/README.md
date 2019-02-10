# HTML Parser

## HTML 형식
```
A = <TAG>BODY</TAG>
B = <TAG>/
C = TEXT
BODY = (A | B | C)N
```
* 확정적인 케이스를 짤 수 있는 개발자는 초급 개발자다. 재귀적이면서 복합적인 케이스를 짤 수 있는 개발자는 중급 개발자다. 

```html
<div>
  a
  <a>b</a>
  c
  <img/>
  d
</div>
```

```js
const textNode = (input, cursor, curr) => {
  const idx = input.indexOf('<', cursor);
  curr.tag.children.push({
    type:'text', text:input.substring(cursor, idx)
  });
  return idx;
}

const elementNode = (input, cursor, idx, curr, stack) => {
  const isClose = input[idx - 1] === '/';
  const tag = { name: input.substring(cursor + 1, idx - (isClose ? 1 : 0)), type: 'node', children: []};
  curr.tag.children.push(tag);
  if (!isClose) {
    stack.push({tag, back:curr});
    return true;
  }
  return false;
}

const parser = input => {
  input = input.trim();
  const result = { name: 'ROOT', type: 'node', children:[] };
  const stack = [{ tag: result }];
  let curr, i = 0, j = input.length;
  while(curr = stack.pop()) {
    while(i < j) {
      const cursor = i;
      if (input[cursor] === '<') {
        // A, B의 경우
        const idx = input.indexOf('>', cursor);
        i = idx + 1;
        if (input[cursor + 1] === '/') {
          curr = curr.back;
        } else {
          if(elementNode(input, cursor, idx, curr, stack)) break;
        }
      } else {
        // C의 경우
        i = textNode(input, cursor, curr);
      }
    }
  };
  return result;
}
```

* 동적계획은 루프를 결정하는 요인(조건)이 루프 내부에 있는 루프를 돌다가 결정될 때를 발생한다.
* 코드는 쉬운 코드부터 짜도록 한다. 왜냐하면 쉽다는 건 그만큼 의존성이 낮아지기 쉽기 때문이다. 따라서 나중에 얘에게 의존하는 애를 짜는 게 편하다. 