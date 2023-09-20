---
layout  : wiki
title   : Spring, Vuejs 페이징 처리 구현
summary : 
date    : 2023-09-15 19:50:13 +0900
updated : 2023-09-20 15:03:08 +0900
tag     : spring vuejs
resource: 31/CE8A32-1369-404D-A682-85635CD5BA8A
toc     : true
public  : true
parent  : [[/Spring]] 
latex   : false
---
* TOC
{:toc}

## 개요

Spring은 페이징(Paging) 기능을 Pageable 인터페이스로 제공하여 쉽게 다룰 수 있도록 도와준다. 여기서 페이징은 무엇일까?

>> **페이징**은 메인 메모리에서 사용하기 위해 2차 기억장치로부터 데이터를 저장하고 검색하는 메모리 관리 기법으로 가상기억 장치를 모두 같은 크기의 블록으로 편성하여 운용하는 기법이다. 이 때 일정한 크기를 가진 블록을 페이지라고 한다.

이 페이징의 개념을 가져와 설명하자면 사용자가 어떤 데이터를 요청했을 때, 전체 데이터 중에 일부(페이지)를 가져와서 보여주는 것이다.

예를 들어 쇼핑몰이나 게시판에는 많은 데이터를 DB에 저장하고 있는데, 이 모든 데이터를 한번에 가져오도록 시도한다면 많은 시간과 메모리가 소요될 것이다. 이 때 데이터를 블록으로 잘라서 조금씩 가져온다면 해결될 것이다.

개념을 이해했다면, 인터페이스를 한번 살펴보자.

### Pageable 인터페이스

```java
public interface Pageable {

    ...
    
    int getPageNumber();

    int getPageSize();

    long getOffset();

    Sort getSort();
    
    Pageable next();

    Pageable previousOrFirst();

    Pageable first();

    ...
}
```

인터페이스의 구조는 위와 같은데 중요한 몇 가지만 살펴보면

- getPageNumber(): 현재 페이지 번호를 반환
- getPageSize(): 한 페이지에 보여줄 항목들의 갯수를 반환
- getOffset(): 페이즈 크기에 따른 오프셋을 반환
- next(), first(): 다음, 첫번째 페이지 조회 시 Pageabble 인터페이스 반환

이제 페이징 처리를 하려면 Pageable 인터페이스를 객체를 JpaRepository에 전달해야한다. Spring은 이 구현체를 생성하기 위한 클래스를 제공한다.

### PageRequest 클래스

```java
public class PageRequest extends AbstractPageRequest {

    ...

    public static PageRequest of(int page, int size) {
        return of(page, size, Sort.unsorted());
    }

    public static PageRequest of(int page, int size, Sort sort) {
        return new PageRequest(page, size, sort);
    }

    public static PageRequest of(int page, int size, Sort.Direction direction, String... properties) {
        return of(page, size, Sort.by(direction, properties));
    }
    
    ...
}
```

of() 정적 메서드로 정렬 조건과 속성이 적용된 새 항목을 만드는 메서드가 있다. 예를 들어

```java
Pageable pageable = PageRequest.of(0, 10, Sort.by(Direction.DESC, "id"));
```

위와 같이 인터페이스를 구현했다면, "id"필드로 정렬된 항목을 10개씩 하나의 페이지로 만들었을 때, 0번 째 페이지를 조회할 수 있는 Pagealbe 구현체를 만들게된다.

페이징 처리된 결과는 Page<T> 클래스에 담겨 반환되는데, 

```java
public interface Page<T> extends Slice<T> {
    static <T> Page<T> empty() {
        return empty(Pageable.unpaged());
    }

    static <T> Page<T> empty(Pageable pageable) {
        return new PageImpl(Collections.emptyList(), pageable, 0L);
    }

    int getTotalPages();

    long getTotalElements();

    <U> Page<U> map(Function<? super T, ? extends U> converter);
}
```

페이징 elements의 총량과 총 페이지 수를 반환하는 메서드를 포함하고 있다. 

이제 코드로 구현해보자.

## 백엔드 구현

### Service, Controller

```java
// ItemService.java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ItemService { 
    
    private final ItemRepository itemRepository;
    
    public Page<Item> findItems(Pageable pageable) {
        return itemRepository.findAll(pageable);
    }
}

// ItemApi.java
@RestController
@RequiredArgsConstructor
public class ItemApi {

    private final ItemService itemService;
    
    @GetMapping("/products")
    public ItemPageDto getItems(@PageableDefault(size = 3, sort = "id", direction = Sort.Direction.DESC) Pageable pageable) {
        return ItemPageDto.of(itemService.findItems(pageable));
    }
}
```

