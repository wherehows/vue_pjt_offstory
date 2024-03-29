<template>
  <Card>
    <section class="comments">
      <h1 class="comments__header">{{ comments.length }}개의 댓글</h1>
      <ul class="comments__list">
        <li
          ref="item"
          class="item"
          v-for="(commentList, _id) in comments"
          :key="_id"
          :commentId="commentList._id"
          :commentorId="commentList.author._id"
        >
          <div class="left">
            <button class="user__profile">
              <img
                :src="commentList.author.coverImage || defaultProfileUrl"
                alt="유저 프로필"
              />
            </button>
            <div class="user__infos">
              <strong class="nickname"
                ><span class="txt">{{ commentList.author.fullName }}</span>
                <span class="uploadDate">{{
                  timeForToday(commentList.updatedAt)
                }}</span></strong
              >
              <p v-html="putBr(commentList.comment)" class="content"></p>
            </div>
          </div>
          <div class="right">
            <div v-if="commentList.author._id === userId" class="edit-area">
              <button @click="deleteComments($event)" aria-label="삭제">
                삭제
              </button>
            </div>
          </div>
        </li>
        <li class="item etc">
          <div class="left">
            <button class="user__profile">
              <img :src="returnUserProfileImg" alt="유저 프로필" />
            </button>
            <div class="user__infos">
              <template v-if="isLogin">
                <strong class="nickname">
                  {{ userName }}
                </strong>
                <textarea
                  v-model="comment"
                  @input="resizeContent($event)"
                  @keyup.enter.exact="submitComment"
                  ref="comment"
                  class="add__comment"
                  type="text"
                  placeholder="댓글 달기"
                ></textarea>
              </template>

              <button v-else class="button--logouted" @click="recommendToLogin">
                로그인 해주세요
              </button>
            </div>
          </div>
          <div v-if="isLogin" class="right">
            <Button
              v-if="isCommentLength"
              @click="submitComment"
              v-bind="{
                width: '80px',
                height: '40px',
                boxShadow: 'none',
              }"
              ><i class="material-icons">arrow_forward</i></Button
            >
            <Button
              v-else
              v-bind="{
                width: '80px',
                height: '40px',
                backgroundColor: '#CCCCCC',
                boxShadow: 'none',
              }"
              disabled
              ><i class="material-icons">arrow_forward</i></Button
            >
          </div>
        </li>
      </ul>
    </section>
  </Card>
</template>

<script>
import { mapGetters } from 'vuex'

import { timeForToday, putBr } from '~/utils/function'
import { createComment, deleteComment } from '~/api/postContent'
import Card from '~/components/designs/Card.vue'
import Button from '~/components/designs/Button.vue'
import LoadingSpinner from '~/components/designs/LoadingSpinner'

export default {
  data() {
    return {
      comment: '',
      isSubmit: false,
    }
  },
  components: {
    Card,
    Button,
  },
  props: ['postId', 'comments', 'author', 'userId', 'userName', 'userImage'],
  emits: ['rerender'],
  computed: {
    ...mapGetters('Login', ['isLogin']),
    defaultProfileUrl() {
      return require('~/assets/images/user-profile__default.svg')
    },
    returnUserProfileImg() {
      return !this.userImage ? this.defaultProfileUrl : this.userImage
    },
    isCommentLength() {
      return this.comment.trim().length
    },
    checkHost() {
      return this.author._id === this.userId
    },
  },
  methods: {
    async submitComment() {
      if (this.doubleSubmitCHeck()) {
        this.isSubmit = false
        return
      }
      const commentValue = this.$refs.comment.value
      if (!commentValue.trim().length) return
      const userData = {
        comment: commentValue,
        postId: this.postId,
      }
      this.isLoading = true
      const res = await createComment(userData)
      await this.$emit('rerender')
      this.comment = ''
      this.isLoading = false
      //낙관적 업데이트
      const Listdata = this.$storage.getItem('PostListData')
      for (let i in Listdata) {
        console.log(Listdata[i]._id)
        if (Listdata[i]._id === this.postId) {
          Listdata[i].comments.push('dummy')
        }
      }
      console.log(Listdata)
      // this.$storage.removeItem('PostListData')
      this.$storage.setItem('PostListData', Listdata)
    },
    async deleteComments(event) {
      if (window.confirm('댓글을 삭제하시겠어요?')) {
        const li = event.target.closest('li')
        const commentId = li.getAttribute('commentId')
        const userData = {
          id: commentId,
        }
        this.isLoading = true
        console.log('삭제')
        const Listdata = this.$storage.getItem('PostListData')
        console.log('삭제시', Listdata)
        //낙관적 업데이트
        for (let i in Listdata) {
          console.log(Listdata[i]._id)
          if (Listdata[i]._id === this.postId) {
            console.log(Listdata[i].comments.length)
            Listdata[i].comments.splice(Listdata[i].comments.length - 1, 1)
            console.log(Listdata[i].comments)
          }
        }
        this.$storage.setItem('PostListData', Listdata)
        await deleteComment({ data: userData })

        window.alert('댓글이 삭제되었어요')
        this.$emit('rerender')
      } else {
        return
      }
      this.isLoading = false
    },
    checkCommentor(event) {
      const li = event.target.closest('li')
      const commentorId = li.getAttribute('commentorId')
      return commentorId === this.userId
    },
    doubleSubmitCHeck() {
      if (this.isSubmit) {
        return this.isSubmit
      } else {
        this.isSubmit = true
        return false
      }
    },
    resizeContent(event) {
      event.target.style.height = '1px'
      event.target.style.height = 20 + event.target.scrollHeight + 'px'
    },
    recommendToLogin() {
      if (window.confirm('로그인 페이지로 이동하시겠어요?')) {
        this.$router.push('/login')
      }
    },
    timeForToday,
    putBr,
  },
}
</script>

