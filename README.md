# wasm-chapter-4
기존 c++ 코드 베이스 재활용하기 편
- 출처 WebAssembly in Action 한빛 미디어, 제러드 갤런트 지음/ 이일웅 옮김

## 이 장의 요약
- 웹어셈블리의 진짜 장점은 코드를 재활용 하는 능력 -> 동일한 로직이 구현된 같은 코드를 데탑이든 브라우저든 같이 쓸수있기 때문!

- 사용예시: c++로 구현된 입력값 검증 로직이 있을때, 이를 엠스크립튼으로 컴파일 할 수 있도록 약간의 변화를 커친후, 컴파일해서, 이를 브라우저에서 처리하도록 할 수 있다. 이를통해 브라우저단에서도 일차적으로 데이터 검증이 가능하고, 서버에서도 동일한 로직으로 재점검할 수 있다.(같은 로직을 관리하는 것이 유지보수성이 더 좋다)

- [선형메모리에 대한 뜬금없는 메모](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9B%B9%EC%96%B4%EC%85%88%EB%B8%94%EB%A6%AC%EC%99%80%EC%9D%98-%EB%B9%84%EA%B5%90-%EC%96%B8%EC%A0%9C-%EC%9B%B9%EC%96%B4%EC%85%88%EB%B8%94%EB%A6%AC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EA%B2%8C-%EC%A2%8B%EC%9D%80%EA%B0%80-cf48a576ca3)

## __EMSCRIPTEN__이라는 조건부 컴파일 심볼

c/c++코드를 웹어셈블리 모듈로 만들려면 <emscripten.h> 헤더를 포함 시켜야한다. 그런데 엠스크립튼으로 컴파일 할때만 저 헤더를 포함하고 싶을 수 있다. 
그럴땐

```
#include <cstdlib>
#include <cstring>

#ifdef __EMSCRIPTEN__
#include <emscripten.h>
#endif
```

를 통해서 해결할 수 있다! [instellisense가 emscripten.h를 인식하지 못할때](https://mytutorials.tistory.com/179)

## 웹 어셈블리 함수를 두는 폼맷
* 참고: [네임 맹글링](https://5kyc1ad.tistory.com/343)

c++은 네임 맹글링이라는 것을 수행하는데, 이걸 해버리면 외부에서 함수를 호출할수가 없어진다. 

그래서 

```
#ifdef __cplusplus
extern "C"
{
#endifㄴ
    // 웹어셈블리 함수 두는곳
#ifdef __cplusplus
}
#endif
```

이 함수를 통해 특정 블록안의 코드는 네임 맹글링을 하지말라고 지정한다. 

`emcc validate.cpp -o validate.js -s EXTRA_EXPORTED_RUNTIME_METHODS=['ccall','UTF8ToString'] -s EXPORTED_FUNCTIONS=['_malloc','_free']`

['ccall','UTF8ToString'] 여기를 공백이 없도록 잘 써야한다

validate.cpp를 잘 작성후에 컴파일을 하고, html 파일은 예제 폴더에서 복붙했다. 
이제 해야할건, editproduct.js 라는 내가 컴파일해서 만들어낸 js, wasm 파일과 index.html을 연결해줄 징검다리 파일이 필요하다. 

## 모듈 메모리
웹어셈블리 모듈은 네가지 기본 자료형만 지원해서 문자열같이 복잡한건 모듈 메모리를 사용해야함. 엠스크립튼에 내장된 ccall는 문자열 메모리 관리를 도와준다. 

UTF8ToString을 이용하면 모듈 메모리에서 문자열을 읽을수있다. 
자세한건 나중에 더 이해해봐야겠다. 


## 실행
```
python3 -m venv .env
. .env/bin/activate
python -m http.server 8080
```


