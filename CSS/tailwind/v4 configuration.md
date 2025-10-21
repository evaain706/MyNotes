


## TailwindV4부터의 Configuration


이전버전인 3.x까지 설정은 `tailwind.config.js` 에서 

``` jsx
module.exports = {
theme:{

}

}
```
위와 같은 방식으로 자바스크립트 형식으로 작성을하여 설정을 하였지만


V4부터는 CSS에서 직접설정이 가능하도록 변경되었다.
``` css
@import `tailwindcss`;

@theme{
 --font-sans: 'Simple', system-ui, sans-serif;
  --font-dospilgi: 'DOSPilgiMedium';
  --font-Hakgyoansim: 'HakgyoansimByeolbichhaneulTTF-B';
  --color-navy: #272f3d;
  --color-black: #02000e;
  --color-gray-100: #34383f;
}
```

---

`@layer base`
- 기본 HTML태그의 스타일들을 정의

`@layer components`
- 커스텀 유틸리티 클래스나 재사용가능한 컴포넌트스타일 정의

`@theme`
- v4에서 새로 추가된 CSS 커스텀 프로퍼티 기반 테마 변수 정의 영역