Service 코드에서는 Item Entity에 접근해 모든 상품을 조회하고 Page<Item> 형식으로 반환하도록하고, Controller에서 "/products" URL에서 데이터를 조회한다. 

@PageableDefault 애노테이션은 컨트롤러 메서드에 Pagealbe 구현체를 주입할 때 사용하며, 정렬 조건과 방향을 지정할 수 있다. 코드에서는 한 페이지당 3개의 데이터, 정렬 조건은 id, 방향은 내림차순으로 설정했다. 

이제 Spring은 API 앤드포인트 메서드에 Pageable 인터페이스가 패러미터에 있는 것을 확인하고, API 질의를 통해 전달받은 정보로 "/products?page=1&size=3&sort=id,desc" 조건으로 조회할 수 있는 Pagealbe 구현체를 제공한다.

### DTO

```java
// ItemDto.java
@Getter
@Builder
@AllArgsConstructor
public class ItemDto {

    private long id;

    private String name;

    private int price;

    private int stockQuantity;

    private String imagePath;

    public static ItemDto of(Item item) {
        return ItemDto.builder()
                .id(item.getId())
                .name(item.getName())
                .price(item.getPrice())
                .stockQuantity(item.getStockQuantity())
                .imagePath(item.getImagePath())
                .build();
    }
}

// ItemPageDto.java
@Getter
@Builder
@AllArgsConstructor
public class ItemPageDto {

    private List<ItemDto> elements;

    private long totalElements;

    private int currentPage;

    private int totalPages;

    public static ItemPageDto of(Page<Item> itemPage) {
        return ItemPageDto.builder()
                .elements(itemPage.getContent().stream().map(ItemDto::of).collect(Collectors.toList()))
                .totalElements(itemPage.getTotalElements())
                .totalPages(itemPage.getTotalPages())
                .currentPage(itemPage.getNumber())
                .build();
    }
}
```

ItemDto는 Item 엔티티를 DTO로 변환하고, ItemPageDto는 Page 객체를 DTO로 변환한다.

elements에는 조회할 페이지에 해당하는 데이터를 List형태로 담았고, 각각 총 데이터 수, 현재 페이지, 총 페이지수를 전달하도록 했다.

이제 클라이언트에서 이를 받을 수 있도록 설정해보자.

## 프론트엔드 구현

Vue와 tailwind를 사용해 프론트엔드를 구현해보자.

### API 호출

```javascript
// ItemPosts.js
export function fetchList(query) {
  return ItemRequest({
    url: '/products',
    method: 'get',
    params: query
  });
}

// ItemRequest.js
import axios from 'axios';

const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API,
  timeout: 5000
})

export default service;
```

ItemRequest.js는 axios로 API를 호출하는 코드이다. create로 HTTP 요청을 처리할 서비스 객체를 생성하도록 하고, ItemPosts에서는 전달받은 query를 ItemRequest 객체를 이용하여 호출하여 "/products" 경로로 GET 요청해 query가 패러미터값으로 전달된다.

### ProductsPage.vue

```
<template>
<ItemCard
      :itemList="itemList"
      :currentPage="page.page"
      :totalPages="totalPages"
      :pageChange="onPageChange"
    />
</template>

<script>
export default {
  name: "ProductsPage",
  components: { ItemCard },
  data() {
    return {
      itemList: [],
      totalPages: 0,
      page: {
        page: 0,
        size: 3,
        sort: "id,desc"
      },
    };
  },
  created() {
    fetchList(this.page).then((response) => {
      this.itemList = response.data.elements;
      this.totalPages = response.data.totalPages;
      this.page.page = response.data.currentPage;
    });
  },
  methods: {
    onPageChange(value) {
      this.page.page = value.requestPage;
      this.created();
    }
  },
}
</script>
```

Productspage를 router로 호출하면 fetchList 함수로 axios 요청을 보내 API 요청을 수행하고, 정상적인 응답을 받으면 컴포넌트의 상태(state)를 변경한다. 

상태를 변경하면 ItemCard 컴포넌트에 다음 props 정보를 전달한다.

- itemList: ItemDto의 elements
- currentPage: 현재 페이지
- totalPages : ItemDto의 총 페이지 수
- pageChange: 페이지 변경 이벤트

### ItemCard.vue

```
<template>
  <div class="grid gap-8 grid-cols-1 sm:grid-cols-2 md:grid-cols-3 mt-6">
    <div v-for="(item, index) in itemList" :key="'item-' + index" @click="goItemDetail(item.id)"
         class="w-full max-w-sm mx-auto rounded-md shadow-md overflow-hidden ml-4 mr-4">
      <div class="px-5 py-3">
        <h3 class="text-gray-700 uppercase">{{ item.name }}</h3>
        <span class="text-gray-500 mt-2">{{ item.price }} 원</span>
      </div>
    </div>
  </div>
  <Pagination :currentPage="currentPage" :totalPages="totalPages" :pageChange="pageChange" />
</template>

<script>
export default {
  name: "ItemCard",
  props: ["itemList", "currentPage", "totalPages", "pageChange"],
  components: { Pagination },
  methods: ...
};
</script>
```

