<template>
  <div class="search">
    <div class="search-box-wrapper">
      <search-box ref="searchBox" @query="onQueryChange"></search-box>
    </div>
    <div ref="shortcutWrapper" class="shortcut-wrapper" v-show="!query">
      <scroll ref="shortcut" class="shortcut"
              :data="shortcut" :refreshDelay="refreshDelay">
        <div>
          <div class="hot-key">
            <h1 class="title">Pencarian populer</h1>
            <ul>
              <li @click="addQuery(item.k)" v-for="item in hotKey" class="item">
                <span>{{ item.k }}</span>
              </li>
            </ul>
          </div>
          <div class="search-history" v-show="searchHistory.length">
            <h1 class="title">
              <span class="text">Riwayat pencarian</span>
              <span class="clear" @click="showConfirm">
                <i class="icon-clear"></i>
              </span>
            </h1>
            <search-list @select="addQuery" @delete="deleteSearchHistory" :searches="searchHistory"></search-list>
          </div>
        </div>
      </scroll>
    </div>
    <div ref="searchResult" class="search-result" v-show="query">
      <suggest ref="suggest" @listScroll="blurInput" @select="saveSearch" :query="query"></suggest>
    </div>
    <confirm ref="confirm" @confirm="clearSearchHistory" title="Apakah akan menghapus semua riwayat pencarian" confirmBtnText="Kosong"></confirm>
    <router-view></router-view>
  </div>
</template>


<script type="text/ecmascript-6">
  import SearchBox from 'base/search-box/search-box'
  import Suggest from 'components/suggest/suggest'
  import SearchList from 'base/search-list/search-list'
  import Confirm from 'base/confirm/confirm'
  import Scroll from 'base/scroll/scroll'
  import { getHotKey } from 'api/search'
  import { ERR_OK } from 'api/config'
  import { mapActions } from 'vuex'
  import { playlistMixin, searchMixin } from 'common/js/mixin'

  export default {
    data() {
      return {
        hotKey: []
      }
    },
    /* 使用了跟搜索相关的混合，即和添加歌曲组件共用了搜索相关的逻辑 */
    mixins: [playlistMixin, searchMixin],
    computed: {
      /* 将热门搜索数据和搜索历史数据连接起来传入scroll
         组件的data接口，让其在数据发生变化时刷新高度 */
      shortcut() {
        return this.hotKey.concat(this.searchHistory)
      }
    },
    created() {
      this._getHotKey()
    },
    watch: {
      /* 当query发生变化，且变为空字符串时，刷新scroll组件的高度 */
      query(newQuery) {
        if (!newQuery) {
          setTimeout(() => {
            this.$refs.shortcut.refresh()
          }, 20)
        }
      }
    },
    methods: {
      /* 点击搜索历史的清空按钮时，显示弹窗 */
      showConfirm() {
        this.$refs.confirm.show()
      },
      /* 这里使用mixin根据播放器的播放列表解决搜索页面底部与mini播放器不适配的问题 */
      handlePlaylist(playlist) {
        const bottom = playlist.length > 0 ? '60px' : ''
        this.$refs.shortcutWrapper.style.bottom = bottom
        this.$refs.shortcut.refresh()
        this.$refs.searchResult.style.bottom = bottom
        this.$refs.suggest.refresh()
      },
      /* 获取热门搜索词 */
      _getHotKey() {
        getHotKey().then((res) => {
          if (res.code === ERR_OK) {
            this.hotKey = res.data.hotkey.slice(0, 10)
          }
        })
      },
      ...mapActions([
        'clearSearchHistory'
      ])
    },
    components: {
      SearchBox,
      Suggest,
      SearchList,
      Confirm,
      Scroll
    }
  }
</script>


<style scoped lang="stylus" rel="stylesheet/stylus">
  @import '~common/stylus/variable'
  @import '~common/stylus/mixin'

  .search
    .search-box-wrapper
      margin: 20px
    .shortcut-wrapper
      position: fixed
      top: 178px
      bottom: 0
      width: 100%
      .shortcut
        height: 100%
        overflow: hidden
        .hot-key
          margin: 0 20px 20px 20px
          .title
            margin-bottom: 20px
            font-size: $font-size-medium
            color: $color-text-l
          .item
            display: inline-block
            margin: 0 20px 10px 0
            padding: 5px 10px
            border-radius: 6px
            font-size: $font-size-medium
            color: $color-text-d
            background: $color-highlight-background
        .search-history
          position: relative
          margin: 0 20px
          .title
            display: flex
            align-items: center
            height: 40px
            font-size: $font-size-medium
            color: $color-text-l
            .text
              flex: 1
            .clear
              extend-click()
              .icon-clear
                font-size: $font-size-medium
                color: $color-text-d
    .search-result
      position: fixed
      top: 178px
      bottom: 0
      width: 100%
</style>
