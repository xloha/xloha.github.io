---
layout: post
title:  "(iOS) Border And Shadow, 테두리 둥글게, 그림자 적용하기"
date: 2020-11-17 16:00:00 +0900
categories: iOS swift 
tags:
- CollectionViewCell
- Shadow
- Border
---

카드 형태로 무언가를 만들려면 `border`값을 줘야하는 일이 생기는데 

이게 `shadow`와 같이 주려고 하면 생각보다 잘 안된다...ㅎㅅㅎ


`border`와 `shadow`는 많이 쓰이기 때문에 따로 extension으로 관리하는 것이 더욱 편리하다. 

<br>

## 최종 결과물 ~👻
<img src="/assets/image/basicCarouselBorder.png" style="zoom: 25%;"/>

<br>

❓ `Shadow와 Border`

✅  `shadow`와 `border`는 하나의 레이어에 같이 적용하지 못한다.

<br>

# 1. `Extension` 파일 생성 및 코드 작성

```swift
import UIKit

extension UICollectionViewCell {
    func setShadow(cornerRadius: CGFloat, shadowRadius: CGFloat, shadowOpacity: Float, shadowOffsetWidth: CGFloat, shadowOffsetHeight: CGFloat) {
        self.layer.masksToBounds = false
        self.layer.shadowColor = UIColor.black.cgColor
        
        self.layer.cornerRadius = cornerRadius
        self.layer.shadowRadius = shadowRadius
        self.layer.shadowOpacity = shadowOpacity
        self.layer.shadowPath = UIBezierPath(roundedRect:bounds, cornerRadius:contentView.layer.cornerRadius).cgPath
        self.layer.shadowOffset = CGSize(width: shadowOffsetWidth, height: shadowOffsetHeight)
    }
    
    func setBorderRound(cornerRadius: CGFloat) {
        self.contentView.layer.cornerRadius = cornerRadius
        self.contentView.layer.masksToBounds = true
    }
}
```

* `UIView`를 `extension`해도 되는데 나는 `CollectionViewCell`에서 사용할 것이라서 `CollectionViewCell`을 선택하였다.
    * **❗️여기서 주의할 점은 `shadow`와 `border`는 하나의 `layer`에 적용하지 못한다!!!**
    * 코드에서도 보이듯이 `shadow`는 자신의 `layer`에 적용해주었고, `border`는 `contentView의 layer`에 적용해준 것을 확인할 수 있다.

<br>

# 2. 적용해보기
이전에 만들어두었던 collectionView의 carousel에 적용해보자

```swift
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as! CollectionViewCell
        cell.imageView.image = UIImage(named: ColorImg.colorList[indexPath.row % ColorImg.colorList.count].img)
    
        cell.setBorderRound(cornerRadius: 10)
//        cell.setShadow(cornerRadius: 10, shadowRadius: 8, shadowOpacity: 0.1, shadowOffsetWidth: 2, shadowOffsetHeight: 0)
        return cell
    }
```

* 간단하게 이렇게 작성할 수 있다. 혹은 셀 내의 코드에서 작성해도 된다.
> shadow는 안어울려서 주석처리 했고 대충 요렇게 작성하면 된다!

<br>

# 3. 결과

<img src="/assets/image/basicCarouselBorder&Shadow.gif" style="zoom: 90%;"/>

동글동글하구 예뿌댬 *^___________^*

