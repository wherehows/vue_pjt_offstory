<template>
  <LoadingSpinner v-if="this.$store.getters['Loading/loading']" />

  <div class="container">
    <div class="searchresult-page">
      <div class="row">
        <div class="col-lg-13">
          <h1 class="searchresult-find">
            <template
              v-if="
                getUserCity === `undefined` || getUserCounty === `undefined`
              "
            >
              ""
            </template>
            <template v-else>
              "{{ getUserCity }} {{ getUserCounty }} {{ getdetailAddress }}"
            </template>
            <span>검색결과</span>
          </h1>
          <template v-if="getPostListData.length === 0">
            <div class="searchresult-find-noresult">
              <div class="nontext">
                검색결과가 없습니다. 동행을 직접 모집해보세요!
              </div>
              <Button
                @click="$router.push('/newpost')"
                v-bind="{ width: '20%' }"
                class="party"
                >동행 모집하러가기</Button
              >
            </div>
          </template>
          <template v-else>
            <div class="resultlist">
              <ul class="resultlist-listcards">
                <li v-for="i in getPostListData" :key="i">
                  <div
                    class="resultlist-listcard"
                    @click="
                      $router.push({
                        name: 'PostContent',
                        params: { postId: i._id },
                      })
                    "
                  >
                    <div class="resultlist-listcard-header">
                      <div class="resultlist-listcard-createtime">
                        {{ i.createdAt.substring(0, 10) }}
                      </div>
                      <div class="resultlist-listcard-timefortoday">
                        {{ timeForToday(i.createdAt) }}
                      </div>
                      <div class="resultlist-listcard-location">
                        {{ i.location }}
                      </div>
                    </div>
                    <div class="postimage">
                      <template v-if="i.image === undefined">
                        <!-- <img
                          class="postimage-default"
                          src="../assets/images/non_postimage.png"
                          alt=""
                        /> -->
                        <div class="postimage-default" alt=""></div>
                      </template>
                      <template v-else>
                        <img class="postimage-user" :src="`${i.image}`" />
                      </template>
                      <div class="resultlist-listcard-active">
                        <span class="hidden">1 </span>
                        <Like
                          :post="i"
                          :color="`white`"
                          :symbolSize="`18px`"
                          :fontSize="`18px`"
                        />

                        <Comment
                          :post="i"
                          :marginLeft="`10px`"
                          :color="`white`"
                          :symbolSize="`18px`"
                          :fontSize="`18px`"
                        />
                        <!-- <div class="resultlist-listcard-like"> -->
                        <!-- <img
                          class="resultlist-listcard-likeimage"
                          :src="heartImageUrl"
                          alt=""
                        /> -->
                        <!-- +{{ i.likes.length }} -->
                        <!-- </div> -->
                        <!-- <div class="resultlist-listcardt-comment"> -->
                        <!-- <img
                          class="resultlist-listcard-commentimage"
                          :src="commentImageUrl"
                          alt=""
                        /> -->
                        <!-- +{{ i.comments.length }} -->
                        <!-- </div> -->
                      </div>
                    </div>

                    <div class="resultlist-listcard-userinfo">
                      <div class="resultlist-listcard-userprofile">
                        <template v-if="i.author.coverImage === undefined">
                          <img
                            class="resultlist-listcard-userprofile-basicimage"
                            :src="imageUrl"
                            alt=""
                          />
                        </template>
                        <template v-else>
                          <img
                            class="resultlist-listcard-userprofile-userimage"
                            :src="`${i.author.coverImage}`"
                            alt=""
                          />
                        </template>
                        <div class="movie-result-author">
                          {{ i.author.fullName }}
                        </div>
                        <div class="resultlist-listcard-title">
                          {{ parseTitleContent(i.title) }}
                        </div>
                      </div>
                    </div>
                  </div>
                </li>
              </ul>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { timeForToday } from '~/utils/function'
import { mapState, mapActions, mapMutations } from 'vuex'
import LoadingSpinner from '~/components/designs/LoadingSpinner'
import {
  getAuth,
  createPost,
  channelsList,
  createChannel,
  channelPostList,
} from '../api/index'
import Button from '~/components/designs/Button'
import Like from '~/components/designs/Like'
import Comment from '~/components/designs/Comment'