<style lang="scss" scoped>
.hidden {
  display: none;
}

.comments {
  min-height: 160px;

  &__header {
    font-size: $FONT_L;
    font-weight: 700;
    padding: $INNER_PADDING_VERTICAL 0;
  }

  &__list {
    .item {
      @include flexbox($jc: between, $ai: stretch);
      min-height: 60px;
      padding: calc($INNER_PADDING_VERTICAL / 2) 0;

      & {
        border-bottom: 1px solid $COLOR_BORDER;
      }

      &.etc {
        .left {
          width: 100%;

          .user__infos {
            /* width: 350px; */
            height: 100%;
          }
        }

        .right {
          position: relative;
          justify-content: center;

          .button {
            margin-top: $LG_PADDING_VERTICAL;
          }
        }
      }

      .left {
        @include flexbox($jc: start);

        .button--logouted {
          width: 100%;
          height: 100%;
          border-radius: $BORDER_RADIOUS;
          text-align: left;
          transition: background-color 300ms;

          &:hover {
            background-color: $COLOR_BORDER;
          }
        }

        .user__infos {
          flex-grow: 1;
          .nickname {
            @include flexbox($jc: start);
            .txt {
              display: inline-block;
            }
            margin-bottom: $INNER_PADDING_SMALL;
          }

          .uploadDate {
            flex-shrink: 0;
            color: $KEY_COLOR;
            font-size: $FONT_XS;
            font-weight: 400;
            text-align: start;
            margin-left: $INNER_PADDING_SMALL;
          }

          .content {
            line-height: 1.3em;
          }

          .add__comment {
            font-family: inherit;
            width: 100%;
            border: none;
            font-size: inherit;
            resize: none;
          }
        }
      }

      .right {
        @include flexbox($jc: between);
        flex-direction: column;
        flex-shrink: 0;
      }

      .edit-area {
        width: 100%;
        text-align: end;

        button {
          color: $COLOR_GRAY_DARKEN;
          font-size: $FONT_XS;
          cursor: pointer;
        }
      }
    }

    .user {
      &__profile {
        flex-shrink: 0;
        border-radius: 50%;
        overflow: hidden;
        margin-right: $INNER_PADDING_HORIZONTAL;
        img {
          @include flexbox;
          width: 35px;
          height: 35px;
          object-fit: cover; // 일단은 contain으로 해놓음
        }
      }
    }
  }

  @include responsive('sm') {
    .comments {
      &__header {
        font-size: $FONT_L;
      }

      /* depth가 너무 길어짐 3depth 이상 들어가지 않게 이후 수정... */
      &__list {
        .item {
          .left {
            .user {
              &__profile {
                img {
                  width: $SM_PROFILE_SIZE;
                  height: $SM_PROFILE_SIZE;
                }
              }
              &__infos {
                .content,
                .add__comment {
                  font-size: $FONT_S;
                }
              }
            }
          }

          .right {
            justify-content: center;
            .button {
              width: 50px !important;
            }
          }
        }
      }
    }
  }
}
</style>
