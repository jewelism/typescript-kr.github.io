# 소개 (Introduction)

이 장에서는 TypeScript의 타입 추론(Type Inference)에 대해 설명합니다. 즉 타입이 어디서 어떻게 추론되는지 논의할 것입니다.

# 기본 (Basics)

TypeScript에는 타입을 명시해주지 않았을 때 타입 정보를 제공하기 위해 타입 추론이 사용되는 여러 위치가 있습니다. 아래 예시 코드를 봅시다.

```ts
let x = 3;
```

`x` 변수의 타입은 `number` 라고 추론됩니다.
이러한 종류의 추론은 변수와 멤버를 초기화하거나 매개 변수의 기본값을 설정하거나, 함수 반환 타입을 결정할 때 발생합니다.

대부분의 타입 추론은 간단합니다.
다음 장에서는 타입 추론의 미묘한 차이에 대하여 살펴보겠습니다.

# 가장 일반적인 타입 (Best common type)

여러 표현식에서 타입 추론이 이루어지면 그 표현식의 타입이 "가장 일반적인 타입"을 계산하는 데 사용됩니다. 예:

```ts
let x = [0, 1, null];
```

위의 예제에서 `x` 의 타입을 추론하기 위해서는 각 배열 요소의 타입을 반드시 고려해야합니다.
여기서는 배열의 타입에 대한 두가지 선택사항 `number` 와 `null` 이 주어집니다.
가장 일반적인 타입 알고리즘은 각 후보 타입을 고려하고 다른 호부와 호환되는 타입을 선택합니다.

제공되는 후보 타입 중에서 가장 일반적인 타입을 선택해야하기 때문에 타입이 공통 구조를 공유하지만 모든 타입의 수퍼 타입(상위 타입)이 하나도 없는 경우도 있습니다. 예:

```ts
let zoo = [new Rhino(), new Elephant(), new Snake()];
```

이상적으로, `zoo` 배열이 `Animal[]` 로 추론되기를 원할 수도 있습니다.
하지만 배열에는 정확히 `Animal` 타입의 객체가 없기 때문에 배열 요소 타입에 대한 추측을 할 수 없습니다.
이 문제를 해결하려면 다른 유형의 수퍼 타입이 없는 유형을 명시적으로 제공해야합니다:

```ts
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

가장 일반적인 타입이 발견되지 않으면 추론의 결과는 유니온 배열(Union Array) 타입인 `(Rhino | Elephant | Snake)[]` 가 됩니다.

# 상황적 타입 (Contextual Type)

타입 추론은 TypeScript의 "다른 방향"에서도 작동합니다.
이를 "상황적 타이핑(Contextual Typing)" 이라고 합니다. 상황적 타입은 표현식의 타입의 위치에 의해 암시될 때 발생합니다. 예:

```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.button);  //<- 오류
};
```

위의 코드에서 타입 오류를 제공하기 위해 TypeScript 타입 검사기는 `Window.onmousedown` 함수 타입을 사용하여 오른쪽 함수 표현식의 타입을 추론했습니다.
이렇게 했을 때 `mouseEvent` 매개 변수의 타입을 추론할 수 있었습니다.
이 함수 표현식이 문맥적으로 입력 된 위치에 있지 않으면 `mouseEvent` 매개변수는 `any` 타입을 가지며 오류는 발생하지 않습니다.

문맥적으로 타입이 정해진 표현식에 명시적인 타입 정보가 포함되어 있면 해당 타입이 무시됩니다.
위 예제를 작성했다면:

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.button);  //<- 이제 오류가 없습니다
};
```

매개 변수에 명시적 타입 주석이 있는 함수 표현식은 상황적 타입을 대체합니다.
일단 그렇게 되면 상황적 타입이 적용되지 않으므로 오류가 발생하지 않습니다.

상황적 타이핑은 많은 경우에 적용됩니다.
일반적인 경우에는 함수 호출에 대한 인수, 할당의 우측 표현식, 타입 표명(Type Assertions), 객체의 멤버와 배열 리터럴, 그리고 return 문입니다.
상황적 타입은 또한 가장 일반적인 타입의 후보 타입으로도 작용합니다.
예:

```ts
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

이 예에서 가장 일반적인 타입은 `Animal`, `Rhino`, `Elephant`, 그리고 `Snake` 네가지 집합으로 구성됩니다.
이 중에서 `Animal` 은 가장 일반적인 타입 알고리즘으로 선택이 가능합니다.