export default {
  data() {
    return {
      searchCity: '',
      searchCounty: '',
      deatailAddress: '',
      channelId: '',
      postList: [],
      Title: '',
      imageUrl: require('~/assets/images/user-profile.svg'),
      heartImageUrl: require('../assets/images/postlist_cil_heart.svg'),
      commentImageUrl: require('../assets/images/comment.svg'),
    }
  },
  components: { Button, Like, Comment, LoadingSpinner },
  computed: {
    ...mapMutations('Loading', ['startLoading', 'endLoading']),

    getUserCity() {
      //let userCity=this.$storage.getItem('userCity')
      //this.$storage.removeItem('userCity')
      return this.$storage.getItem('userCity')
    },
    getUserCounty() {
      return this.$storage.getItem('userCounty')
    },
    getdetailAddress() {
      return this.$storage.getItem('userdetailAddress')
    },
    getSearchChannelId() {
      return this.$storage.getItem('channelId')
    },
    getPostListData() {
      //console.log(this.$store.getters['address/getPostListData'])
      return this.$storage.getItem('PostListData')
    },
    getLikeData() {
      let likeCount = this.$store.getters['address/getPostListData'].map(
        x => x.likes,
      )
      console.log(likeCount)
      return likeCount.length
    },
  },
  mounted() {
    // this.searchCity = this.getUserCity()
    // this.searchCity = this.$store.getters['address/getUserCity']
    // this.searchCounty = this.$store.getters['address/getUserCounty']
    // this.channelId = this.$store.getters['address/getSearchChannelId']
    // this.deatailAddress = this.$store.getters['address/getdetailAddress']
    // const poaslist = this.$store.getters['address/getPostListData']
    this.endLoading
  },
  async created() {
    // this.searchCity = this.$store.getters['address/getUserCity']
    // this.searchCounty = this.$store.getters['address/getUserCounty']
    // this.channelId = this.$store.getters['address/getSearchChannelId']
    // this.deatailAddress = this.$store.getters['address/getdetailAddress']
    // const postListdata = await channelPostList(this.channelId)
    // this.postList = postListdata.data
    // console.log(this.postList)
  },
  methods: {
    parseTitleContent(titledata) {
      //   const data = titleContent
      //   const titledata = data.map(x => x.title)
      const titleList = []
      console.log(titledata)
      titleList.push(titledata.split('/')[0])
      //   for (let i in titledata) {
      //     let titlestr = titledata[i]
      //     titleList = titlestr.split('/')[0]
      //   }
      console.log(titleList[0])

      return titleList[0]
    },
    timeForToday,
  },
}
</script>
<style lang="scss" scoped>
@include responsive('sm') {
  .container {
    z-index: 2;
    position: relative;
    top: $LG_HEADER_HEIGHT;

    .row {
      justify-content: center;
      .col-lg-13 {
        width: 100%;
      }
    }
    .searchresult-find {
      margin-top: 20px;
      padding-bottom: $INNER_PADDING_HORIZONTAL;
      color: $COLOR_GRAY_DARKEN;
      border-bottom: solid $COLOR_GRAY_DARKEN;
      font-size: $FONT_BASE;
      span {
        color: black;
      }
    }
    .searchresult-find-noresult {
      margin-top: 250px;
      display: flex;
      flex-direction: column;
      width: 100%;
      text-align: center;
      align-items: center;
      justify-content: center;
      .nontext {
        font-size: $FONT_L;
        color: $COLOR_GRAY_DARKEN;
      }
      .party {
        min-width: 100px;
        margin-top: 20px;
      }
    }
    .resultlist {
      display: flex;
      flex-direction: column;
      width: 100%;
      text-align: center;
      align-items: center;
      height: 100%;

      .resultlist-listcards {
        display: flex;
        height: 100%;
        width: 100%;
        flex-direction: column;
        text-align: center;
        align-items: center;

        .resultlist-listcard {
          position: relative;
          cursor: pointer;
          margin: 30px 10px 0px 10px;
          width: 300px;
          height: 365px;
          // border: solid 1px $COLOR_GRAY_LIGHTEN;
          // border-radius: 30px;
          padding: 20px;
          box-shadow: $BOX_SHADOW;
          .resultlist-listcard-header {
            display: flex;
            margin-bottom: 10px;
            .resultlist-listcard-createtime {
              width: 30%;
              font-size: $FONT_S;
              color: $COLOR_GRAY_DARKEN;
            }
            .resultlist-listcard-timefortoday {
              width: 20%;
              margin: 0px;
              padding: 0px;
              margin-top: 5px;
              font-size: 5px;
              color: $KEY_COLOR;
              //opacity: 0.8;
            }
            .resultlist-listcard-location {
              width: 130px;
              margin: 0px;
              padding: 0px;
              margin-left: 10px;
              font-size: $FONT_S;
              color: $COLOR_GRAY_DARKEN;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }
          }
          .postimage {
            border-radius: 10px;
            overflow: hidden;
            position: relative;
            height: $EDIT_BASE_SIZE;
            .postimage-default {
              border-radius: 10px;
              background-color: lighten(rgb(243, 237, 191), 10%);
              width: 100%;
              height: $EDIT_BASE_SIZE;
            }
            .postimage-user {
              object-fit: cover;
              border-radius: 10px;
              width: 100%;
              height: $EDIT_BASE_SIZE;
              filter: brightness(65%);
              transform: scale(1);
              -webkit-transform: scale(1);
              -moz-transform: scale(1);
              -ms-transform: scale(1);
              -o-transform: scale(1);
              transition: all 0.3s ease-in-out;

              &:hover {
                transform: scale(1);
                -webkit-transform: scale(1.1);
                -moz-transform: scale(1);
                -ms-transform: scale(1);
                -o-transform: scale(1.1);
              }
            }
          }
          .resultlist-listcard-active {
            .hidden {
              opacity: 0;
              position: absolute;
              display: inline-block;
              width: 34px;
              height: 30px;
              background-color: red;
            }
            position: absolute;
            display: flex;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.6);
            padding: 0 5px;
            border-radius: 5px;
          }
          .resultlist-listcard-userinfo {
            position: relative;
            top: -20px;
            display: flex;
            justify-self: center;
            //   align-content: center;
            flex-direction: column;
            .resultlist-listcard-userprofile {
              display: flex;
              align-items: center;
              flex-direction: column;
              .resultlist-listcard-userprofile-basicimage {
                width: 40px;
                height: 40px;
                background: white;
                border-radius: 30px;
              }
              .resultlist-listcard-userprofile-userimage {
                width: 40px;
                height: 40px;
                background: white;
                border-radius: 30px;
              }
              .movie-result-author {
                @include ellipsis($line: 1); //   text-overflow: ellipsis;
                font-size: 14px;
              }
              .resultlist-listcard-title {
                width: 300px;
                text-align: center;
                margin-top: 10px;
                font-size: $FONT_BASE;
                //   white-space: nowrap;
                //   overflow: hidden;
                @include ellipsis($line: 2); //   text-overflow: ellipsis;
              }
            }
          }
          &:hover {
            //filter: brightness(120%);
            opacity: 0.6;
          }
        }
      }
    }
  }
}
@media screen and (min-width: 769px) {
  .container {
    z-index: 2;
    position: relative;
    top: $LG_HEADER_HEIGHT;

    .row {
      justify-content: center;
      .col-lg-13 {
        width: 100%;
      }
    }
    .searchresult-find {
      margin-top: 20px;
      padding-bottom: $INNER_PADDING_HORIZONTAL;
      color: $COLOR_GRAY_DARKEN;
      border-bottom: solid $COLOR_GRAY_DARKEN;
      font-size: $FONT_L;
      span {
        color: black;
      }
    }
    .searchresult-find-noresult {
      margin-top: 250px;
      display: flex;
      flex-direction: column;
      width: 100%;
      text-align: center;
      align-items: center;
      justify-content: center;
      .nontext {
        font-size: $FONT_XXL;
        color: $COLOR_GRAY_DARKEN;
      }
      .party {
        margin-top: 20px;
      }
    }
    .resultlist {
      width: 100%;
      height: 100%;

      .resultlist-listcards {
        display: flex;
        flex-flow: wrap;
        height: 100%;
        width: 100%;
        justify-content: center;

        .resultlist-listcard {
          position: relative;
          cursor: pointer;
          margin: 30px 8px 0px 10px;
          width: 355px;
          height: 380px;
          // border: solid 1px $COLOR_GRAY_LIGHTEN;
          // border-radius: 30px;
          padding: 20px;
          box-shadow: $BOX_SHADOW;
          .resultlist-listcard-header {
            display: flex;
            margin-bottom: 10px;
            .resultlist-listcard-createtime {
              font-size: $FONT_BASE;
              color: $COLOR_GRAY_DARKEN;
            }
            .resultlist-listcard-timefortoday {
              margin: 0px;
              padding: 0px;
              margin-top: 4px;
              margin-left: 6px;
              font-size: $FONT_XS;
              color: $KEY_COLOR;
              //opacity: 0.8;
            }
            .resultlist-listcard-location {
              width: 130px;
              margin: 0px;
              padding: 0px;
              margin-left: 50px;
              font-size: $FONT_BASE;
              color: $COLOR_GRAY_DARKEN;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }
          }
          .postimage {
            border-radius: 10px;
            overflow: hidden;
            height: $EDIT_BASE_SIZE;
            position: relative;

            .postimage-default {
              background-color: lighten(rgb(243, 237, 191), 10%);
              border-radius: 10px;
              width: 100%;
              height: $EDIT_BASE_SIZE;
            }
            .postimage-user {
              object-fit: cover;
              border-radius: 10px;
              width: 100%;
              height: $EDIT_BASE_SIZE;
              filter: brightness(65%);
              transform: scale(1);
              -webkit-transform: scale(1);
              -moz-transform: scale(1);
              -ms-transform: scale(1);
              -o-transform: scale(1);
              transition: all 0.3s ease-in-out;

              &:hover {
                transform: scale(1);
                -webkit-transform: scale(1.1);
                -moz-transform: scale(1);
                -ms-transform: scale(1);
                -o-transform: scale(1.1);
              }
            }
          }
          .resultlist-listcard-active {
            .hidden {
              opacity: 0;
              position: absolute;
              display: inline-block;
              width: 34px;
              height: 30px;
              background-color: red;
            }
            position: absolute;
            display: flex;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.6);
            padding: 0 5px;
            border-radius: 5px;
          }
          /* .resultlist-listcard-active {
            display: flex;
            width: 120px;
            padding-left: 5px;
            .resultlist-listcard-like {
              color: $COLOR_GRAY_LIGHTEN;
              .resultlist-listcard-likeimage {
                color: $COLOR_GRAY_LIGHTEN;
                width: 15px;
                height: 15px;
                opacity: 0.6;
              }
            }
            .resultlist-listcardt-comment {
              margin-left: 10px;
              color: $COLOR_GRAY_LIGHTEN;
              .resultlist-listcard-commentimage {
                color: $COLOR_GRAY_LIGHTEN;
                width: 13px;
                height: 13px;
                opacity: 0.5;
              }
            }
          } */
          .resultlist-listcard-userinfo {
            position: relative;
            top: -20px;
            display: flex;
            justify-self: center;
            //   align-content: center;
            flex-direction: column;
            .resultlist-listcard-userprofile {
              display: flex;
              align-items: center;
              flex-direction: column;
              .resultlist-listcard-userprofile-basicimage {
                width: 40px;
                height: 40px;
                background: white;
                border-radius: 30px;
              }
              .resultlist-listcard-userprofile-userimage {
                width: 40px;
                height: 40px;
                background: white;
                border-radius: 30px;
              }
              .movie-result-author {
                @include ellipsis($line: 1) //   text-overflow: ellipsis;
;
              }
              .resultlist-listcard-title {
                width: 300px;
                text-align: center;
                margin-top: 10px;
                font-size: $FONT_L;
                //   white-space: nowrap;
                //   overflow: hidden;
                @include ellipsis($line: 2) //   text-overflow: ellipsis;
;
              }
            }
          }
          &:hover {
            //filter: brightness(120%);
            opacity: 0.6;
          }
        }
      }
    }
  }
}
</style>
