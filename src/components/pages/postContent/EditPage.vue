<template>
  <form @submit.prevent="submitInfo" class="editor">
    <div class="edit-area">
      <button @click="$emit('cancelEdit')">취소</button>
      <button>저장</button>
    </div>
    <input
      v-focus
      @change="loadFile($event)"
      multiple
      accept="image/*"
      type="file"
      class="inputfile"
    />
    <input
      v-focus
      class="editor__title"
      :value="title"
      @input="changeTitle($event)"
      @keyup.enter="submitInfo"
    />
    <img v-if="postImgUrl" class="postImg" :src="postImgUrl" alt="" />
    <textarea
      @input="changeContent($event)"
      @keyup="resizeContent($event)"
      class="content"
      type="text"
      :value="content"
    ></textarea>
  </form>
</template>

<script>
import { updatePost } from '~/api/postContent'
export default {
  directives: {
    focus: {
      mounted(el) {
        el.focus()
      },
    },
  },
  emits: ['rerender'],
  props: {
    postId: {
      type: String,
      default: '',
      required: true,
    },
    channel: {
      type: Object,
      default: () => {},
      required: true,
    },
    initialTitle: {
      type: String,
      default: '',
      required: true,
    },
    initialContent: {
      type: String,
      default: '',
      required: true,
    },
    initialPostImgUrl: {
      type: String,
      default: '',
      required: true,
    },
  },
  data() {
    // 수정할 값들은 props에서 data로 받아옴
    return {
      title: this.initialTitle,
      content: this.initialContent,
      image: '',
      postImgUrl: this.initialPostImgUrl,
    }
  },
  methods: {
    changeTitle(event) {
      this.title = event.target.value
    },
    changeContent(event) {
      this.content = event.target.value
    },
    async submitInfo() {
      const userData = new FormData()
      userData.append('postId', this.postId)
      userData.append('title', `${this.title}/${this.content}`)
      userData.append('image', this.image || null)
      userData.append('channelId', this.channel._id)

      const res = await updatePost(userData)
      await this.$emit('rerender')
      await this.$emit('saveEdit')
    },
    loadFile(event) {
      const file = event.target.files[0]

      const newImageSrc = URL.createObjectURL(file)
      this.postImgUrl = newImageSrc
      this.image = file
    },
    resizeContent(event) {
      event.target.style.height = '1px'
      event.target.style.height = 20 + event.target.scrollHeight + 'px'
    },
  },
}
</script>

<style lang="scss" scoped>
.editor {
  height: calc(100% - $LG_HEADER_HEIGHT);
  display: flex;
  flex-direction: column;

  &__title {
    padding: $INNER_PADDING_VERTICAL 0;
    font-size: $FONT_XL;
    border: transparent;
    font-weight: 700;
  }

  .postImg {
    position: relative;
    left: 50%;
    transform: translateX(-50%);
    width: $EDIT_BASE_SIZE;
    height: $EDIT_BASE_SIZE;
  }

  .content {
    flex-grow: 1;
    min-height: $EDIT_BASE_SIZE;
    padding: $INNER_PADDING_VERTICAL 0;
    border: transparent;
    font-size: $FONT_L;
  }

  .edit-area {
    position: absolute;
    top: 0;
    right: 0;

    button {
      color: $COLOR_GRAY_DARKEN;

      &:first-child {
        margin-right: $INNER_PADDING_HORIZONTAL;
      }
    }
  }
}
</style>
