<script setup>
import { onMounted, ref } from 'vue';

const jsonUrl = 'https://raw.githubusercontent.com/hexschool/KCGTravel/master/datastore_search.json';
// const content = document.getElementById('content');
// const pageid = document.getElementById('pageid');
let jsonData = ref();
import axios from 'axios';


const getData = async()=>{
  const res = await axios.get(jsonUrl)
  // console.log(res.data.result.records)
  jsonData.value = res.data.result.records;
  pagination(jsonData, 1)
}

onMounted(()=>{
//   fetch(jsonUrl, {method: 'get'})
//   .then((response) => {
//   return response.json();
// }).then((data) => {
//   jsonData = data.result.records;
//   console.log(jsonData)
  // pagination(jsonData, 1);
// })
  getData();
  
})

function pagination(jsonData, nowPage) {
  console.log(nowPage);
  // 取得全部資料長度
  const dataTotal = jsonData.value.length;
  
  // 設定要顯示在畫面上的資料數量
  // 預設每一頁只顯示 20 筆資料。
  const perpage = 3;
  
  // page 按鈕總數量公式 總資料數量 / 每一頁要顯示的資料
  // 這邊要注意，因為有可能會出現餘數，所以要無條件進位。
  const pageTotal = Math.ceil(dataTotal / perpage);
  console.log('pageTotal', pageTotal)
  // 當前頁數，對應現在當前頁數
  let currentPage = nowPage;
  
  // 因為要避免當前頁數筆總頁數還要多，假設今天總頁數是 3 筆，就不可能是 4 或 5
  // 所以要在寫入一個判斷避免這種狀況。
  // 當"當前頁數"比"總頁數"大的時候，"當前頁數"就等於"總頁數"
  // 注意這一行在最前面並不是透過 nowPage 傳入賦予與 currentPage，所以才會寫這一個判斷式，但主要是預防一些無法預期的狀況，例如：nowPage 突然發神經？！
  if (currentPage > pageTotal) {
    currentPage = pageTotal;
  }
  
  // 由前面可知 最小數字為 6 ，所以用答案來回推公式。
  const minData = (currentPage * perpage) - perpage + 1 ;
  const maxData = (currentPage * perpage) ;
  
  // 先建立新陣列
  const data = ref([]);
  // 這邊將會使用 ES6 forEach 做資料處理
  // 首先必須使用索引來判斷資料位子，所以要使用 index
  jsonData.value.forEach((item, index) => {
    // 獲取陣列索引，但因為索引是從 0 開始所以要 +1。
    const num = index + 1;
    // 這邊判斷式會稍微複雜一點
    // 當 num 比 minData 大且又小於 maxData 就push進去新陣列。
    if ( num >= minData && num <= maxData) {
      data.value.push(item);
    }
  })
  // console.log('data = ', data)

  // 用物件方式來傳遞資料
  const page = {
    pageTotal,
    currentPage,
    hasPage: currentPage > 1,
    hasNext: currentPage < pageTotal,
  }
  displayData(data.value);
  pageBtn(page);
}

// 方法一

// let str1 = ref('');
// function displayData(data) {
  
//   data.forEach((item) => {
//     str1.value += `<div class="col-md-3 py-2">
//       <div class="card">
//         <div class="card bg-dark text-white text-left">
//           <img class="card-img-top img-cover" height="155px" src="${item.Picture1}">
//           <div class="card-img-overlay d-flex justify-content-between align-items-end p-0 px-3" style="background-color: rgba(0, 0, 0, .2)">
//             <h5 class="card-img-title-lg">${item.Name}</h5><h5 class="card-img-title-sm">${item.Zone}</h5>
//           </div>
//         </div>
//         <div class="card-body text-left">
//             <p class="card-text"><i class="far fa-clock fa-clock-time"></i>&nbsp;${item.Opentime}</p>
//             <p class="card-text"><i class="fas fa-map-marker-alt fa-map-gps"></i>&nbsp;${item.Add}</p>
//             <p class="card-text"><i class="fas fa-mobile-alt fa-mobile"></i>&nbsp;${item.Tel}</p>
//           ${ item.Ticketinfo ?
//     `<div>
            
