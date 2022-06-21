<template>
  <div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">
          {{ currentItem.name }}
        </p>
      </header>
      <section class="modal-card-body preview">
        <template>
          <prism-editor v-model="content" language="md" :readonly="!can('write')" :line-numbers="lineNumbers" />
        </template>
      </section>
      <footer class="modal-card-foot">
        <button class="button" type="button" @click="copyContent()">
          {{ lang('Copy') }}
        </button>
        <button v-if="can('write')" class="button" type="button" @click="saveFile()">
          {{ lang('Save') }}
        </button>
        <button class="button" type="button" @click="$parent.close()">
          {{ lang('Close') }}
        </button>
      </footer>
    </div>
  </div>
</template>

<script>
import api from '../../api/api'
import 'prismjs'
import 'prismjs/themes/prism.css'
import 'vue-prism-editor/dist/VuePrismEditor.css'
import PrismEditor from 'vue-prism-editor'

export default {
  name: 'Editor',
  components: { PrismEditor },
  props: [ 'item' ],
  data() {
    return {
      content: '',
      currentItem: '',
      lineNumbers: true,
    }
  },
  mounted() {
    this.currentItem = this.item
    api.downloadItem({
      path: this.item.path,
    })
      .then((res) => {
        this.content = res
      })
      .catch(error => this.handleError(error))
  },
  methods: {
    saveFile() {
      api.saveContent({
        name: this.item.name,
        content: this.content,
      })
        .then(() => {
          this.$toast.open({
            message: this.lang('Updated'),
            type: 'is-success',
          })
          this.$parent.close()
        })
        .catch(error => this.handleError(error))
    },
    copyContent() {
        const textarea = document.createElement('textarea')
        // 将该 textarea 设为 readonly 防止 iOS 下自动唤起键盘，同时将 textarea 移出可视区域
        textarea.readOnly = 'readonly'
        textarea.style.position = 'absolute'
        textarea.style.left = '-9999px'
        // 将要 copy 的值赋给 textarea 标签的 value 属性  
        // 网上有些例子是赋值给innerText,这样也会赋值成功，但是识别不了\r\n的换行符，赋值给value属性就可以
        textarea.value = this.content
        // 将 textarea 插入到 body 中
        document.body.appendChild(textarea)
        // 选中值并复制
        textarea.select()
        textarea.setSelectionRange(0, textarea.value.length)
        document.execCommand('Copy')
        document.body.removeChild(textarea)
    }
  },
}
</script>

<style scoped>
@media (min-width: 1100px) {
  .modal-card {
    width: 100%;
    min-width: 640px;
  }
}

.preview {
  min-height: 450px;
}
</style>
