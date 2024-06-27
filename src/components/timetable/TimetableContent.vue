<script setup lang="ts">
import TimetableAction from './TimetableAction.vue'
import TimetableHeader from './TimetableHeader.vue'
import type { CourseModel } from '~/stores/course'
import { courseTimeList } from '~/stores/course'

withDefaults(
  defineProps<{ showCourseAction: boolean }>(), {
    showCourseAction: false,
  },
)

const emit = defineEmits(['courseItemClick'])

const { customBarHeight } = storeToRefs(useAppStore())
const { weekNum, weekCourseList, currentWeekIndex, originalWeekIndex } = storeToRefs(useCourseStore())
const { hasConflictCourseByMap, setCurrentWeekIndex } = useCourseStore()

const deleteWeekCourse = computed(() => {
  const weekCourse = Array.from(weekCourseList.value)
  if (weekCourse.length <= 1)
    return weekCourse
  for (let i = 1; i < weekCourse.length; i++) {
    const { start, week } = weekCourse[i]
    const { start: prevStart, week: prevWeek } = weekCourse[i - 1]
    if (start === prevStart && week === prevWeek) {
      weekCourse.splice(i, 1)
      i--
    }
  }
  return weekCourse
})

const startX = ref(0)
const startY = ref(0)
const towardsX = ref(0)
const towardsY = ref(0)
const arrowX = ref(0)
const arrowrX = ref(0)
const arrowrrX = ref(0)
const screenWidth = window.innerWidth; 
const hash = window.location.hash;  
const match = hash.match(/week=(\d+)/);  
const paramWeek = match ? parseInt(match[1]) : null;  

if (paramWeek !== null) {  
  setCurrentWeekIndex(paramWeek - 1);  
}

function resetTouchStatus() {
  startX.value = 0
  startY.value = 0
  towardsX.value = 0
  towardsY.value = 0
}

function handleTouchStart(e: TouchEvent) {
  startX.value = e.touches[0].clientX
  startY.value = e.touches[0].clientY
}

function handleTouchMove(e: TouchEvent) {
  towardsX.value = e.touches[0].clientX - startX.value
  towardsY.value = e.touches[0].clientY - startY.value
  arrowX.value= (towardsX.value)/100
  arrowrrX.value= (1-towardsX.value)/100
  arrowrX.value= (screenWidth - towardsX.value)/100
  if (towardsX.value >= 0) {  
    var elements = document.getElementsByClassName('list');  
    for(var i = 0; i < elements.length; i++) {  
        elements[i].style.opacity = `${arrowX.value}`;   
		elements[i].style.transform = `translateX(${arrowX.value}px) scale(0.5) translateY(80vh)`;
		}
    }
	
	
  if (towardsX.value <= 0) {
    var elements = document.getElementsByClassName('list');  
    for(var i = 0; i < elements.length; i++) {  
        elements[i].style.opacity = `${arrowrrX.value}`;   
		elements[i].style.transform = `translateX(calc(100vw - 100px - ${arrowrX.value}px )) scale(0.5) translateY(40vh)`;
    }  
    }

	if(towardsX.value>150){
		document.getElementById('list').style.transform = `translateX(100px)` 
	}
	if(towardsX.value<-150){
		document.getElementById('list').style.transform = `translateX(-100px)` 
	}
	var elements = document.getElementsByClassName('arrow');
	for(var i = 0; i < elements.length; i++) {  
	    elements[i].style.opacity = `0`;   
		elements[i].style.transform = `translateX(-100px) scale(0.5) translateY(80vh)`;
	}  
	var elements = document.getElementsByClassName('arrowr');
	for(var i = 0; i < elements.length; i++) {  
	    elements[i].style.opacity = `0`;   
		elements[i].style.transform = `translateX(100vw) scale(0.5) translateY(80vh)`;
	}
	
}


function translateXToZero() {  
	document.getElementById('list').style.transform = `translateX(0px)` 
    var elements = document.getElementsByClassName('arrow');
    for(var i = 0; i < elements.length; i++) {  
        elements[i].style.opacity = `0.8`;   
    	elements[i].style.transform = `scale(0.5) translateY(80vh)`;
    }  
	var elements = document.getElementsByClassName('arrowr');
	for(var i = 0; i < elements.length; i++) {  
	    elements[i].style.opacity = `0.8`;   
		elements[i].style.transform = `translateX(calc(100vw - 100px)) scale(0.5) translateY(40vh)`;
	}
	let currentWeekIndexTemp = currentWeekIndex.value
	if(currentWeekIndexTemp == 0){
		 var elements = document.getElementsByClassName('arrow');
		 for(var i = 0; i < elements.length; i++) {  
		 elements[i].style.opacity = `0`;
		 elements[i].style.transform = `translateX(-100px) scale(0.5) translateY(80vh)`;
		 }
		 }else{
			 var elements = document.getElementsByClassName('arrow');
			 for(var i = 0; i < elements.length; i++) {  
			 elements[i].style.opacity = `0.8`;
			 elements[i].style.transform = `scale(0.5) translateY(80vh)`;
			 }
		 }
		 if(currentWeekIndexTemp == 19){
		 	 var elements = document.getElementsByClassName('arrowr');
		 	 for(var i = 0; i < elements.length; i++) {  
		 	 elements[i].style.opacity = `0`;
		 	 elements[i].style.transform = `translateX(100vw) scale(0.5) translateY(80vh)`;
		 	 }
		 	 } else {
				 var elements = document.getElementsByClassName('arrowr');
				 for(var i = 0; i < elements.length; i++) {  
				 elements[i].style.opacity = `0.8`;
				 elements[i].style.transform = `translateX(calc(100vw - 100px)) translateY(40vh) scale(0.5)`;
				 }
			 }
}  

