<script setup lang="ts">
const props = defineProps<{ showCourseAction: boolean }>()
const { parsedCourseList, originalWeekIndex, currentWeekIndex } = storeToRefs(useCourseStore())
const { setCurrentWeekIndex } = useCourseStore()
const scrollTo = ref('week0')
watch(
  () => +props.showCourseAction + currentWeekIndex.value,
  () => {
    if (props.showCourseAction)
      scrollTo.value = `week${currentWeekIndex.value - 1}`
  })
</script>
<template>
  <scroll-view
    class="transition-height duration-300 overflow-scroll whitespace-nowrap"
	style="overflow: auto hidden; filter: drop-shadow(rgba(147, 197, 253, 0.5) 0px 0px 4px); background-color: rgba(0, 0, 0, 0.4); color: rgb(211, 211, 211);"
    :class="showCourseAction ? 'h-20' : 'h-0'" scroll-x scroll-with-animation :scroll-into-view="scrollTo"
  >
    <template v-for="(weeksTimetable, weeksIndex) of parsedCourseList" :key="weeksIndex">
      <div
        :id="`week${weeksIndex + 1}`" class="py-1 px-2 inline-block"
        @click="setCurrentWeekIndex(weeksIndex)"
      >
        <div style="margin-top:1rem;border-radius: 100px;"
          class="rounded-lg py-1 px-2"
          :class="originalWeekIndex === weeksIndex ? 'bg-gray-400/50 dark:!bg-op60' : currentWeekIndex === weeksIndex ? 'bg-gray-300 bg-op80 dark:!bg-op20' : ''"
		  :style="originalWeekIndex === weeksIndex ? 'color: rgb(80, 191, 254)!important;box-shadow: rgb(81, 191, 254) 0px 2px 2px, rgb(81, 192, 255) 0px 0px 0px 1px inset!important;' : currentWeekIndex === weeksIndex ? 'color: rgb(80, 191, 254)!important;box-shadow: rgb(81, 191, 254) 0px 2px 2px, rgb(81, 192, 255) 0px 0px 0px 1px inset!important;' : 'box-shadow: rgba(255, 255, 255, 0.4) 0px 2px 2px, rgb(255, 255, 255) 0px 0px 0px 1px inset;'"
        >
          <div class="text-xs text-center mb-1">
            {{ `第 ${weeksIndex + 1} 周` }}
          </div>
          
        </div>
      </div>
    </template>
  </scroll-view>
</template>
