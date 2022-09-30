# patch-package (react-native-image-modal)

앱 내에서 신고를 하거나 피드백을 보낼 때 이미지를 첨부할 수 있도록 했다. 그 때 쓰는 라이브러리가 `react-native-image-modal` 인데 이미지 첨부 후 모달에서 이미지 선택 시 **저화질로 노출되는 안드로이드 이슈**가 있었습니다.

> iOS에서는 정상동작, 안드로이드에서만 나타나는 현상이었음

한 참을 찾다보니 해당 라이브러리 깃헙 이슈탭에도 올라와있는 버그임을 확인했습니다. (개발자는 고칠 생각이 없는..)

![](https://velog.velcdn.com/images/rhfovk/post/d10f8711-f465-4f0c-a4ac-108db8da28ff/image.png)

그리고 그 밑에 달린 한 줄기의 빛과 같은 답변 ✨

![](https://velog.velcdn.com/images/rhfovk/post/7ce6b74d-374d-4997-94b1-035ab0d47621/image.png)

바로 node_modules에서 `imageDetail` ➡️ `index.js`로 접근하여 애니메이션과 스타일을 커스텀 진행했습니다.

### 위치

```jsx
react - native - image - modal > dist > ImageDetail > index.js;
```

### code

```jsx
var ImageDetail = (function (_super) {

  //...중략

  _this.close = function () {
            var _a = _this.props, isTranslucent = _a.isTranslucent, willClose = _a.willClose, onClose = _a.onClose;
            var windowHeight = Dimensions.get('window').height;
            if (isTranslucent) {
                StatusBar.setHidden(false);
            }
            setTimeout(function () {
                _this._isAnimated = true;
                if (typeof willClose === 'function') {
                    willClose();
                }
                Animated.parallel([
										// x , y 쪽은 애니메이션을 제거하고 opacity만 duration 조절
                    // Animated.timing(_this._animatedScale, { toValue: 1, useNativeDriver: false }),
                    // Animated.timing(_this._animatedPositionX, { toValue: 0, useNativeDriver: false }),
                    // Animated.timing(_this._animatedPositionY, { toValue: 0, useNativeDriver: false }),
                    Animated.timing(_this._animatedOpacity, { toValue: windowHeight, duration: 300, useNativeDriver: false }),
                    // Animated.spring(_this._animatedFrame, { toValue: 0, useNativeDriver: false }),
                ]).start(function () {
                    onClose();
                    _this._isAnimated = false;
                });
```

호기롭게 개발 환경에서 node_modules 해당 라이브러리 index.js에 접근하여 수정하여 이슈 resolved를 한 줄 알았는데..

<u>**이렇게 node_modules을 통해 직접 수정했을 경우 각 환경(dev, qa 등등)마다 다르게 됩니다.**</u>

```jsx
rm-rf node_modules //node_modules 삭제
yarn //재설치
```

그래서 빌드가 안되거나 다른 작업 환경의 변경 사항을 새롭게 저장할때 쓰는 위 명령어를 쓸 경우 index.js에서 작성했던 코드들이 모두 지워지고 패키지 코드를 다시 설치됩니다.

그래서 팀장님과 상의 후에 **patch-package** 를 사용하여 커스텀을 진행!

---

## patch-package

🔗 참고 링크 : [pathch-pakage](https://www.npmjs.com/package/patch-package)

![](https://velog.velcdn.com/images/rhfovk/post/65649195-b3d7-4abf-9055-d2b398a6656f/image.png)

`patch-package` 를 이용하면 node_modules 속 라이브러리를 커스텀한 상태가 배포 상태에서도 지속되도록 수정사항을 생성된 patch 파일에 기억해뒀다가 배포시 노드모듈 위에 덮어 씌워줍니다.

> 💡 이 후 다른 환경의 node_modules를 세팅할 때에 적용될 수 있음

## 설치 방법

```jsx
yarn add patch-pacakage postinstall-postinstall
```

설치 후에

```jsx
//package.json
"scripts": {
"postintall" : "patch-package"
}
```

package.json 파일에 추가

```jsx
yarn patch-package react-native-image-modal
```

해당 라이브러리 patch파일을 생성하기 위해 적은 것이고 뒤에 패키지 이름을 적으면 됩니다.

![](https://velog.velcdn.com/images/rhfovk/post/7e81d5b9-aabf-4d61-94c3-352f4c9b3916/image.png)

그럼 바로 **로컬의 변경 사항을 바탕으로 패치 파일이 생성됩니다**. node_modules 안의 특정 package의 수정사항이 patches 폴더안에 patch 파일로 자동 저장됩니다.

---

### 발견된 단점

patch-package를 사용할 시 매번 버전이 달라질 때마다 **patch 파일의 버전을 install 되어 있는 버전과 맞춰야하는 단점이 있습니다. ** 그러나 이러한 단점이 있음에도 빠르고 간편하게 노드모듈속 라이브러리를 수정 배포할 수 있다는 점에서 알아두면 좋은 대처법 중의 하나인 것 같습니다.
