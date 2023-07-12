<template>
  <div class="list-group" ref="group">
    <div
      v-for="(item) in list" :key="item.id"
      class="list-group-item"
      draggable="false"
      style=""
      :class="{ghost: item.id === (ghostItem && ghostItem.id)}"
      @mousedown="mouseDownHandler($event, item)"
    >
      {{ item.name }}
    </div>
  </div>

  <div
    v-if="chosenItem"
    @mousemove="mouseMoveHandler"
    @mouseup="mouseUpHandler"
    class="cover"
  >
    <div
      class="chosen-item list-group-item"
      :style="{
      top: chosenItem.top + 'px',
      left: chosenItem.left + 'px',
      width: chosenItem.width + 'px',
      height: chosenItem.height + 'px',
    }"
    >
      {{ chosenItem.name }}
    </div>
  </div>

</template>

<script setup lang="ts">
import {reactive, ref} from 'vue'

const list = reactive([
  {
    "name": "John",
    "id": 0
  },
  {
    "name": "Joao",
    "id": 1
  },
  {
    "name": "Jean",
    "id": 2
  }
])
let chosenItem: object = ref(null)
let group = ref(null)
let ghostItem = ref({})
let isChosenItem = false

const mouseDownHandler = (event: Event, item: Object = {}) => {
  isChosenItem = true
  ghostItem.value = list?.find(e => e.id === item.id)
  chosenItem.value = {
    ...item,
    top: event?.target?.offsetTop,
    left: event?.target?.offsetLeft,
    height: event?.target?.offsetHeight,
    width: event?.target?.offsetWidth,
    touchX: event.clientX - event?.target?.offsetLeft,
    touchY: event.clientY - event?.target?.offsetTop,
    startLeft: event?.target?.offsetLeft,
    startTop: event?.target?.offsetTop,
  }
}

const mouseMoveHandler = (event) => {
  if (!isChosenItem) {
    return
  }
  chosenItem.value.top = event.clientY - chosenItem.value.touchY
  chosenItem.value.left = event.clientX - chosenItem.value.touchX

  Array.from(group.value?.children).forEach((e, index) => {
    if (Math.abs(chosenItem.value.top - e?.getBoundingClientRect().top) < 25) {
      let changePlaceItemIndex = list.findIndex(el => el.id === ghostItem.value.id)
      let changePlaceItem = list.find(el => el.id === ghostItem.value.id)
      if (index === changePlaceItemIndex) {
        return
      }

      list[changePlaceItemIndex] = list[index]
      list[index] = changePlaceItem
    }
  })

}

const mouseUpHandler = () => {
  isChosenItem = false

  let changePlaceItemIndex = list.findIndex(el => el.id === ghostItem.value.id)
  chosenItem.value.top = group.value?.children[changePlaceItemIndex].offsetTop
  chosenItem.value.left = group.value?.children[changePlaceItemIndex].offsetLeft

  document.querySelector('.chosen-item')?.classList.add('animated')

  setTimeout(() => {
    ghostItem.value = {}
    chosenItem.value = null
  }, 200)
}


</script>

<style scoped>
.list-group {
  width: 300px;
  display: -ms-flexbox;
  display: -webkit-box;
  display: flex;
  -ms-flex-direction: column;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  flex-direction: column;
  padding-left: 0;
  margin-bottom: 0;
  font-family: -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,"Noto Sans",sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol","Noto Color Emoji";
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
}
.list-group-item:first-child {
  border-top-left-radius: 0.25rem;
  border-top-right-radius: 0.25rem;
}
.list-group-item:last-child {
  margin-bottom: 0;
  border-bottom-right-radius: 0.25rem;
  border-bottom-left-radius: 0.25rem;
}
.list-group-item {
  position: relative;
  display: block;
  padding: 0.75rem 1.25rem;
  margin-bottom: -1px;
  background-color: #fff;
  border: 1px solid rgba(0,0,0,.125);
  -webkit-user-select: none;  /* Chrome all / Safari all */
  -moz-user-select: none;     /* Firefox all */
  -ms-user-select: none;      /* IE 10+ */
  user-select: none;
  cursor: move;
  text-align: left;
  color: #212529;
}
.animated {
  transition: all 0.2s;
}
.ghost {
  opacity: 0.5;
  background: #c8ebfb;
}
.chosen-item {
  position: absolute;
  z-index: 1;
  /*background-color: red;*/
  box-sizing: border-box;
}
.cover {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
}
</style>
