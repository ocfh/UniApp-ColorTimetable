<script setup lang="ts">
import TimetableAction from './TimetableAction.vue'
import TimetableHeader from './DayHeader.vue'
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
const hash = window.location.hash;  
const match = hash.match(/date=(\d+)/);  
const paramDate = match ? match[1] : null;  
let formattedDate;  
if (paramDate && typeof paramDate === 'string' && paramDate.length === 8) {  
formattedDate = `${paramDate.slice(0,4)}-${paramDate.slice(4,6)}-${paramDate.slice(6,8)}`;  
const startDate = new Date('2024-02-26');  
const endDate = new Date(formattedDate);  
const diffInMilliseconds = endDate - startDate;  
const diffInDays = Math.ceil(diffInMilliseconds / (1000 * 60 * 60 * 24));
const weeks = Math.floor(diffInDays / 7);
setCurrentWeekIndex(weeks);
} else {  
  const now = new Date();  
  formattedDate = `${now.getFullYear()}-${('0' + (now.getMonth() + 1)).slice(-2)}-${('0' + now.getDate()).slice(-2)}`;  
}  
const dateObj = new Date(formattedDate);  
const today = dateObj.getDay();  
const deleteWeekCourse = computed(() => {
const weekCourse = Array.from(weekCourseList.value.filter(course => {
  if (course.week === today || (today === 0 && course.week === 7)) {
    return true;
  }
  return false;
}));
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
function getCoursePosition(item: CourseModel) {
  return {
    'grid-row': `${item.start} / ${item.start + item.duration}`,
    'grid-column': `2 / 9`,
  }
}
let style = document.createElement('style');  
style.innerHTML = `  
.uno-pltnqj{  
	background: linear-gradient(90deg,#50c0ff,#20a0ff 8%,#50c0ff)!important;
	box-shadow: 2px 4px 8px rgba(0,150,255,.3);
    position: fixed;  
    z-index: 30;  
    border-top-left-radius: 9999px;  
    border-bottom-left-radius: 9999px;  
    padding-top: 0.5rem;  
    padding-bottom: 0.5rem;  
    padding-left: 1rem;  
    padding-right: 0.5rem;  
    font-size: 0.875rem;  
    line-height: 1.25rem;  
    color: rgba(255,255,255,var(--un-text-opacity));  
    transition-property: all;  
    transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);  
    transition-duration: 150ms;  
    transition-duration: 300ms;  
}`;  
document.head.appendChild(style);
</script>
<template>
  <div class="overflow-y-scroll relative bg-base" :style="{ height: `calc(100vh - ${customBarHeight}px)` }">
    <div class="w-full top-0 z-10 fixed bg-base" :style="{ 'padding-top': `${customBarHeight}px` }">
      <TimetableHeader />
    </div>
    <div
      class="min-h-max pb-safe p-1 transition-all z-20 duration-300 bg-base"
      grid="~ flow-col rows-12 cols-[0.7fr_repeat(7,1fr)] gap-1" :class="showCourseAction ? 'pt-31' : 'pt-11'"
    >
      <template v-for="(courseTime, courseIndex) in courseTimeList" :key="courseIndex">
        <div class="text-sm  min-h-14" flex="~ col" justify-evenly items-center>
          <div class="px-0.5 text-8px" style="text-shadow: 2px 2px 4px rgba(128, 128, 128, 0.35);">
            {{ courseTime.s }}<hr style="border-style: dashed;border-width: 1px;">{{ courseTime.e }}
          </div>
        </div>
      </template>
      <template v-for="(courseItem, _courseIndex) of deleteWeekCourse" :key="_courseIndex">
        <div
          class="rounded-lg p-0.5 relative dark:bg-op40" b="white 2 !op-50" style="box-shadow: rgba(255, 255, 255, 0.4) 0px 0px 0px 1px inset, rgb(128 128 128 / 15%) 0px 13px 15px;"
          :style="[getCoursePosition(courseItem), `background:${hasConflictCourseByMap(courseItem)[0].color}`]"
          @click="emit('courseItemClick', courseItem)"
        >
          <div class="h-full w-full" text="center white xs" flex="~ col" justify-around items-center style="position:relative;font-size:.9rem;">
              <div><p style="font-weight: bold;font-size: 1rem">{{ hasConflictCourseByMap(courseItem)[0].title }}</p>
			  <p class="flex gap-1 justify-start items-center" style="justify-content: center;"><div class="i-carbon-light" />{{ hasConflictCourseByMap(courseItem)[0].teacher }}</p></div>
			  <p class="flex gap-1 justify-start items-center" style="position: absolute;top: 0.2rem;left: 0.2rem;"><div class="i-carbon-alarm" />{{ hasConflictCourseByMap(courseItem)[0].start }}-{{ hasConflictCourseByMap(courseItem)[0].duration+hasConflictCourseByMap(courseItem)[0].start-1 }}节</p>
			  <p class="flex gap-1 justify-start items-center" style="position: absolute;top: 0.2rem;right: 0.2rem;"><div class="i-carbon-up-to-top" />{{ hasConflictCourseByMap(courseItem)[0].weeks[0] }}-{{ hasConflictCourseByMap(courseItem)[0].weeks[hasConflictCourseByMap(courseItem)[0].weeks.length - 1] }}周</p>
			  <p class="flex gap-1 justify-start items-center" style="bottom:0.2rem;position: absolute;"><div class="i-carbon-location" />{{ hasConflictCourseByMap(courseItem)[0].location }}</p>
			 <!-- <p style="font-weight: bold;font-size:1.1rem;">{{ hasConflictCourseByMap(courseItem)[0].title }}</p>
			  <p style=""><div class="i-carbon-location" />{{ hasConflictCourseByMap(courseItem)[0].location }}</p>
			  <p style=""><div class="i-carbon-light" />{{ hasConflictCourseByMap(courseItem)[0].teacher }}</p>
			  <p style=""><div class="i-carbon-alarm" />{{ hasConflictCourseByMap(courseItem)[0].start }}-{{ hasConflictCourseByMap(courseItem)[0].duration+hasConflictCourseByMap(courseItem)[0].start-1 }}节</p> -->
            <div
              v-if="hasConflictCourseByMap(courseItem).length > 1"
              class="rounded h-1 top-1 left-1 right-1 absolute bg-white/80" style="bottom:0;top:unset;height: 0.15rem;"
            />
          </div>
        </div>
      </template>
    </div>
  </div>
</template>
