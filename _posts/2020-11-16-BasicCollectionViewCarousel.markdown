---
layout: post
title:  "(iOS) CollectionView로 Carousel만들기 (배너)"
date:   2020-11-16 15:00:00 +0900
categories: iOS swift 
tags:
- CollectionView
- scrollViewWillEndDragging
---
<br>

무려 한달만의 포스팅;;;
    > 일 배우느라 아요개발을 많이 못했당ㅠ_______ㅠ

앱에서 많이 쓰이는 배너형태 즉 `carousel`을 만들어보기로 했다. 여러가지 종류의 carousel이 있지만 그 중에서 가장 기본적인 형태의 배너를 포스팅해본다. 전체코드는 [EUNCAROUSEL](https://github.com/EunYeongKim/EUNCAROUSEL/tree/develop) 이곳에서 확인할 수 있다.  

[여기서](https://nsios.tistory.com/45) 도움을 많이 받았고, 코드를 참고하였다. 

<br>

## 만들고 싶었던 배너
앱 스토어에 아래와 같은 배너를 똑같이 만들어보고자 하였다.  

<img src="/assets/image/appstore_carousel.jpeg" style="zoom: 30%;"/>

<br>

## 최종 결과물 ~🎊
<img src="/assets/image/basic_carousel.png" style="zoom: 25%;"/> <img src="/assets/image/baiscCollectionViewCarousel.gif" style="zoom: 50%;"/>

<br>

❓ `scrollViewWillEndDragging`

✅  `scrollViewWillEndDragging` 메소드에서 셀 크기와 `Inset, lineSpacing,,,`을 계산하여 원하는 `offset`으로 스크롤을 멈출 수 있다. 

<br>

# 0. 이미지 배열 구조체 
```swift
struct ColorImg {
    var img: String
    
    init(img: String) {
        self.img = img
    }
    
    static var colorList: [ColorImg] {
        let array = Array(1...10)
        return array.map { ColorImg(img: "img_\($0)") }
    }
}
```

* `colorList` : `img_1, img_2 ... ` 이미지들의 이름을 가지고 있다. 

<br>

# 1. `CollectionView Setting`

<img src="/assets/image/basicColletionViewCarousel_collectionVIewSetting.png" style="zoom: 90%;"/> 

* 셀 사이즈는 고정으로 이미지의 크기와 같은 사이즈를 주었다. 
* 가로 스크롤이기 때문에 `lineSpacing` 을 10으로 주었다. 
* 제일 처음의 셀의 왼쪽, 제일 마지막 셀의 오른쪽의 여백을 10으로 주었다. 

<img src="/assets/image/basicCollectionVIewAttributeSetting.png"  style="zoom: 100%;"/>

* `CollectionView`의 `attribute`는 위와 같이 주면 된다.
    * `pageEnabled`는 반드시 `false`상태!!

<br>

# 2. `CollectionView Datasource, Delegate` 설정

```swift
class BasicCarouselViewController: UIViewController {
    
    @IBOutlet weak var collectionView: UICollectionView!
    
    let cellSize = CGSize(width: 300, height: 200)
    let insetSize: CGFloat = 10
    
    override func viewDidLoad() {
        super.viewDidLoad()
        configureCollectionView()
    }
    
    func configureCollectionView() {
        collectionView.contentInsetAdjustmentBehavior = .never
        collectionView.decelerationRate = .fast
    }
}

extension BasicCarouselViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return ColorImg.colorList.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as! CollectionViewCell
        cell.imageView.image = UIImage(named: ColorImg.colorList[indexPath.row % ColorImg.colorList.count].img)
        
        return cell
    }
}
```

<br>

# 3. `scrollViewWillEndDragging`을 이용하여 원하는 위치에서 스크롤 멈추기

```swift
extension BasicCarouselViewController: UIScrollViewDelegate {
    func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
        let cellWithSpacingWidth =  cellSize.width + lineSpacing
        var offset = targetContentOffset.pointee
        let index = (offset.x + scrollView.contentInset.left) / cellWithSpacingWidth
        let roundedIndex: CGFloat = round(index)

        offset = CGPoint(x: roundedIndex * cellWithSpacingWidth - (scrollView.contentInset.left + lineSpacing + insetSize), y: scrollView.contentInset.top)
        targetContentOffset.pointee = offset
    }
}
```
* `scrollViewWillEndDragging` : 사용자가 컨텐츠의 스크롤을 거의 끝낼 때즘 `delegate`에게 전달해준다.
* `cellWithSpacingWidth` : 셀의 넓이와 lineSpacing을 더한 하나의 셀 값을 구한다.
* `targetContentOffset.pointee` : 스크롤 정지될 때 예상되는 offset
* `(offset.x + scrollView.contentInset.left) / cellWithSpacingWidth` : 스크롤이 정지 될 offset과 스크롤 뷰 왼쪽의 inset값을 더한 후 셀 한 개의 넓이를 나누면 앞으로 보일 현재의 index값이 나옴

* `x: roundedIndex * cellWithSpacingWidth - (scrollView.contentInset.left + lineSpacing + insetSize), y: scrollView.contentInset.top)` : `(현재 인덱스 * 셀 한개의 넓이) - (스크롤 뷰 왼쪽 inset값 + 왼쪽에 이전 셀 노출할 넓이 값)` 으로 offset을 바꿔치기한다면 원하는만큼 이전 셀을 보일 수 있음


<br>


# 4. 결론
생각보다 간단한 코드로 내가 바라던 `carousel`을 구현할 수 있다.