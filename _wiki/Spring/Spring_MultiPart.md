---
layout  : wiki
title   : Spring, Vue에서 MultiPart를 이용한 파일 업로드 하기 
summary : 
date    : 2023-08-27 21:20:00 +0900
updated : 2023-08-30 23:48:43 +0900
tag     : java vuejs
resource: 82/55999D-3F90-4DB8-AD4A-FDE11BA5687B
toc     : true
public  : true
parent  : [[/Spring]]
latex   : false
---
* TOC
{:toc}

## 개요

웹에서는 이미지를 화면에 보여주는 일이 많은데, 이를 클라이언트와 서버에서 어떻게 처리하는지 정리했다.

아직 S3같은 스토리지를 사용하지 않고 로컬 환경에서 실행 중이라서 로컬 경로(PC)의 이미지를 화면에 띄우는 방법을 알아보자.

## 클라이언트 단

먼저 Vue에서 전달하는 데이터를 살펴보자.

```
<template>
  <section class="py-12 bg-white sm:py-16 lg:py-20">
    <div class="px-4 mx-auto sm:px-6 lg:px-8 max-w-7xl">
      <div class="max-w-md mx-auto text-center mb-8">
        <h2 class="text-2xl font-bold text-gray-900 sm:text-3xl">상품 등록</h2>
      </div>

      <form class="max-w-md mx-auto space-y-6" @submit.prevent="registerItem">
        <div>
          <label for="product-name" class="block text-sm font-medium text-gray-700">상품 명</label>
          <input v-model="name" type="text" id="product-name" :class="{ 'border-red-500': showErrors && !isNameValid }" class="mt-1 block w-full p-2.5 border rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
          <p v-if="showErrors && !isNameValid" class="mt-1 text-sm text-red-500">상품 명을 입력해주세요.</p>
        </div>

        <div>
          <label for="product-price" class="block text-sm font-medium text-gray-700">상품 가격</label>
          <input v-model="price" id="product-price" :class="{ 'border-red-500': showErrors && !isPriceValid }" class="mt-1 block w-full p-2.5 border rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
          <p v-if="showErrors && !isPriceValid" class="mt-1 text-sm text-red-500">1000원 이상 50000원 이하의 가격을 입력해주세요.</p>
        </div>

        <div>
          <label for="product-stock" class="block text-sm font-medium text-gray-700">상품 재고</label>
          <input v-model.number="stockQuantity" type="number" id="product-stock" :class="{ 'border-red-500': showErrors && !isStockQuantityValid }" class="mt-1 block w-full p-2.5 border rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
          <p v-if="showErrors && !isStockQuantityValid" class="mt-1 text-sm text-red-500">최소 1개 이상 10개 이하의 재고를 입력해주세요.</p>
        </div>

        <div>
          <label for="product-image" class="block text-sm font-medium text-gray-700">사진 업로드</label>
          <input type="file" id="product-image" v-on:change="uploadImage" ref="imageFile" class="mt-1 block w-full p-2.5 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
        </div>

        <div>
          <label for="category" class="block text-sm font-medium text-gray-700">카테고리 선택</label>
          <select v-model="category" id="category" class="mt-1 block w-full p-2.5 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
            <option value="album">앨범</option>
            <option value="book">책</option>
            <option value="movie">영화</option>
          </select>
        </div>

        <div v-if="category === 'album'">
          <label for="artist" class="block text-sm font-medium text-gray-700">아티스트</label>
          <input v-model="categoryDetail" type="text" id="categoryDetail" :class="{ 'border-red-500': showErrors && !isCategoryDetailValid }" class="mt-1 block w-full p-2.5 border rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
          <p v-if="showErrors && category === 'album' && !isCategoryDetailValid" class="mt-1 text-sm text-red-500">아티스트를 입력해주세요.</p>
        </div>

        <div v-if="category === 'book'">
          <label for="author" class="block text-sm font-medium text-gray-700">저자</label>
          <input v-model="categoryDetail" type="text" id="categoryDetail" :class="{ 'border-red-500': showErrors && !isCategoryDetailValid }" class="mt-1 block w-full p-2.5 border rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
          <p v-if="showErrors && category === 'book' && !isCategoryDetailValid" class="mt-1 text-sm text-red-500">저자를 입력해주세요.</p>
        </div>

        <div v-if="category === 'movie'">
          <label for="director" class="block text-sm font-medium text-gray-700">감독</label>
          <input v-model="categoryDetail" type="text" id="categoryDetail" :class="{ 'border-red-500': showErrors && !isCategoryDetailValid }" class="mt-1 block w-full p-2.5 border rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
          <p v-if="showErrors && category === 'movie' && !isCategoryDetailValid" class="mt-1 text-sm text-red-500">감독을 입력해주세요.</p>
        </div>

        <div class="flex justify-end">
          <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 focus:ring-2 focus:ring-blue-300">
            상품 등록
          </button>
        </div>
      </form>
    </div>
  </section>
</template>

<script>
...

data() {
    return {
      name: "",
      price: null,
      stockQuantity: null,
      category: "album",
      categoryDetail: "",
      imageFile: null,
      showErrors: false
    };
  },
  
  ...
  
  // 이미지 업로드
  methods: {
    uploadImage() {
      this.imageFile = this.$refs.imageFile.files[0];
    },

    ...
    
    
    formData.append('image', this.imageFile);
    formData.append('request', new Blob([JSON.stringify(requestData)], { 
        type: 'application/json' 
    }));

      try {
        const response = await axios.post('/products/register', formData, {
            headers: {
              Authorization: `Bearer ${token}`,
              "Content-Type": `multipart/form-data`,
            },
          }
        );

        if (response.status === 200) {
          const itemId = response.data;
          useProductStore().setProductDetails({
            name: this.name,
            price: this.price,
          });
          await router.replace(`/products/${itemId}`);
          alert('상품이 등록되었습니다.');
        }
    }
    ...
```

