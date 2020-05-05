# bounds

bounds 직사각형은 view의 위치와 크기를 자체 view의 좌표계 상에서 표현한다. bounds의 위치는 (0,0)이고 크기는 frame의 크기와 같지만 항상 같은 것은 아니다. 아래 그림과 같이 view가 회전할 경우 bounds의 크기가 frame의 크기와 다를 수 있다.

![](https://i.imgur.com/XiZx1lG.png)

bounds 변경 시 frame도 함께 변경되며, draw(:) 메서드 호출 없이 뷰를 재표시한다. bounds가 변경될 때 draw(:) 메서드를 호출하고 싶다면, contentMode를 redraw로 설정하면 된다.

<br>

# frame

frame 직사각형은 view의 위치와 크기를 superview의 좌표계 상에서 표현한다. view의 layout을 설정할 경우 frame을 사용한다. frame 설정 시 center와 bounds도 함께 변경된다.

frame이 변경되면 draw(:) 메서드 호출 없이 뷰를 재표시한다. frame이 변경될 때 draw(:) 메서드를 호출하고 싶다면, contentMode를 redraw로 설정하면 된다.

<br>

# center

center는 superview의 좌표계 상에서 정중앙 지점을 표현한다. view의 위치를 변경하고 싶을 경우 frame 대신 center를 사용하는 것이 가능하며, view의 크기 조정 및 회전 시에도 유효하다.

<br>

# References

- [bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds)
- [frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame)
- [center](https://developer.apple.com/documentation/uikit/uiview/1622627-center)
