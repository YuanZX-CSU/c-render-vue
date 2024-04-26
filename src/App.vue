<script setup>
import { computed, provide, ref } from 'vue';
import axios from 'axios'
import { base_url } from '@/url.js'
import { oj_base_url } from '@/url.js'
import CodeEditor from './components/CodeEditor.vue';
import StdinEditor from './components/StdinEditor.vue';
import ControlPanel from './components/ControlPanel.vue';
import CodeViewer from './components/CodeViewer.vue';
import InfoView from './components/InfoView.vue';
import StackView from './components/StackView.vue';
import CryptoJS from 'crypto-js';

const code_input = ref('') // 用户输入的代码
const stdin_input = ref('') // 用户输入的测试用例
function send_request() { // 将输入的代码/测试用例发送给后端
  data.value = {}
  let code = code_input.value
  code = code.replace(/ /g, ' ')
  if (stdin_input.value.length == 0) {
    axios.post(base_url + '/visualize', {
      code: code
    }, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
      .then(response => {
        data.value = response.data
        parse_result()
      })
      .catch(error => {
        data.value = {
          "trace": [
            {
              "event": "uncaught_exception",
              "exception_msg": "network error",
              "line": 1
            }
          ]
        }
        console.error(error);
        parse_result()
      });
  } else {
    axios.post(base_url + '/visualize', {
      code: code,
      stdin: stdin_input.value
    }, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
      .then(response => {
        data.value = response.data
        parse_result()
      })
      .catch(error => {
        data.value = {
          "trace": [
            {
              "event": "uncaught_exception",
              "exception_msg": "network error",
              "line": 1
            }
          ]
        }
        console.error(error);
        parse_result()
      });
  }
}
const data = ref({}) // 后端返回的数据
function parse_result() { // 解析后端返回的结果
  remove_back_slash_r()
  loading.value = false
  if (success.value) {
    total_steps.value = data.value['trace'].length
    try_failed.value = false
    edit_mode.value = false
  } else {
    try_failed.value = true
    total_steps.value = 1
  }
  current_step.value = 1
  highlight_lines.value = []
  highlight_steps.value = []
}
function remove_back_slash_r() { // 去除后端返回的code中的\r
  if ('code' in data.value) {
    let code = data.value['code']
    code = code.replace(/\r/g, '')
    data.value['code'] = code
  }
}
const success = computed(() => { // 获取后端返回的运行结果
  try {
    if (data.value['trace'][0]['event'] == 'step_line') {
      return true
    }
  } catch (e) { }
  return false
})
const exception_msg = computed(() => { // 获取后端返回的异常信息
  try {
    if (data.value['trace'][0]['event'] == 'uncaught_exception') {
      return data.value['trace'][0]['exception_msg'].replace(/`/g, "'").replace(/‘/g, "'").replace(/’/g, "'")
    }
  } catch (e) { }
  return ''
})
function get_step(_step) { // 获取某一步的运行结果(1~n)
  if (_step < 1) {
    _step = 1
  }
  if (_step > total_steps.value) {
    _step = total_steps.value
  }
  try {
    return data.value['trace'][_step - 1]
  } catch (e) { }
}
const current_line = computed(() => { // 播放的当前代码对应的行数
  try {
    return get_step(current_step.value)['line']
  } catch (e) { }
})
const prev_line = computed(() => { // 播放的下一步代码对应的行数
  try {
    return get_step(current_step.value - 1)['line']
  } catch (e) { }
})
const try_failed = ref(false) // 尝试运行代码失败
const total_steps = ref(1) // 总步骤数
const current_step = ref(1) // 当前步骤
const highlight_steps = ref([]) // 高亮的步骤
function click_step(clicked) { // 跳转到某一步骤
  current_step.value = clicked
}
const highlight_lines = ref([]) // 高亮的代码行
function enable_line(clicked) { // 高亮某一行
  let actived = 0;
  for (let i = 1; i <= total_steps.value; i++) {
    if (get_step(i)['line'] == clicked) {
      highlight_steps.value.push(i)
      actived += 1
    }
  }
  if (actived != 0) {
    highlight_lines.value.push(clicked)
  }
}
function disable_line(clicked) { // 取消高亮某一行
  for (let i = 1; i <= total_steps.value; i++) {
    if (get_step(i)['line'] == clicked) {
      highlight_steps.value = highlight_steps.value.filter(item => item != i)
    }
  }
  highlight_lines.value = highlight_lines.value.filter(item => item != clicked)
}
function click_line(clicked) { // 点击某一行
  if (Object.values(highlight_lines.value).includes(clicked)) {
    disable_line(clicked)
  } else {
    enable_line(clicked)
  }
}
const edit_mode = ref(true) // 编辑模式，对应的是播放模式
const loading = ref(false) // 发送请求等待后端返回的状态
const solution_output = ref('')
function getMD5(string){ // 获取字符串的md5
  return CryptoJS.MD5(string).toString(CryptoJS.enc.Hex)
}
function generate_random_string(length = 32) { // 生成32位随机字符串
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  for (let i = 0; i < length; i++) {
    result += characters.charAt(Math.floor(Math.random() * characters.length));
  }
  return result;
}
function query_oj_solution(solution_id){ // 查询solution信息
  const key = 'cd57c770-5384-493e-e231-51e74438287a'
  const nonce = generate_random_string()
  const appID = 'ProgrammingLearningJudgingPlatform'
  const timestamp = Math.floor(Date.now() / 1000)
  const join = 'solutionID=' + solution_id + '&key=' + key + '&nonce=' + nonce + '&appID=' + appID + '&timestamp=' + timestamp
  const signature = getMD5(join)
  axios.get(oj_base_url,{
    params: {
      solutionID: solution_id
    },
    headers: {
      'pljp-appID': appID,
      'pljp-timestamp': timestamp,
      'pljp-nonce': nonce,
      'pljp-signature': signature
    }
  })
  .then(response => {
    const resp = response.data
    if(resp.code == 10000){
      code_input.value = resp.data.code
      stdin_input.value = resp.data.input
      solution_output.value = resp.data.output
      loading.value = true
      try_failed.value = false
      send_request()
    }else{
      console.log(resp)
    }
  })
  .catch(error => {
    console.error(error);
  });
}
function get_oj_solution_id(){ // 从网址获取solutionID
  const currentUrl = window.location.href;
  const queryObject = new URLSearchParams(window.location.search);
  const solution_id = queryObject.get('solution');
  if(solution_id != null){
    query_oj_solution(solution_id)
  }
}
get_oj_solution_id()

const feature_show_new_vars = ref(false)

provide('app', {
  loading,
  code_input,
  stdin_input,
  send_request,
  data,
  success,
  exception_msg,
  get_step,
  current_line,
  prev_line,
  try_failed,
  total_steps,
  current_step,
  highlight_steps,
  click_step,
  highlight_lines,
  click_line,
  edit_mode,
  feature_show_new_vars,
  solution_output
})
</script>

<template>
  <div class="oj-logo">
    <img src="@/assets/ojlogo.png" style="height: 100%;">
  </div>

  <div class="out-out-container">
    <div class="out-container">
      <div class="out-big-box">
        <div class="big-box" v-show="edit_mode">
          <CodeEditor />
        </div>
        <div class="big-box" v-show="!edit_mode">
          <CodeViewer />
        </div>
      </div>

      <div class="out-big-box">
        <div class="big-box" v-show="edit_mode">
          <InfoView />
        </div>
        <div class="big-box" v-show="!edit_mode">
          <StackView />
        </div>
      </div>

      <div class="out-big-box">
        <div class="big-box">
          <ControlPanel />
        </div>
      </div>

      <div class="out-big-box">
        <div class="big-box">
          <StdinEditor />
        </div>
      </div>

    </div>
  </div>

</template>

<style scoped>
.oj-logo {
  height: 3rem;
  margin: 0;
  background-color: #186299;
}

.out-out-container {
  /* 黑魔法，勿动 */
  height: calc(100% - 4rem);
  width: calc(100% - 1rem);
  padding: 0.5rem;
  margin: 0;
}

.out-container {
  /* 黑魔法，勿动 */
  display: grid;
  grid-template-columns: 34% 66%;
  grid-template-rows: 74% 26%;
  height: 100%;
  width: 100%;
}

.out-big-box {
  /* 黑魔法，勿动 */
  padding: 0.5rem;
}

.big-box {
  /* 黑魔法，勿动 */
  border-radius: 1rem;
  height: calc(100% - 2rem);
  width: calc(100% - 2rem);
  padding: 1rem;
  background-color: #EEF4FA;
}
</style>

<style>
* {
  margin: 0;
  padding: 0;
  font-family: "Maple Mono SC Lite";
  font-weight: normal;
  color: #011E2B;
}

@font-face {
  font-family: "Maple Mono SC Lite";
  src: url("./assets/MapleMono-SC-Lite.woff2")format('woff2');
}

#app {
  height: 100vh;
}
</style>