먼저 template에서 전달하는 데이터를 바인딩해야한다. 상품 명과 가격, 재고, 이미지 등를 초기화한다.

여기서 컨텐츠 타입을 보면 multipart/form-data라고 명시되어있다.

파일을 업로드 할 때, 이미지 설명을 위한 input인 text와 이미지 파일을 위한 input인 file을 함께 전송해야 한다.

즉, 하나의 요청에 컨텐츠 타입이 서로 다른 것이 2개가 있는 것이다. 

이런 경우 두 종류의 데이터가 하나의 HTTP Request Body에 담겨야하는데, 때문에 데이터를 구분하여 넣어주는 방법도 필요해졌다. 

그래서 등장한 것이 Multipart 타입이다. Formdata는 ajax로 form 전송을 가능하게 해주는 객체로 key-value 구조로 데이터를 전송한다.

코드에서는 formdata에 이미지와 요청 데이터를 담아서 전송하고 있다.

## 서버 단

이제 서버에서 요청을 받아야 한다.

```java
@RestController
@RequiredArgsConstructor
public class ItemApi {

    private final ItemService itemService;

    @PostMapping("/products/register")
    @PreAuthorize("hasRole('SELLER')")
    public ResponseEntity<?> registerItem(@RequestPart(name = "request") @Valid ItemRequest request,
                                          @RequestPart(name = "image", required = false) MultipartFile imgFile) throws IOException {
        Long itemId = itemService.registerItem(request, imgFile);
        return ResponseEntity.ok(itemId);
    }
}
```

regitserItem() 메서드의 패러미터를 보면 @RequestPart 애노테이션을 볼 수 있다.

보통 컨트롤러에서 요청을 받을 때 JSON 형식으로 전달받아 @RequestBody로 전달받지만, formData로 전달한 데이터를 받기 때문에 @RequestPart를 명시한다. 설명을 살펴보면

>> Annotation that can be used to associate the part of a "multipart/form-data" request with a method argument.
Supported method argument types include MultipartFile in conjunction with Spring's MultipartResolver abstraction, javax.servlet.http.Part in conjunction with Servlet 3.0 multipart requests, or otherwise for any other method argument, the content of the part is passed through an HttpMessageConverter taking into consideration the 'Content-Type' header of the request part. This is analogous to what @RequestBody does to resolve an argument based on the content of a non-multipart regular request.

“multipart/form-data” 요청의 일부를 메소드 인수와 연결하는 데 사용할 수 있는 어노테이션이다. 라고 설명되어있다.

이제 서비스 코드에서 이미지를 생성하는 코드를 작성해야한다.

```java
@Transactional
    public Long registerItem(ItemRequest request, MultipartFile imgFile) throws IOException {
        Item newItem = createItem(request, imgFile);
        itemRepository.save(newItem);
        ...
        
        }
    }
    
    ...

private Item createItem(ItemRequest request, MultipartFile imgFile) throws IOException {
        String category = request.getCategory();
        String categoryDetail = request.getCategoryDetail();

        String imgPath = generateImageUrl(imgFile);
    }
...


private String generateImageUrl(MultipartFile file) throws IOException {
        if (file.isEmpty()) {
            throw new IllegalArgumentException("File is empty");
        }

        String fileName = generateUniqueFileName(file.getOriginalFilename());
        File targetFile = new File(uploadPath, fileName);
        file.transferTo(targetFile);
        return fileName;
    }

    private String generateUniqueFileName(String originalFilename) {
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        return System.currentTimeMillis() + "_" + UUID.randomUUID().toString() + extension;
    }
}
```

MultipartFile을 패러미터로 받아 무작위 파일명을 생성하고, 파일명을 반환하도록 했다.

반환된 파일명과 DTO로 전달받은 데이터로 Item 객체를 생성하고 DB에 저장한다.

## 외부 경로 생성

보통 보안상의 이유로 로컬에 있는 파일은 브라우저에서 보안상 접근이 불가능하지만, Spring에서는 외부 경로에 있는 리소스에 접근할 수 있는 방법을 제공한다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Value("${upload.handler}")
    private String uploadPath;

    @Value("${upload.resource}")
    private String resourcePath;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler(uploadPath)
                .addResourceLocations(resourcePath);
    }
}
```

WebMvcConfigurer 인터페이스를 구현하고 addResourceHandlers()를 오버라디딩했다.

이는 application.yml에 설정한 uploadPath와 resourcePath를 참조하여 요청을 전달한다.

>>  resource: file:///Users/voyager3/uploadImage/ (MAC 폴더의 경로)
>>  handler: /products/**

위와 같이 설정했다면, 요청이 'http://localhost:8080/products' 로 시작되는 경우, resourcePath경로에 있는 파일을 찾아 응답하게 된다.

```
      <div class="lg:w-4/5 mx-auto flex flex-wrap">
        <img alt="ecommerce" class="lg:w-1/2 w-full object-cover object-center rounded border border-gray-200" :src="'/products/' + productDetails.imageUrl" />
        <div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
          <h1 class="text-gray-900 text-3xl title-font font-medium mb-1"> {{ productDetails.name }} </h1>
          <div class="flex mb-4">
          ...
```

이제 클라이언트에서 :src 태그를 통해 서버에 있는 이미지를 찾아 업로드한 이미지를 확인할 수 있다.

## 참고자료

- https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/POST 
- https://kim-jong-hyun.tistory.com/28 - Spring 로컬 경로 이미지 불러오기