function handleTouchEnd() {
  let currentWeekIndexTemp = currentWeekIndex.value
  
  if (towardsX.value > 150) {
    if (currentWeekIndexTemp === 0)
      return
    currentWeekIndexTemp--
  }
  else if (towardsX.value < -150) {
    if (currentWeekIndexTemp === weekNum.value - 1)
      return
    currentWeekIndexTemp++
  }
  setCurrentWeekIndex(currentWeekIndexTemp)
  resetTouchStatus()
  document.getElementById('list').style.transform = `translateX(0px)` 
  setTimeout(translateXToZero, 5438);
}

function getCoursePosition(item: CourseModel) {
  return {
    'grid-row': `${item.start} / ${item.start + item.duration}`,
    'grid-column': `${item.week + 1} / ${item.week + 1 + 1}`,
  }
}

function nexttoweek() {
	let currentWeekIndexTemp = currentWeekIndex.value
 currentWeekIndexTemp++
 setCurrentWeekIndex(currentWeekIndexTemp)
 resetTouchStatus()
 translateXToZero()
}

function lastweek() {
	let currentWeekIndexTemp = currentWeekIndex.value
  currentWeekIndexTemp--
  setCurrentWeekIndex(currentWeekIndexTemp)
  resetTouchStatus()
  translateXToZero()
}
setTimeout(translateXToZero, 5438);
</script>
<style>.text-sm1{transition: all 0.3s ease}.arrow{
	    position: fixed;
	    opacity: 0;
		transform: translateX(-100px) translateY(80vh) scale(0.5); 
		z-index: 9;filter: drop-shadow(rgba(147,197,253,0.5) 0px 0px 4px);
		object-fit: contain;transition: all 0.1s ease
}
.arrowr{
	    position: fixed;
		transform: scale(0.5); 
	    opacity: 0;
		z-index: 9;filter: drop-shadow(rgba(147,197,253,0.5) 0px 0px 4px);
		object-fit: contain;
		transform: translateX(100vw) scale(0.5) translateY(80vh);transition: all 0.1s ease
}
</style>
<template>
  <div class="overflow-y-scroll relative bg-base" :style="{ height: `calc(100vh - ${customBarHeight}px)` }">
    <div class="w-full top-0 z-10 fixed bg-base" :style="{ 'padding-top': `${customBarHeight}px` }">
      <TimetableAction :show-course-action="showCourseAction" />
      <TimetableHeader />
    </div><img class="arrow" src="https://pic.superbed.cc/item/667da9bdfcada11d37b732e8.png" @click="lastweek"> 
	<img class="arrowr" src="https://pic.superbed.cc/item/667da9cafcada11d37b7332d.png" @click="nexttoweek"> 
    <div
      class="min-h-max pb-safe p-1 transition-all z-20 duration-300 bg-base" style="padding-right:0!important"
      grid="~ flow-col rows-12 cols-[0.7fr_repeat(7,1fr)] gap-1" :class="showCourseAction ? 'pt-31' : 'pt-11'"
      @touchstart="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd" id="list"
    >
      <template v-for="(courseTime, courseIndex) in courseTimeList" :key="courseIndex">
        <div class="text-sm text-sm1 min-h-20" flex="~ col" justify-evenly items-center>
          <div class="font-medium" style="text-shadow: 2px 2px 4px rgba(128, 128, 128, 0.35);">
            {{ courseIndex + 1 }}
          </div>
          <div class="px-0.5 text-8px">
            {{ courseTime.s }}<hr style="border-style: dashed;border-width: 1px;">{{ courseTime.e }}
          </div>
        </div>
      </template>
      <template v-for="(courseItem, _courseIndex) of deleteWeekCourse" :key="_courseIndex">
        <div
          class="text-sm1 rounded-lg p-0.5 relative dark:bg-op40" b="white 2 !op-50" style="box-shadow:rgba(255, 255, 255, 0.4) 0px 0px 0px 1px inset, rgb(0 0 0 / 10%) 0px 13px 15px;"
          :style="[getCoursePosition(courseItem), `background:${hasConflictCourseByMap(courseItem)[0].color}`]"
          @click="emit('courseItemClick', courseItem)"
        >
          <div class="h-full w-full" text="center white xs" flex="~ col" justify-around items-center>
            <div class="font-medium break-all">
              <p style="font-weight: bold;display: inline;">{{ hasConflictCourseByMap(courseItem)[0].title }}</p>@{{ hasConflictCourseByMap(courseItem)[0].location }}
            </div>
           <div
              v-if="hasConflictCourseByMap(courseItem).length > 1"
              class="rounded h-1 top-1 left-1 right-1 absolute bg-white/80" style="bottom:0.15rem;top:unset;height: 0.15rem;"
            />
          </div> 
        </div>
      </template>
    </div>
    <div style="top: 25%;filter: drop-shadow(rgba(147, 197, 253, .5) 0px 0px 4px);background-color: rgba(0, 0, 0, .4);color: #d3d3d3;"
      class="bg-primary fixed z-30 rounded-l-full transition-all duration-300 fhbz"
      text="white sm"
      p="l-4 y-2 r-2"
      :class="originalWeekIndex !== currentWeekIndex ? 'right-0' : '-right-full'"
      @click="setCurrentWeekIndex(originalWeekIndex)"
    >
      返回本周
    </div>
  </div>
</template>
