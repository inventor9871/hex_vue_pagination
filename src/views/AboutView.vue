<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';

const jsonData = ref([])
const getData = async()=>{
  const res = await axios.get('https://raw.githubusercontent.com/hexschool/KCGTravel/master/datastore_search.json')
  // console.log(res)
  jsonData.value = res.data.result.records;
  // console.log(jsonData.value)
  pagination(jsonData,2)
}

const page = ref({})
function pagination(jsonData, nowPage){

  // 取得所有資料
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


// 給HTML渲染資料
let perData = ref([])
function displayData(data){
  data.forEach((item)=>{
    perData.value.push(item)
  });
}

// let pageArea = ref([])
// function pageBtn(page){
//   const total = page.pageTotal;
//   // if(page.hasPage)
// }
// console.log('page',page)

// 點擊按鈕
function switchPage(num){
// 分成三個區塊，上一頁，最後一頁，中間數字頁

//上一頁
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

// 最後一頁    
    // 邏輯跟上一頁雷同
  }else if(num === page.value.pageTotal+1 && page.value.currentPage != page.value.pageTotal+1 ){
    let p = page.value.currentPage + 1;
    pagination(jsonData, p)
    page.value.currentPage = p
    if (page.value.currentPage > page.value.pageTotal) page.value.currentPage = page.value.pageTotal

// 中間數字頁    
  }else{
    // 就直接執行切換動作
    pagination(jsonData, num)
    page.value.currentPage = num
  }
  // console.log('currentPage', page.value.currentPage)
}

onMounted(()=>{
  getData();
})
</script>
<template>
  <div class="row">

    <div class="col-md-3 py-2" v-for="x in page.data" :key="x">
      <div class="card">
        <div class="card bg-dark text-white text-left">
          <img class="card-img-top img-cover" height="155px" :src="x.Picture1">
          <div class="card-img-overlay d-flex justify-content-between align-items-end p-0 px-3" style="background-color: rgba(0, 0, 0, .2)">
            <h5 class="card-img-title-lg"> {{ x.Name }} </h5><h5 class="card-img-title-sm"> {{ x.Zone }} </h5>
          </div>
        </div>
        <div class="card-body text-left">
            <p class="card-text"><i class="far fa-clock fa-clock-time"></i> {{ x.Opentime }} </p>
            <p class="card-text"><i class="fas fa-map-marker-alt fa-map-gps"></i> {{ x.Add }}  </p>
            <p class="card-text"><i class="fas fa-mobile-alt fa-mobile"></i> {{x.Tel}} </p>
            <div >
            <p class="card-text"><i class="fas fa-tags text-warning"></i> {{ x.Ticketinfo }} </p>
          </div> 
        </div>
      </div>
    </div>
  </div>
  <div class="d-flex justify-content-center mt-4">
    <nav aria-label="Page navigation example">
      <ul  class="pagination" id="pageid" > 
      <li class="page-item" :class="{disabled: page.currentPage==1}" data-page="0" @click="switchPage(0)">
        <a class="page-link" href="#" data-page
        @click.prevent="">Previous</a></li>
      
      <li class="page-item" :class="{active: i==page.currentPage}" v-for="i in page.pageTotal" :key="i.id" :data-page="i" @click="switchPage(i)">
        <a class="page-link" href="#" :data-page="i">{{i}}</a></li>

      <li class="page-item " :class="{disabled: page.currentPage==page.pageTotal}" :data-page="page.pageTotal+1" @click="switchPage(page.pageTotal+1)">
        <a class="page-link" href="#" data-page="${Number(page.currentPage) + 1}">Next</a></li>
      </ul>
    </nav>
  </div>
</template>

<style>

</style>
