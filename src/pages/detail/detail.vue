<script setup lang="ts">
import { onLoad } from '@dcloudio/uni-app'
import { cloneDeep } from 'lodash-es'
import type { CourseModel } from '~/stores/course'

const appStore = useAppStore()
const { setPageConfig } = usePageStore()

const defaultCourse: CourseModel = {
  title: '',
  location: 'Địa điểm',
  week: 1,
  weeks: [1, 3, 5],
  start: 1,
  duration: 2,
}

const courseStore = useCourseStore()
const courseList = ref<CourseModel[]>([])

const originalCourseTitle = ref('')
const courseTitle = ref('')
const isUpdate = ref(false)

onLoad((option: any) => {
  isUpdate.value = !!option?.course
  if (isUpdate.value) {
    const courseListTemp = courseStore.courseList.filter(item => item.title === option?.course)
    for (const courseItem of courseListTemp)
      courseList.value.push(cloneDeep(courseItem))
    originalCourseTitle.value = option?.course
    courseTitle.value = option?.course
  }
  else {
    courseList.value = [defaultCourse]
  }
})

onShow(() => {
  setPageConfig({ pageTitle: isUpdate.value ? 'Chỉnh sửa khóa học' : 'Thêm khóa học', showBackAction: true })
})

function handleDeleteCourseItem(courseIndex: number) {
  uni.showModal({
    title: 'cảnh báo',
    content: 'Bạn có chắc chắn muốn xóa khóa học trong khoảng thời gian này không?',
    success: (res) => {
      if (res.confirm)
        courseList.value.splice(courseIndex, 1)
    },
  })
}

function handleAddNewTime() {
  courseList.value.push(cloneDeep(courseList.value[courseList.value.length - 1]))
}

function handleSaveCourse() {
  if (!courseTitle.value) {
    uni.showToast({
      title: 'Vui lòng nhập tên khóa học',
      icon: 'none',
    })
    return
  }
  for (const courseItem of courseList.value)
    Object.assign(courseItem, { title: courseTitle.value })

  if (isUpdate.value)
    courseStore.deleteCourseItemByTitle(originalCourseTitle.value)

  courseStore.setCourseList(courseStore.courseList.concat(courseList.value))
  uni.showModal({
    title: 'gợi ý',
    content: 'Đã lưu thành công',
    showCancel: false,
  })
}

const showWeekActionSheet = ref(false)
const clickedIndex = ref(-1)
const clickedWeeks = ref<number[]>([])

function handleShowWeekActionSheet(clickIndex: number) {
  showWeekActionSheet.value = true
  clickedIndex.value = clickIndex
  clickedWeeks.value = courseList.value[clickIndex].weeks
}

/**
 * transform weeks to string eg: [1, 2, 3, 5, 6, 8] to '1-3,5-6,8'
 * @param weeks week list
 */
function transformArray2String(weeks: number[]): string {
  let weeksString = ''
  for (let i = 0; i < weeks.length; i++) {
    if (i === 0) {
      weeksString += weeks[i]
    }
    else {
      if (weeks[i] !== weeks[i - 1] + 1) {
        const last = weeksString.split(',')[weeksString.split(',').length - 1]
        if (Number.parseInt(last) !== weeks[i - 1])
          weeksString += `-${weeks[i - 1]}`
        weeksString += `,${weeks[i]}`
      }
      else {
        if (i === weeks.length - 1)
          weeksString += `-${weeks[i]}`
      }
    }
  }
  return weeksString
}

function handleClickWeek(week: number) {
  if (!clickedWeeks.value.includes(week)) {
    clickedWeeks.value.push(week)
    clickedWeeks.value.sort((a, b) => a - b)
  }
  else {
    const index = clickedWeeks.value.indexOf(week)
    clickedWeeks.value.splice(index, 1)
  }
}

function handleConfirmWeekActionSheet() {
  showWeekActionSheet.value = false
  courseList.value[clickedIndex.value].weeks = clickedWeeks.value
}

const showTimeActionSheet = ref(false)
const timeValue = ref([1, 1, 1])

function handleShowTimeActionSheet(clickIndex: number) {
  showTimeActionSheet.value = true
  clickedIndex.value = clickIndex
  const { week, start, duration } = courseList.value[clickIndex]
  timeValue.value = [week, start, duration]
}

