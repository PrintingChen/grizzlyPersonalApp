<template>
  <div class="content">
    <!-- <Tabs :value="active" inverse indicator-color="#009688" color="#009688" text-color="rgba(0, 0, 0, .54)" full-width>
      <Tab v-for="(messageList,index) in lists" :key="index" @click="changeTab(index)">
        {{messageList.name}}
        <i class='message-tips'>{{messageList.list.length}}</i>
      </Tab>
    </Tabs> -->
    <!-- <TabContainer v-model="active" swipeable class="list-con">
      <TabContainerItem v-for="(messageList,index) in lists" :key="index" :id="index" class="message-list"> -->
    <!-- <LoadMore :ref="'container' + index" @refresh="refresh(index)" @load="load(index)" :refreshing="messageList.refreshing" :loading="messageList.loading">
      <div class="message" v-for="message in messageList.list" :key="message.id"> -->
    <div ref="container0" class="list-con">
      <LoadMore class="mu-stepper mu-stepper-vertical" @refresh="refresh(0)" @load="load(0)" :refreshing="lists[0].refreshing" :loading="lists[0].loading">
        <div v-for="row in lists[0].list" :key="row.id" class="mu-step">
          <span class="mu-step-label active completed">
            <span class="mu-step-label-icon-container">
              <div class="stepIcon"></div>
            </span>
            <div class="stepHeader">{{row.head}}</div>
          </span>
          <div class="mu-step-content">
            <div style="position: relative; overflow: hidden; height: 100%;">
              <div class="mu-step-content-inner" data-old-padding-top="" data-old-padding-bottom="" data-old-overflow="" style="">
                <div class="stepConTitle">{{row.title}}</div>
                <div v-html="row.info" class="stepConInfo"></div>
              </div>
            </div>
          </div>
        </div>
      </LoadMore>
    </div>
    <!-- </TabContainerItem>
    </TabContainer> -->
  </div>
</template>

<script>
import service from 'service';
// import moment from 'moment';
import tools from 'util/tools';
import adapter from 'util/adapter';
import { LoadMore } from 'muse-ui';
// import { Tabs, LoadMore } from 'muse-ui';
// import { Tab } from 'muse-ui/lib/Tabs';
// import { TabContainer, TabContainerItem } from 'mint-ui';

export default {
  data () {
    return {
      // refreshing: false,
      // loading: false,
      active: 0,
      lists: [
        {
          name: '全部',
          refreshing: false,
          loading: false,
          page: 1,
          list: [
            {
              id: 0,
              head: '2017年6月8日',
              title: '本周在师傅指导下，学会留言板本周在师傅指导下，学会留言板本周在师傅指导下，学会留言板本周在师傅指导下，学会留言板',
              info: 'info'
            },
            {
              id: 1,
              head: '2017年6月8日',
              title: '本周在师傅指导下，学会留言板本周在师傅指导下，学会留言板本周在师傅指导下，学会留言板本周在师傅指导下，学会留言板',
              info: 'info'
            }
          ]
        }
      ]
    };
  },
  components: {
    // Tabs,
    // Tab,
    LoadMore
    // TabContainer,
    // TabContainerItem
  },
  computed: {
    isLoading () {
      return this.refreshing || this.loading;
    }
  },
  methods: {
    async doHasRead (message) {
      tools.showProgress();
      const response = await service.messageDoRead(message.id);
      tools.hideProgress();
      switch (response.code) {
        case 0:
          message.is_ready = 1;
          break;
        default:
          tools.toast({
            position: 'top',
            message: '标记失败，请稍后重试！！'
          });
          break;
      }
    },
    async getList (active) {
      const response = await service.getMessageList({
        page: this.lists[active].page
      });
      // console.log(JSON.stringify(response));
      if (this.lists[active].page === 1) {
        this.lists[active].refreshing = false;
      }
      if (this.lists[active].page > 1) {
        this.lists[active].loading = false;
      }
      switch (response.code) {
        case 0:
          if (this.lists[active].page === 1) {
            this.lists[active].list = response.result.lists.map(message =>
              adapter.messageAdapter(message)
            );
          }
          if (this.lists[active].page > 1) {
            response.result.lists.forEach(message => {
              this.lists[active].list.push(adapter.messageAdapter(message));
            });
          }
          break;
        default:
          tools.toast({
            position: 'top',
            message: '列表加载失败，请稍后重试！！'
          });
          break;
      }
    },
    changeTab (index) {
      // if (!this.isLoading) {
      this.active = index;
      // }
    },
    refresh (active) {
      if (!this.lists[active].refreshing && !this.lists[active].loading) {
        this.lists[active].page = 1;
        this.lists[active].refreshing = true;
        this.$refs[`container${active}`].scrollTop = 0;
        this.getList(active);
      }
    },
    load (active) {
      if (!this.lists[active].refreshing && !this.lists[active].loading) {
        this.lists[active].page = this.lists[active].page + 1;
        this.lists[active].loading = true;
        this.getList(active);
      }
    }
  },
  mounted () {
    this.refresh(this.active);
  }
};
</script>
<style lang="less">
@import url("../../../assets/css/base.less");
.unread {
  float: right;
  color: @danger;
}
.readed {
  float: right;
}
.content {
  height: 100%;
  display: flex;
  flex-direction: column;
}
.list-con {
  flex: 1;
  .mu-load-more {
    min-height: 200px;
  }
  .mint-tab-container-wrap {
    height: 100%;
  }
  .message-list {
    padding: 15px 0 0;
    height: 100%;
    overflow: auto;
    -webkit-overflow-scrolling: touch;
  }
}

.mu-tab-wrapper {
  flex-direction: row;
}

.message-tips {
  border-radius: 14px;
  color: #fff;
  background: red;
  font-style: normal;
  transform: scale(0.6);
  margin-left: 5px;
  padding: 6px;
  font-size: 16px;
  min-width: 14px;
  text-align: center;
  line-height: 14px;
  display: inline-block;
  min-width: 26px;
}

.stepHeader {
  font-size: 12px;
  color: #999;
}
.stepConInfo {
  color: #999;
}
.stepIcon {
  width: 16px;
  height: 16px;
  border-radius: 8px;
  background-color: #009688;
  border: 2px solid #B3DFDB;
  margin-left: 3px;
}
</style>
