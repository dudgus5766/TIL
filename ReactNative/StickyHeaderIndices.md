# `stickyHeaderIndices`

---

**scrollView 또는 flatList 등의 스크롤리스트에서 스크롤시 특정 아이템 sticky 만들어줍니다.**

```
<Animated.ScrollView stickyHeaderIndices={[0}} ... >
	<Header />
	<Content />
</Animated.ScrollView>
```

- `{[0]}`은 첫 번째 하위 항목이 스크롤 보기 상단에 고정되도록 하게 해줍니다.
- `[x, y, z]`와 같은 것을 사용해 여러개 의 아이템이 위에 있을 때 도킹되도록 해줍니다.

> 💡 수평에는 지원하지 않음!

![](https://velog.velcdn.com/images/rhfovk/post/a0ad16b4-2ee3-4b5e-8e5b-41a21cb21ae0/image.png)

🔗 참고 링크 : [ScrollView 공식문서](https://reactnative.dev/docs/scrollview)