ItemCard에서는 ProductsPage에서 전달받은 props 정보로 Item 화면을 구성한다.

itemList의 상품 정보를 v-for로 반복하여 상품의 이름과 가격을 표시하고, Pagination 컴포넌트에 현재 페이지, 총 페이지 수, 페이지 변경 이벤트 props를 전달한다.

### Pagination.vue

```
<template>
  <nav aria-label="Page navigation">
    <ul class="list-style-none flex justify-center">
      <li>
        <a @click="onPageChange(currentPage - 1)" class="relative block rounded bg-transparent px-3 py-1.5 text-sm text-neutral-600 transition-all duration-300 hover:bg-neutral-100 dark:text-white dark:hover:bg-neutral-700 dark:hover:text-white"> Previous </a>
      </li>
      <li v-for="(paging, index) in pages" :key="index">
        <a @click="onPageChange(paging - 1)" :class="paging - 1 === currentPage ? 'currentPage' : ''" class="relative block rounded bg-transparent px-3 py-1.5 text-sm text-neutral-600 transition-all duration-300 hover:bg-neutral-100 dark:text-white dark:hover:bg-neutral-700 dark:hover:text-white">
          {{ paging }}
        </a>
      </li>
      <li>
        <a @click="onPageChange(currentPage + 1)" class="relative block rounded bg-transparent px-3 py-1.5 text-sm text-neutral-600 transition-all duration-300 hover:bg-neutral-100 dark:text-white dark:hover:bg-neutral-700 dark:hover:text-white"> Next </a>
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: 'Pagination',
  props: ['currentPage', 'totalPages', 'pageChange'],
  data() {
    return {};
  },
  computed: {
    pages: function() {
      const list = [];
      for (let index = this.startPage; index <= this.endPage; index += 1) { list.push(index); }
      return list;
    },
    startPage() {
      return parseInt(this.currentPage / 5) * 5 + 1;
    },
    endPage() {
      let lastPage = parseInt(this.currentPage / 5) * 5 + 5;
      return lastPage <= this.totalPages ? lastPage : this.totalPages;
    }
  },
  methods: {
    onPageChange(val) {
      if (val < 0) {
        alert('첫 페이지입니다.');
        return;
      }
      if (val >= this.totalPages) {
        alert('마지막 페이지입니다.');
        return;
      }
      const param = {
        requestPage: val,
      };
      this.pageChange(param);
    }
  }
}
</script>
```

Pagination 컴포넌트는 페이징을 위한 숫자와 이전, 다음 버튼을 표시하고, 전달받은 Props 정보로 페이징 변경 시, 상위 컴포넌트로 페이지 번호와 페이지 이벤트 변경을 전달한다.

## 결과

<img width="1175" alt="스크린샷 2023-09-20 오후 2 56 44" src="https://github.com/Voyager003/toy-shoppingmall/assets/85725033/8e1b92cd-bd06-46f3-9072-5fbeee329418">

<img width="1196" alt="스크린샷 2023-09-20 오후 2 57 40" src="https://github.com/Voyager003/toy-shoppingmall/assets/85725033/848172e8-9162-4ac9-b6f9-f33aaff5ed39">

<img width="1201" alt="스크린샷 2023-09-20 오후 2 57 46" src="https://github.com/Voyager003/toy-shoppingmall/assets/85725033/5f9d54dc-a1be-4129-9b92-75ec2f769f3e">

<img width="1189" alt="스크린샷 2023-09-20 오후 2 57 55" src="https://github.com/Voyager003/toy-shoppingmall/assets/85725033/7b24eede-ff42-4587-9088-eff2852c5e12">

페이징을 3개로 설정했기 때문에, 4개 이상의 상품이 등록되면 2페이지가 생기고, 2페이지에서 이동을 한다면 마지막 페이지임을 알리는 경고를 확인할 수 있다.

페이지를 전환하게 되면

```java
http://localhost:8080/products?page=1&size=3&sort=id,desc 
```

브라우저에서는 위와 같은 GET 요청을 보내는 것을 확인할 수 있다.

## 참고자료

- https://ko.wikipedia.org/wiki/%ED%8E%98%EC%9D%B4%EC%A7%95
- https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html - PageRequest docs
- https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html - Page docs
- https://junhyunny.github.io/spring-boot/jpa/junit/jpa-paging/ - 강준현님의 블로그
- https://junhyunny.github.io/spring-boot/vue.js/spring-boot-vue-js-paging-table/


