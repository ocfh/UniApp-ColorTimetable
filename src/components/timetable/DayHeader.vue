<script setup lang="ts">
import { weekTitle } from '~/stores/course'
const hash = window.location.hash;  
const match = hash.match(/date=(\d+)/);  
const paramDate = match ? match[1] : null;  
let formattedDate;  
if (paramDate && typeof paramDate === 'string' && paramDate.length === 8) {  
  formattedDate = `${paramDate.slice(0,4)}-${paramDate.slice(4,6)}-${paramDate.slice(6,8)}`; 
	 const startDate = new Date('2023-08-28');
	 const endDate = new Date(formattedDate);
	 const today = new Date();  
	 today.setHours(0, 0, 0, 0); // 将今天的日期设置为午夜开始  
	 const isSameDate = endDate.getFullYear() === today.getFullYear() &&  
	                    endDate.getMonth() === today.getMonth() &&  
	                    endDate.getDate() === today.getDate();  
	   
	 if (isSameDate) {  
	     setTimeout(find, 1);  
	 }
	 const diffInMilliseconds = endDate - startDate;  
	 const diffInDays = Math.ceil(diffInMilliseconds / (1000 * 60 * 60 * 24));
	 const weeks = Math.floor(diffInDays / 7);
} else {
  const now = new Date();  
  formattedDate = `${now.getFullYear()}-${('0' + (now.getMonth() + 1)).slice(-2)}-${('0' + now.getDate()).slice(-2)}`;
  setTimeout(find, 1);
}  
const today = new Date(formattedDate);  
const month = (today.getMonth() + 1).toString().replace(/^0+/, '');
const day = today.getDate().toString().replace(/^0+/, '');
let dayOfWeek = today.getDay();  
const weekdayMap = {  
  0: '日', // 星期日  
  1: '一', // 星期一  
  2: '二', // 星期二  
  3: '三', // 星期三  
  4: '四', // 星期四  
  5: '五', // 星期五  
  6: '六'  // 星期六  
};  
const {
  isStart, currentMonth, originalWeekIndex, currentWeekIndex,
  originalWeekWeekIndex, currentWeekDayArray,
} = storeToRefs(useCourseStore())
const isCurrentWeek = (weekIndex: number) => {
  if (!isStart.value)
    return false
  return originalWeekIndex.value === currentWeekIndex.value && originalWeekWeekIndex.value === weekIndex
}
const timeSlots = [  
    { start: '08:15', end: '09:00' },  
    { start: '09:10', end: '09:55' },  
    { start: '10:10', end: '10:55' },  
    { start: '11:05', end: '11:50' },  
    { start: '14:30', end: '15:15' },  
    { start: '15:25', end: '16:10' },  
    { start: '16:20', end: '17:05' },  
    { start: '17:15', end: '18:00' },
	{ start: '18:10', end: '18:55' },
	{ start: '19:30', end: '20:15' },
	{ start: '20:25', end: '21:10' },
	{ start: '21:20', end: '22:05' }
];  
let currentIndex = 0;  
function find() {   
	let countdownList = document.getElementById('countdownList');
	if (countdownList === null) {  
	    setTimeout(find, 1);
	}  else {
		setTimeout(showCountdown, 1);
	}
}
function showCountdown() {    
    const timeSlot = timeSlots[currentIndex];    
    const now = new Date();  
    const endTime = new Date(now.getFullYear() + '-' +('0' + (now.getMonth() + 1)).slice(-2) + '-' +('0' + now.getDate()).slice(-2) + ' ' +`${timeSlot.end}` ).getTime();
    const distance = endTime - now;  
    const minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));  
    const seconds = Math.floor((distance % (1000 * 60)) / 1000);  
    const hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));  
	let countdownList = document.getElementById('countdownList');
    if (distance < 0) {    
        currentIndex++;  
        if (currentIndex < timeSlots.length) {    
            setTimeout(showCountdown, 1);    
        }  else {
			currentIndex = 0;
			countdownList.innerHTML = `休息时间吃饭睡觉打豆豆`;  
			setTimeout(showCountdown, 1000);    
		}
    } else {
	if (hours > 0) { 
         countdownList.innerHTML = `休息时间吃饭睡觉打豆豆`;  
		 setTimeout(showCountdown, 1000);  
	} else {
			if (minutes < 45) { 
			if(minutes === 0)
			{
			countdownList.innerHTML = `第${currentIndex+1}节课下课还剩${seconds}秒`; 
			} else {
			countdownList.innerHTML = `第${currentIndex+1}节课下课还剩${minutes}分${seconds}秒`; 
			} 
            setTimeout(showCountdown, 1000);    
        } else {    
			const startTime = new Date(`${now.getFullYear()}-${now.getMonth()+1}-${now.getDate()}T${timeSlot.start}`).getTime();  
			const distance = startTime - now; 
			const minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));  
			const seconds = Math.floor((distance % (1000 * 60)) / 1000); 
			if(minutes === 0)
			{
			countdownList.innerHTML = `第${currentIndex+1}节课上课还剩${seconds}秒`; 
			} else {
			countdownList.innerHTML = `第${currentIndex+1}节课上课还剩${minutes}分${seconds}秒`;
			}
            setTimeout(showCountdown, 1000);    
        } 
	}
    }
}
function navigateToToday() {
const currentUrl = window.location.href;   
const pathRegex = /#\/pages\/(.*)\??/;  
const pathMatch = currentUrl.match(pathRegex);  
let targetPath = '/';
if (pathMatch && pathMatch[1]) {  
  targetPath = `/pages/${pathMatch[1]}`;
}
const pathWithoutQuery = targetPath.split('?')[0];  
console.log(pathWithoutQuery);
uni.navigateTo({  
  url: pathWithoutQuery,  
});
}
</script>
<template>
  <div class="h-10 shadow-sm px-1 top-0" grid="~ cols-[0.7fr_repeat(7,1fr)] gap-1">
  <template v-for="(item, index) in currentWeekDayArray" :key="index">
    <div 
      class="text-xs transition-all duration-300 !bg-op40"
	  style="width:100vw;margin-left: -0.25rem;    border-bottom-color: #50bfff!important;
    box-shadow: 0 3px 4px rgba(32,160,255,.5);
"
      flex="~ col" justify-evenly items-center
      b="y-transparent x-none t-4 b-4"
      :class="'bg-light-blue-300 !b-b-light-blue-500'"
	  @click="navigateToToday"
    >
      <p class="font-medium">{{ `${month}月${day}日 星期${weekdayMap[dayOfWeek]}` }}</p>
      <p id="countdownList">点击返回今日课表日程</p>
    </div>
  </template>
  </div>
</template>
