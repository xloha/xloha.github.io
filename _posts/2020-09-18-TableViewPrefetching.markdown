---
layout: post
title:  "(iOS) TableView Prefetching"
date:   2020-09-18 16:30:00 +0900
categories: iOS swift 
tags:
- Table View
- Prefetching
---
<br>

Naver 영화 검색 OpenAPI를 사용하였는데, 최대 100개의 결과를 가져올 수 있었기 때문에 100개 이상의 결과가 return될 경우 모든 검색 결과를 보여주고 싶어서 여러 방법을 찾다가 Prefetch라는 기능을 알게되었고 좋은 것 같아서 사용해보았다.  

Prefetch를 적용하는 것은 생각보다는 간단했는데, 처음에 적용하려고 했을 때 prefetch에 대한 개념을 잘못이해해서 그런지 살짝 헤멨다! 

<br>

❓ TableView Prefetching이란?

✅ Row가 display 영역 근처에 있을 경우 불리게 되는데 cell을 준비하는 것이 아니라  cell이 표시되기 전에 **cell을 구성할 datasource를 준비할 수 있도록 하는 함수**

<br>

# TableViewCell의 Life Cycle

**`awakeFromNib` -> `cellForRowAt` -> `willDisplay` -> `prefetch`**

```swift
class CustomCell: UITableViewCell {
  override func awakeFromNib() {
    super.awakeFromNib()
  }
  override func prepareForReuse() {
    super.prepareForReuse()
  }
  deinit {

  }
}
```

* 프로젝트 실행 시 셀마다 다음의 순서대로 함수가 불리게 됨

  **`awakeFromNib` -> `cellForRowAt`(셀들이 메모리에 로드) -> `willDisplay`(화면에 보여지기 직전에 호출)**

* 화면에 보여지는 모든 셀들이 위의 과정을 거치고, 앞으로 불리게 될 셀들의 정보를 알려주는 `prefetch`가 실행됨 

<br>

# TableView Prefetching

![prefetchRows](/assets/image/prefetchRows.png)  

* Row가 display 영역 근처에 있을 경우 불리게 되는데 cell을 준비하는 것이 아니라  cell이 표시되기 전에 **cell을 구성할 datasource를 준비할 수 있도록 하는 함수**임

  ```swift
  func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) { ... }
  ```

  * `prefetchRowsAt` 메소드는 tableView가 화면에 로드되고 셀을 로드할 준비가 끝나면 호출되서 **앞으로 보여지게 될 cell들을 indexPaths에 담아줌**
  * prefetch기술은 adaptive technology로 **자동으로 호출시점이 조정**되서, 빠른 스크롤링을 할 경우 prefetchRowsAt 메소드를 완전히 호출하지 않을 수도 있음

* **`Datasource를 미리 준비하는 것`** 이기 때문에 실제 display cell 할 때 부하가 걸리는 동작(고해상도 이미지를 imageView에 set하는데 한 화면에 여러 cell이 표시되는 경우)은 display cell의 동작을 개선해야 함

* 모든 아이템에 대한 datasource를 미리 가지고 있는 경우에는 사용할 필요 X 

  * **데이터를 동적으로 로딩하는 경우에, 그 동작이 preload를 통해 개선될 수 있는 경우에 사용하는 것이 좋음**

<br>

# Prefetch 적용해보기

**[ MovieViewController.swift ]**

```swift
class MovieViewController: UIViewController {
    private var loadMoreFlag: Bool = false
    private let displayCount: Int = 100
    private var itemStartIdx: Int = 0
    private var movies: MovieSearchResult = MovieSearchResult()
    
    @IBOutlet weak var movieTableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
      // 현재의 뷰 컨트롤러를 PrefetchDataSource로 지정
        movieTableView.prefetchDataSource = self
    }
    
    func loadMovies(queryString: String) {
        if !self.loadMoreFlag {
            self.itemStartIdx = 0
            MovieService.movieSearchList(queryString: queryString, country: self.country, start: 1, display: self.displayCount) { result in
                self.movies = result
                self.movieTableView.setContentOffset(.zero, animated: true)
                if let movieDisplay = self.movies.display {
                    self.itemStartIdx += movieDisplay
                }
                self.movieTableView.reloadData()
                self.loadMoreFlag = true
            }
        } else {
            guard movies.items.count - 1 < self.itemStartIdx else { return }
            MovieService.movieSearchList(queryString: self.queryString, country: self.country, start: self.itemStartIdx + 1, display: self.displayCount) { result in
                self.movies.items.append(contentsOf: result.items)
                if let movieDisplay = self.movies.display {
                    self.itemStartIdx += movieDisplay
                }
                self.movieTableView.reloadData()
            }
        }
    }
}

extension MovieViewController: UITableViewDataSourcePrefetching {
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        for indexPath in indexPaths {
          // 배열이 가지고 있는 마지막 인덱스를 보여달라고 할 경우 통신을 통해 데이터를 더 가져옴
            if indexPath.row == self.itemStartIdx - 1 {
                loadMovies(queryString: self.queryString)
                break
            }
        }
    }
}

extension MovieViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return movies.items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "movieCell", for: indexPath) as! MovieTableViewCell
        
        let currentCellMovie = self.movies.items[indexPath.row]
        cell.movieData = currentCellMovie
        
        return cell
    }  
}
```

* `prefetchRowsAt함수`에서 리스트의 마지막 인덱스를 보여달라고 할 경우 통신을 통해서 데이터를 더 가져오도록 구현하였다. 

<br>

CollectionView에도 위와 거의 비슷한 로직으로 Prefetching을 적용했는데, 전체 코드는 다음에서 확인할 수 있다.

[EUNVER 깃허브 : https://github.com/EunYeongKim/EUNVER](https://github.com/EunYeongKim/EUNVER)