#### 沒有公開，所以沒有網頁瀏覽

#### 先用 axios 抓資料
- 安裝 axios
```
npm i axios
```

- 抓資料，並執行pagination(jsonData,2)，顯示資料
```
import axios from 'axios';
const jsonData = ref([])
const getData = async()=>{
  const res = await axios.get('https://raw.githubusercontent.com/hexschool/KCGTravel/master/datastore_search.json')
  jsonData.value = res.data.result.records;
  pagination(jsonData,2)
}
```

#### 切割資料，方便做出頁碼
- 計算出總頁數，每頁要幾個資料
```
const page = ref({})
function pagination(jsonData, nowPage){

  // 取得所有資料長度
  const dataTotal = jsonData.value.length;

  // 設定每頁的資料數量
  const perpage = 4;

  // 計算出總頁數
  const pageTotal = Math.ceil(dataTotal / perpage);

  // nowPage 參數 =  現在的頁數
  let currentPage = nowPage;
  
  // 如果有頁數問題超出總頁數
  if(currentPage > pageTotal){
    currentPage = pageTotal;
  }

  // 計算出每頁第一筆，如第2頁，2*4 -4+1 = 5
  const minData = (currentPage * perpage) - perpage +1;

  // 計算出每頁的最後一筆
  const maxData = (currentPage * perpage);

  // 裝 那頁的資料
  const data =  ref([]);
  jsonData.value.forEach((item, index) => {
    const num = index +1;
    if ( num >= minData && num <= maxData){
      data.value.push(item)
    }
  });

  // 紀錄頁數資訊，那頁資料
  
  page.value = {
    pageTotal,
    currentPage,
    hasPage: currentPage >1,
    hasNext: currentPage < pageTotal,
    data,
  }
  // 將資料顯示
  displayData(data.value);
  // pageBtn(page);
}
```




- 上面的程式碼，可以得到
```
    總頁數，pageTotal,      
    目前頁數，currentPage,    
    往前頁，hasPage: currentPage >1,
    往後頁，hasNext: currentPage < pageTotal,
    每頁的資料，data,
```    

- 給 HTML 渲染資料
```
let perData = ref([])
function displayData(data){
  data.forEach((item)=>{
    perData.value.push(item)
  });
}
```

- 點擊按鈕，分成三個區塊，上一頁，最後一頁，中間數字頁
- 上一頁
```
  // 如果是 previous，在HTML有輸入 = 0
  if(num === 0 ){
    // 如果已經在第一頁時做的動作
    if (page.value.currentPage === 1) {
      pagination(jsonData, 1)
      page.value.currentPage = 1
      // console.log('what currentPage =  ', 0)
    }else{
      // 不是第一頁時做的動作
      // 頁數減一
      let p = page.value.currentPage - 1;
      // 顯示上一頁的資料
    pagination(jsonData, p)
    // 本頁數字改為上一頁
    page.value.currentPage = p
    }
```



- 最後一頁  
```  
    // 邏輯跟上一頁雷同
  }else if(num === page.value.pageTotal+1 && page.value.currentPage != page.value.pageTotal+1 ){
    let p = page.value.currentPage + 1;
    pagination(jsonData, p)
    page.value.currentPage = p
    if (page.value.currentPage > page.value.pageTotal) page.value.currentPage = page.value.pageTotal
```




- 中間數字頁    
  ```
  }else{
    // 就直接執行切換動作
    pagination(jsonData, num)
    page.value.currentPage = num
  }
  ```
- 一開始載入的時候就執行
```
onMounted(()=>{
  getData();
})
```


#### HTML 部分，分成兩區塊
- 資料內容，由每一頁的頁數，從所有的資料中，切割出內頁的資料。
- 頁數，要預防已經是第一頁了，還可以點選前一頁。最後一頁的邏輯相同。
```
if (page.value.currentPage === 1) {
      pagination(jsonData, 1)
      page.value.currentPage = 1
}
```





