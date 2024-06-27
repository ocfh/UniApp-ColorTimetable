<script setup lang="ts">
const { customBarHeight, statusBarHeight } = useAppStore()
const showImage = () => {
  uni.previewImage({
    urls: ["helper.jpg"],
    current: 0
  });
};  

import { ref, watch } from 'vue';
import { useRouter } from 'vue-router';
const classes = [
{ value: 'courses', name:'测试班级'}
];
const searchText = ref('');
const selectedClass = ref(null);
const filteredClasses = ref(classes);
const selectedClassTitle = ref(null);
const cls = localStorage.getItem('selectedClass');
watch(searchText, (val) => {
  const text = val.trim().toLowerCase();
  if (!text) {
    filteredClasses.value = classes;
  } else {
    filteredClasses.value = classes.filter((cls) => cls.name.toLowerCase().includes(text));
  }
});
const router = useRouter();
onMounted(() => {
  restoreSelectedClass();
  goToIndexPage();
  fetchData();
  yiyan();
});
function handleClick(cls) {
  if (selectedClass.value === cls) {
    selectedClass.value = null;
  } else {
    selectedClass.value = cls;

  }
}
function restoreSelectedClass() {
  const cls = localStorage.getItem('selectedClass');
    selectedClass.value = JSON.parse(cls);
}
function handleEnter() {
  if (filteredClasses.value.length > 0) {
    selectedClass.value = filteredClasses.value[0];
  } else {
    selectedClass.value = null;
  }
}
function goToIndexPage() {
  if (!selectedClass.value || !selectedClass.value.value) {
    return;
  }
  const currentDate = new Date();  
  const date = `${currentDate.getFullYear()}${String(currentDate.getMonth() + 1).padStart(2, '0')}${String(currentDate.getDate()).padStart(2, '0')}`;  
  localStorage.setItem('selectedClass', JSON.stringify(selectedClass.value));
  uni.navigateTo({
    url: `/pages/day/${selectedClass.value.value}?${date ? `date=${date}` : 'no'}`,
  })
}
function clearSelectedClass() {
  localStorage.removeItem('selectedClass');
  uni.navigateTo({
    url: '/pages/index/index',
  })
}
function fetchData() {  
    return fetch('https://api.seniverse.com/v3/weather/now.json?key=[seniverse.key]&location=ip')  
        .then(response => response.json())  
        .then(data => {  
            const date =  new Date();
			const formattedDate = date.getFullYear() + '年' + (date.getMonth()+1) + '月' + date.getDate() + '日';
			const weekday = date.getDay(); // 使用getDay()方法获取星期几  
			const weekdays = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"]; // 一周的七天  
            const weather = data.results[0].now.text; 
            const temperature = data.results[0].now.temperature;  
            const formattedInfo = `${formattedDate} ${weekdays[weekday]} ${weather} ${temperature}℃`;  
            document.getElementById('weatherData').innerText = formattedInfo;  
        })
        .catch(error => {  
        });  
}
function yiyan() {  
    return fetch('https://v1.hitokoto.cn/?max_length=25&min_length=20&c=j')  
        .then(response => response.json())  
        .then(data => {  
            const formattedInfo = data.hitokoto;
            document.getElementById('yiyan').innerText = formattedInfo;  
        })
        .catch(error => {  
        });  
}
</script>

<template>
  <UBasePage>
    <div
      class="bg-primary text-white w-full top-0 z-200 fixed font-bold"
      :style="{ height: `${customBarHeight}px` }"
    >
      <div :style="{ 'padding-top': `${statusBarHeight}px`, 'height': `${customBarHeight - statusBarHeight}px` }">
        <div class="h-full text-center px-6 relative" style="background: linear-gradient(rgb(108, 176, 247), rgb(108, 176, 247) 8%, rgb(80, 192, 255));
		box-shadow: 2px 4px 8px rgba(0,150,255,.3); font-size: 24px; display: flex;flex-direction: row; align-items: center;">
