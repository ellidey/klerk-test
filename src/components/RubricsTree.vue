<script setup>
import Tree from 'vue3-treeview';
import "vue3-treeview/dist/style.css";
import axios from 'axios';
import {onMounted, ref} from 'vue';

const rubrics = ref({})
const roots = ref([])
const config = ref({
  roots: ['root'],
  keyboardNavigation: true,
  dragAndDrop: false,
  checkboxes: true,
  editable: false,
  disabled: false,
  padding: 25,
  checkMode: 0
})
const loading = ref(true)
const selectedCount = ref(0)
const allowEmpty = ref(false)

onMounted(async () => {
  await loadRubrics()
})

async function loadRubrics(allowEmpty = false) {
  loading.value = true

  await axios.get('https://www.klerk.ru/yindex.php/v3/event/rubrics', {
    params: {
      allowEmpty
    }
  }).then(({ data }) => {
    rubrics.value = {
      root: {
        title: 'Рубрики',
        text: '',
        count: 0,
        allCount: 0
      }
    }

    data.forEach(rubric => normalizeRubric(rubric))
    roots.value = data.map(item => {
      return 'id' + item.id
    })
  })
  rubrics.value.root.children = roots.value

  console.log(rubrics.value)
  loading.value = false
}

function normalizeRubric(rubric) {
  rubrics.value.root.allCount += rubric.count

  const result = {
    text: '',
    title: rubric.title,
    url: rubric.url,
    children: [],
    count: rubric.count,
    allCount: rubric.count
  }

  if (rubric.children) {
    rubric.children.forEach(child => {
      normalizeRubric(child)
      result.children.push('id' + child.id)
      result.allCount += Math.max(child.count)
    })
  }

  rubrics.value['id' + rubric.id] = result
}

function checkedRubric(rubric) {
  selectedCount.value += rubric.allCount
}

function uncheckedRubric(rubric) {
  selectedCount.value -= rubric.allCount
}

async function changeAllowEmpty() {
  await loadRubrics(!allowEmpty.value)
}
</script>

<template>
  <div v-if="loading" class="loader-container">
    <div class="loader"></div>
  </div>

  <div v-else>
    <div>
      <div class="filters">
        <input
            type="checkbox"
            id="allowEmpty"
            name="subscribe"
            v-model="allowEmpty"
            @input="changeAllowEmpty"
        />
        <label for="allowEmpty">Отображать пустые рубрики</label>
      </div>
    </div>
    Сумма выбранных элементов: {{ selectedCount }}


    <tree :nodes="rubrics" :config="config" @node-checked="checkedRubric" @node-unchecked="uncheckedRubric">
      <template #before-input="props">
        <template v-if="props.node.url">
          <a :href="'https://www.klerk.ru/' + props.node.url" target="_blank" class="before"> {{ props.node.title }} </a>
        </template>
        <template v-else>
          <span class="before"> {{ props.node.title }} </span>
        </template>
      </template>

      <template #after-input="props">
        <span class="after">
          ({{ props.node.count }} / {{ props.node.allCount }})
        </span>
      </template>
    </tree>
  </div>
</template>

<style scoped>
.filters {
  background-color: rgba(0, 0, 0, 0.2);
  font-size: 18px;
  padding: 20px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.loader-container {
  width: 100%;
  height: 100vh;
  top: 0;
  left: 0;
  overflow: hidden;
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}
.loader {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  position: relative;
  animation: rotate 1s linear infinite
}
.loader::before , .loader::after {
  content: "";
  box-sizing: border-box;
  position: absolute;
  inset: 0px;
  border-radius: 50%;
  border: 5px solid #FFF;
  animation: prixClipFix 2s linear infinite ;
}
.loader::after{
  transform: rotate3d(90, 90, 0, 180deg );
  border-color: #FF3D00;
}

@keyframes rotate {
  0%   {transform: rotate(0deg)}
  100%   {transform: rotate(360deg)}
}

@keyframes prixClipFix {
  0%   {clip-path:polygon(50% 50%,0 0,0 0,0 0,0 0,0 0)}
  50%  {clip-path:polygon(50% 50%,0 0,100% 0,100% 0,100% 0,100% 0)}
  75%, 100%  {clip-path:polygon(50% 50%,0 0,100% 0,100% 100%,100% 100%,100% 100%)}
}
</style>