//             <p class="card-text"><i class="fas fa-tags text-warning"></i>&nbsp;${item.Ticketinfo}</p>
//           </div>` : ''}
//         </div>
//       </div>
//     </div>`;
//   });
//   // content.innerHTML = str;
// }

// 方法二
let str1 = ref([]);
function displayData(data) {
  
  data.forEach((item) => {
    str1.value.push(item)
  });

}

let str = ref('');
function pageBtn (page){
  str.value = ''
  
  const total = page.pageTotal;
  
  if(page.hasPage) {
    str.value += `<li class="page-item"><a class="page-link" href="#" data-page="${Number(page.currentPage) - 1}">Previous</a></li>`;
  } else {
    str.value += `<li class="page-item disabled"><span class="page-link">Previous</span></li>`;
  }
  

  for(let i = 1; i <= total; i++){
    if(Number(page.currentPage) === i) {
      str.value +=`<li class="page-item active"><a class="page-link" href="#" data-page="${i}">${i}</a></li>`;
    } else {
      str.value +=`<li class="page-item"><a class="page-link" href="#" data-page="${i}">${i}</a></li>`;
    }
  };

  if(page.hasNext) {
    str.value += `<li class="page-item"><a class="page-link" href="#" data-page="${Number(page.currentPage) + 1}">Next</a></li>`;
  } else {
    str.value += `<li class="page-item disabled"><span class="page-link">Next</span></li>`;
  }

}

function switchPage(e){
  e.preventDefault();
  str1.value = [];
  if(e.target.nodeName !== 'A') return;
  const page = e.target.dataset.page;
  pagination(jsonData, page);
}

</script>

<template>
  <h3>作業條件：</h3>
  <ol>
    <li>將以下範例改為 Vue 的版本</li>
    <li>卡片、分頁要製作成 “元件”</li>
    <li>分頁要善用 props 及 emits 的觀念</li>
  </ol>

  <div class="row" id="content">
      <!-- 方法一 -->
      <!-- <div  v-html="str1"></div> -->

      <!-- 方法二 -->
    <div class="col-md-3 py-2" v-for="item in str1" :key="item.id">
      <div class="card">
        <div class="card bg-dark text-white text-left">
          <img class="card-img-top img-cover" height="155px" :src="item.Picture1">
          <div class="card-img-overlay d-flex justify-content-between align-items-end p-0 px-3" style="background-color: rgba(0, 0, 0, .2)">
            <h5 class="card-img-title-lg">{{item.Name}}</h5><h5 class="card-img-title-sm">{{item.Zone}}</h5>
          </div>
        </div>
        <div class="card-body text-left">
            <p class="card-text"><i class="far fa-clock fa-clock-time"></i>&nbsp;{{item.Opentime}}</p>
            <p class="card-text"><i class="fas fa-map-marker-alt fa-map-gps"></i>&nbsp;{{item.Add}}</p>
            <p class="card-text"><i class="fas fa-mobile-alt fa-mobile"></i>&nbsp;{{item.Tel}}</p>
            <div v-if="item.Ticketinfo">
            <p class="card-text"><i class="fas fa-tags text-warning"></i>{{ item.Ticketinfo}}</p>
          </div> 
        </div>
      </div>
    </div>
    
  </div>

  <div class="d-flex justify-content-center mt-4">
    <nav aria-label="Page navigation example">
      <!-- <ul class="pagination" id="pageid" 
      @click="switchPage" 
        > -->
        <!-- {{ str }} -->
          <ul v-html="str" class="pagination" id="pageid" @click="switchPage"></ul>
      <!-- </ul> -->
    </nav>
  </div>
</template>
<style>
.img-cover {
  object-fit: cover;
}
nav{ display: flex;}
.pagination { display: flex;}
</style>