<a style="text-decoration: none;float: left;font-size: 14px;display: inline-block;padding: 4px 8px;font-weight: normal;border-radius: 100px;line-height: initial;color: rgb(255, 255, 255);margin-right: auto;    background: linear-gradient(rgb(255 255 255 / 10%),rgb(240 242 244 / 10%));
    box-shadow: 0 2px 2px rgb(255 255 255 / 40%), 0 0 0 1px #fff inset;" @click="showImage">更多帮助</a>
<a ref="selectedClassTitle" style="font-size: 18px;margin-right: auto;overflow: hidden;width: 40vmin;text-overflow: ellipsis;white-space: nowrap;">请选择班级</a>
          <div class="flex h-full mx-auto justify-center items-center inline-block text-lg" style="font-weight: normal; font-size: 24px; display: contents; text-align: right; margin-right: auto;">
			  <a style="float: right;font-size: 14px;text-decoration: none;display: inline-block;padding: 4px 8px;border-radius: 100px;color: rgb(255, 255, 255);line-height: initial;    background: linear-gradient(rgb(255 255 255 / 10%),rgb(240 242 244 / 10%));
    box-shadow: 0 2px 2px rgb(255 255 255 / 40%), 0 0 0 1px #fff inset;" @click="goToIndexPage">进入课表</a>
          </div>
        </div>
		<p style="margin-bottom: -20px;text-align: center;color:#4fbfff" id="weatherData">XXXX年X月X日 星期X XX XX°C</p>
      <div class="round-button" style="
        background:linear-gradient(45deg , rgb(108, 176, 247), rgb(108, 176, 247) 8%, rgb(80, 192, 255));
		box-shadow: 2px 4px 8px rgba(0,150,255,.3);
        border-radius: 0.5rem;margin-top: 20px;width:calc(80vw - 40px)
      ">
      <p style="color: #fff;font-size: 18px;">(20XX-20XX) 第X学期</p>
  <div class="class-list">
      <input v-model="searchText" placeholder="请输入班级名称" @keyup.enter="handleEnter" style="
    padding: 4px;
    width: 100%;
    box-sizing: border-box;
    font-size: 18px;
    background-color: rgba(255, 255, 255, 0.3);
    color: #fff;
    border:0;
    height: 40px;
	border-radius: 50px;
	box-shadow: rgba(255, 255, 255, 0.4) 0px 2px 2px, rgb(255, 255, 255) 0px 0px 0px 1px inset;
    line-height: 40px;margin-bottom:4px;
    margin-top: 1%; 
"/>
<div style="max-height:calc(100vh - 190px);overflow-y: scroll;">
      <ul style="line-height:1.5">
        <li v-for="cls in filteredClasses" :key="cls.value" :class="{ selected: cls === selectedClass }" @click="handleClick(cls)">
          {{ cls.name }}
        </li>
      </ul>
      <p v-if="filteredClasses.length === 0">没有匹配结果</p>
    </div>
    </div>
</div> 

<div class="flex h-full mx-auto justify-center items-center inline-block text-lg" style="font-size:12px;color:rgb(108, 176, 247)"><p id="yiyan"></p></div>
</div></div>
  </UBasePage>
</template>
<style scoped>
.round-button{background-color:#6cb0f7;border:none;border-radius:.5rem;color:#fff;cursor:pointer;font-size:24px;font-weight:700;padding:4px 20px;text-align:center;text-decoration:none;text-transform:uppercase;box-shadow:0 8px 16px rgba(0,0,0,.2);transition:transform .3s ease-out,box-shadow .3s ease-out;display:block;margin:auto}.round-button:hover{background-color:#4091e6;box-shadow:0 12px 24px rgba(0,0,0,.4)}.animated{animation:pulse .5s ease-in-out infinite alternate}@keyframes pulse{0%{transform:scale(1)}to{transform:scale(1.1)}}li,ul{margin:0;padding:0}ul{font-size:18px;font-weight:400;list-style:none}#fruit-list-wrapper{max-height:80%;overflow-y:auto}.selected{border-radius: 50px;box-shadow: rgba(255, 255, 255, 0.4) 0px 2px 2px, rgb(255, 255, 255) 0px 0px 0px 1px inset;}.uni-input-placeholder{color:#ffffffd6}</style>