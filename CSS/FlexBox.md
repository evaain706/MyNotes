# Flexbox 정리

## 1. Flexbox란?
- CSS 1차원 레이아웃 시스템
- **컨테이너(Container)** + **아이템(Items)** 구조
- `display: flex` 또는 `display: inline-flex` 사용

---

## 2. 주요 개념
- **Main Axis (주 축)**: `flex-direction`에 따라 달라짐
- **Cross Axis (교차 축)**: 주 축에 수직 방향

---

## 3. Flex Container 속성
```css
.container {
  display: flex;
  flex-direction: row;        /* row | row-reverse | column | column-reverse */
  flex-wrap: nowrap;          /* nowrap | wrap | wrap-reverse */
  justify-content: flex-start;/* 주 축 정렬 */
  align-items: stretch;       /* 교차 축 정렬 */
  align-content: stretch;     /* 여러 줄 정렬 */
}
```
```