const timeList = [
  ['Ngày trong tuần', 'Thứ Hai', 'Thứ Ba', 'Thứ Tư', 'Thứ Năm', 'Thứ Sáu', 'Thứ Bảy', 'Chủ Nhật'],
  ['Bắt đầu', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
  ['Thời gian', '1 Buổi', '2 Buổi', '3 Buổi', '4 Buổi', '5 Buổi', '6 Buổi', '7 Buổi', '8 Buổi'],
]

function handleTimeChange(e: any) {
  const value = e.detail.value
  courseList.value[clickedIndex.value].week = value[0] ? value[0] : 1
  courseList.value[clickedIndex.value].start = value[1] ? value[1] : 1
  courseList.value[clickedIndex.value].duration = value[2] ? value[2] : 1
}

function handleConfirmTimeActionSheet() {
  showTimeActionSheet.value = false
}
</script>

<template>
  <UBasePage>
    <div>
      <div class="bg-white mb-4 py-1 justify-center items-start dark:bg-#121212">
        <div class="px-4">
          <div class="text-lg">
            thông tin chung
          </div>
        </div>
        <div class="flex px-4 justify-start items-center">
          <div class="min-w-14">
            tên
          </div>
          <input v-model="courseTitle" class="w-full" type="text" placeholder="Nhập tên khóa học (bắt buộc)">
        </div>
      </div>

      <template v-for="(courseItem, courseIndex) of courseList" :key="courseIndex">
        <div class="bg-white flex flex-col mb-4 py-1 gap-2 justify-center dark:bg-#121212">
          <div class="flex px-4 justify-between items-center">
            <div class="text-lg">
              {{ `Khoảng thời gian${courseIndex + 1}` }}
            </div>
            <div class="text-red-500 i-carbon-delete" @click="handleDeleteCourseItem(courseIndex)" />
          </div>
          <div class="flex px-4 justify-start items-center">
            <div class="min-w-14">
              Địa điểm
            </div>
            <input v-model="courseItem.location" class="w-full" type="text" placeholder="Nhập địa điểm lớp học (tùy chọn)">
          </div>
          <div class="flex px-4 justify-start items-center">
            <div class="min-w-14">
              Số tuần
            </div>
            <div class="w-full" @click="handleShowWeekActionSheet(courseIndex)">
              {{ transformArray2String(courseItem.weeks) }}
            </div>
          </div>
          <div class="flex px-4 justify-start items-center">
            <div class="min-w-14">
             thời gian
            </div>
            <div class="w-full" @click="handleShowTimeActionSheet(courseIndex)">
              {{ `${timeList[0][courseItem.week]} ${courseItem.start}-${courseItem.start + courseItem.duration - 1}` }}
            </div>
          </div>
        </div>
      </template>

      <div class="flex flex-col pb-safe gap-1 justify-center">
        <div
          class="flex bg-green-500 h-12 text-white text-center justify-center items-center"
          hover-class="bg-green-600" :hover-stay-time="150" @click="handleAddNewTime"
        >
          <div class="i-carbon-add" />
          Thêm một khoảng thời gian khác
        </div>
        <div
          class="flex bg-blue-500 h-12 text-white text-center justify-center items-center"
          hover-class="bg-blue-600" :hover-stay-time="150" @click="handleSaveCourse"
        >
          giữ
        </div>
      </div>
    </div>

    <!-- week action sheet -->
    <div @touchmove.prevent>
      <div
        class="bg-white w-full min-h-10 transition-all ease-in-out z-100 duration-300 fixed dark:bg-#121212"
        :class="showWeekActionSheet ? 'bottom-0' : '-bottom-full'"
      >
        <div class="flex flex-col py-6 gap-6">
          <div class="flex font-medium text-xl px-4 justify-between items-center">
            Chọn tuần học
            <div class="font-normal text-base text-green-500" @click="handleConfirmWeekActionSheet">
              Chắc chắn
            </div>
          </div>
          <div class="grid p-1 gap-1 grid-cols-5 justify-center items-center">
            <template v-for="(item, index) of 20" :key="index">
              <div
                class="flex h-8 text-center text-white transition-all inline-block justify-center items-center"
                :class="clickedWeeks.includes(index + 1) ? 'bg-blue-500' : 'bg-gray-300'"
                @click="handleClickWeek(index + 1)"
              >
                {{ item }}
              </div>
            </template>
          </div>
        </div>
        <div
          class="flex pb-safe border-t-4 border-gray-200 h-12 text-lg justify-center items-center dark:border-opacity-20"
          hover-class="bg-gray-200 bg-opacity-50" :hover-stay-time="150" @click="showWeekActionSheet = false"
        >
          đóng cửa
        </div>
      </div>
      <div
        class=" bg-dark-100 bg-opacity-50 transition-all top-0 right-0 bottom-0 left-0 z-90 fixed"
        :class="showWeekActionSheet ? 'opacity-100 visible' : 'opacity-0 invisible'"
        @click="showWeekActionSheet = false"
      />
    </div>

    <!-- time action sheet -->
    <div @touchmove.prevent>
      <div
        class="bg-white w-full min-h-10 transition-all ease-in-out z-100 duration-300 fixed dark:bg-#121212"
        :class="showTimeActionSheet ? 'bottom-0' : '-bottom-full'"
      >
        <div class="flex flex-col py-6 gap-6">
          <div class="flex font-medium text-xl px-4 justify-between items-center">
            Chọn tuần học
            <div class="font-normal text-base text-green-500" @click="handleConfirmTimeActionSheet">
              Chắc chắn
            </div>
          </div>
          <picker-view class="h-40" :value="timeValue" @change="handleTimeChange">
            <picker-view-column>
              <div v-for="(item, index) in timeList[0]" :key="index" class="flex justify-center items-center">
                {{ item }}
              </div>
            </picker-view-column>
            <picker-view-column>
              <div v-for="(item, index) in timeList[1]" :key="index" class="flex justify-center items-center">
                {{ item }}
              </div>
            </picker-view-column>
            <picker-view-column>
              <div v-for="(item, index) in timeList[2]" :key="index" class="flex justify-center items-center">
                {{ item }}
              </div>
            </picker-view-column>
          </picker-view>
        </div>
        <div
          class="flex pb-safe border-t-4 border-gray-200 h-12 text-lg justify-center items-center dark:border-opacity-20"
          hover-class="bg-gray-200 bg-opacity-50" :hover-stay-time="150" @click="showTimeActionSheet = false"
        >
          đóng cửa
        </div>
      </div>
      <div
        class=" bg-dark-100 bg-opacity-50 transition-all top-0 right-0 bottom-0 left-0 z-90 fixed"
        :class="showTimeActionSheet ? 'opacity-100 visible' : 'opacity-0 invisible'"
        @click="showTimeActionSheet = false"
      />
    </div>
  </UBasePage>
</template>

