<script setup lang="ts">
import type { CourseModel } from '~/stores/course'
import CourseActionSheet from '~/components/timetable/CourseActionSheet.vue'
import TimetableContent from '~/components/timetable/TimetableContent.vue'
import courses from '~/static/courses'

const { customBarHeight, statusBarHeight } = useAppStore()
const { setPageConfig } = usePageStore()
const { currentWeekIndex, isStart } = storeToRefs(useCourseStore())
const { setCourseList, setStartDay } = useCourseStore()

onShow(() => {
  setPageConfig({ showNavBar: false })
})

setCourseList(courses as CourseModel[])

const showCourseAction = ref(false)

// set the start date
const someDate = new Date(2024,0,1)
someDate.setDate(someDate.getDate())
setStartDay(someDate)

// handle course item click
const showActionSheet = ref(false)
const clickedCourseItem = ref<CourseModel>()
function handleShowActionSheet(courseItem: CourseModel) {
  showActionSheet.value = true
  clickedCourseItem.value = courseItem
}

function handleCloseActionSheet() {
  showActionSheet.value = false
}
function handleCreateCourse() {
  const currentDate = new Date();const date = `${currentDate.getFullYear()}${String(currentDate.getMonth() + 1).padStart(2, '0')}${String(currentDate.getDate()).padStart(2, '0')}`;uni.navigateTo({
    url: `/pages/day/courses?${date ? `date=${date}` : 'no'}`,
  })
}
</script>

<template>
  <UBasePage>
    <div
      class="bg-primary text-white w-full top-0 z-200 fixed font-bold"
      :style="{ height: `${customBarHeight}px` }"
    >
      <div :style="{ 'padding-top': `${statusBarHeight}px`, 'height': `${customBarHeight - statusBarHeight}px` }">
        <div class="h-full text-center px-6 relative" style="background: linear-gradient(rgb(108, 176, 247), rgb(108, 176, 247) 8%, rgb(80, 192, 255));box-shadow: rgba(0, 150, 255, 0.3) 2px 4px 8px;font-size: 24px;display: flex;flex-direction: row;align-items: center;">
		<a style="text-decoration: none; float: left; font-size: 14px; display: inline-block; padding: 4px 8px; font-weight: normal; border-radius: 100px; line-height: initial; color: rgb(255, 255, 255); margin-right: auto; background: linear-gradient(rgba(255, 255, 255, 0.1), rgba(240, 242, 244, 0.1)); box-shadow: rgba(255, 255, 255, 0.4) 0px 2px 2px, rgb(255, 255, 255) 0px 0px 0px 1px inset;" @click="handleCreateCourse">返回日程</a>
		<a style="font-size: 18px;display: inline-flex;margin-right: auto;">{{ `第${currentWeekIndex + 1}周${!isStart ? ' (未开学)' : ''}` }}</a>
          <div 
style="font-weight: normal;font-size: 24px;display: contents;text-align: right;margin-right: auto;"
 class="flex h-full mx-auto justify-center items-center inline-block text-lg"
            @click="showCourseAction = !showCourseAction"
>
            <a style="float: right; font-size: 14px; text-decoration: none; display: inline-block; padding: 4px 8px; border-radius: 100px; color: rgb(255, 255, 255); line-height: initial; background: linear-gradient(rgba(255, 255, 255, 0.1), rgba(240, 242, 244, 0.1)); box-shadow: rgba(255, 255, 255, 0.4) 0px 2px 2px, rgb(255, 255, 255) 0px 0px 0px 1px inset;">周数选择</a>
          </div>
        </div>
      </div>
    </div>
    <!-- timetable main content -->
    <TimetableContent :show-course-action="showCourseAction" @course-item-click="handleShowActionSheet" />
    <!-- course card -->
    <CourseActionSheet
      :show-action-sheet="showActionSheet" :course-item="clickedCourseItem"
      @cancel="handleCloseActionSheet"
    />
  </UBasePage>
</template>
