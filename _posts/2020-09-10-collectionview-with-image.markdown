---
layout: post
title:  "(iOS) CollectionView ImageView Cell 사이즈 조정하기"
date:   2020-09-10 17:20:00 +0900
categories: iOS swift 
tags:
- CollectioView
- Cell
- ImageView
---
<br>

[EUNVER](https://github.com/EunYeongKim/eunver/tree/develop) 프로젝트 진행 중 CollectionView를 사용해서 이미지 검색 결과를 나타낸다.

Cell의 size를 코드로 줘도 스크롤을 내리고 위로 다시 스크롤을 올리면 cell의 size가 지멋대로 변하는 문제가 발생했다. 

<br>

❓ CollectionView에서 cell의 사이즈 변경

✅ `sizeForItemAt`메소드에서 cell의 사이즈를 return해줌  

✅ ImageView의 `Size Inspector - Layout`에서 `Automatic` 대신 `Translates Mask Into Constraints` 로 변경

<br>

# 해결방법 1) Cell 사이즈를 코드로 설정

```swift
extension ThumbnailViewController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        guard let layout = collectionViewLayout as? UICollectionViewFlowLayout else { return CGSize.zero }
        var bounds = thumbCollectionView.bounds

        var width = bounds.width - (layout.sectionInset.left + layout.sectionInset.right)
        var height = bounds.height - (layout.sectionInset.top + layout.sectionInset.bottom)

        switch layout.scrollDirection {
        case .vertical:
            width = (width - (layout.minimumInteritemSpacing) * 1) / 2
            height = (height - (layout.minimumLineSpacing * 3)) / 4
        case .horizontal:
            width = (width - (layout.minimumLineSpacing * 2)) / 3
            height = (height - (layout.minimumInteritemSpacing * 3)) / 4
        }
        
        return CGSize(width: width.rounded(.down), height: height.rounded(.down))
    }
}
```

* 여기까지 설정해준 뒤 실행을 시키면 처음에는 정상적인 결과를 보이는 것 같지만, 스크롤을 내렸다가 위로 올리면 다음과 같은 결과가 발생한다.  

<img src="/assets/image/image-2020091000000001.png" alt="오류발생 이미지" style="zoom: 30%;" /> 

<br>
<br>

# 해결방법 2) Size Inspector - Layout 변경
<img src="/assets/image/image-2020091000000002.png" alt="Size Inspector" style="zoom: 150%;" /> 

* `Size Inspector`의 `Layout`에서 `Automatic` 대신 `Translates Mask Into Constraints` 로 변경하면 다음과 같이 원하는 대로 사이즈가 정해진 이미지가 제대로 잘 나온다.  

<img src="/assets/image/image-2020091000000003.png" alt="오류발생 이미지" style="zoom: 30%;" /> 



