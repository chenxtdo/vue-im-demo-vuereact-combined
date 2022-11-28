<template>
  <div>
    <!-- 单呼组件 -->
    <CallViewProvider v-if="!!neCallConfig.nim" :neCallConfig="neCallConfig" ref="callViewProviderRef" />
    <!-- 群呼组件 -->
    <GroupCallViewProvider v-if="!!neCallConfig.nim" :neGroupCallConfig="neCallConfig" ref="groupCallViewProviderRef" />
    <div :class="$style.container">
      <!-- IMUIKIT 相关内容 -->
      <Provider v-if="!!nimKitCore" :nimKitCore="nimKitCore" :initOptions="initOptions">
        <div :class="$style.header">
          <div :class="$style.search">
            <SearchContainer />
          </div>
          <div :class="$style.add">
            <AddContainer />
          </div>
        </div>
        <div :class="$style.content">
          <div :class="$style.left">
            <div :class="$style['avatar-icon']">
              <MyAvatarContainer />
            </div>
            <div :class="{ [$style['chat-icon']]: true, [$style.active]: model === 'chat' }"
              @click="() => model = 'chat'">
              <SmileFilled />
              <div :class="$style['icon-label']">
                会话
              </div>
            </div>
            <div :class="{ [$style['contact-icon']]: true, [$style.active]: model === 'contact' }"
              @click="() => model = 'contact'">
              <MobileFilled />
              <div :class="$style['icon-label']">
                通讯录
              </div>
            </div>
            <div :class="$style['logout-icon']">
              <LoginOutlined />
              <div :class="$style['icon-label']">
                退出
              </div>
            </div>
          </div>
          <div :class="$style.right" v-if="model === 'chat'">
            <div :class="$style['right-list']">
              <ConversationContainer />
            </div>
            <div :class="$style['right-content']">
              <ChatContainer :actions="chatInputActions" />
            </div>
          </div>
          <div :class="$style.right" v-if="model === 'contact'">
            <div :class="$style['right-list']">
              <ContactListContainer />
            </div>
            <div :class="$style['right-content']">
              <ContactInfoContainer :afterSendMsgClick="() => model = 'chat'"
                :onGroupItemClick="() => model = 'chat'" />
            </div>
          </div>
        </div>
      </Provider>
    </div>
  </div>
</template>
<script>
// import Vue from 'vue'
import { applyReactInVue } from "vuereact-combined";
import {
  Provider, // 全局上下文
  ConversationContainer, // 会话列表组件
  ChatContainer, // 聊天（会话消息）组件
  AddContainer, // 搜索——添加按钮组件
  SearchContainer, // 搜索——搜索组件
  ContactListContainer, // 通讯录——通讯录导航组件
  ContactInfoContainer, // 通讯录——通讯录详情组件，包含好友列表、群组列表以及黑名单列表
  MyAvatarContainer, // 用户资料组件
} from '@xkit-yx/im-kit-ui'
import { CallViewProvider, GroupCallViewProvider } from "@xkit-yx/call-kit-react-ui";
import Button from 'antd/lib/button'
import MobileFilled from '@ant-design/icons/MobileFilled';
import SmileFilled from '@ant-design/icons/SmileFilled';
import LoginOutlined from '@ant-design/icons/LoginOutlined';
import AudioOutlined from '@ant-design/icons/AudioOutlined';
import VideoCameraOutlined from '@ant-design/icons/VideoCameraOutlined';
import { NimKitCoreFactory } from '@xkit-yx/core-kit'
import { compile } from 'jsx-web-compiler'
import React from 'react'
// 引入样式
import '@xkit-yx/im-kit-ui/es/style'
import '@xkit-yx/call-kit-react-ui/es/style'


export default {
  name: 'App',
  components: {
    MobileFilled: applyReactInVue(MobileFilled),
    SmileFilled: applyReactInVue(SmileFilled),
    LoginOutlined: applyReactInVue(LoginOutlined),
    CallViewProvider: applyReactInVue(CallViewProvider),
    GroupCallViewProvider: applyReactInVue(GroupCallViewProvider),
    Provider: applyReactInVue(Provider),
    ConversationContainer: applyReactInVue(ConversationContainer),
    ChatContainer: applyReactInVue(ChatContainer),
    AddContainer: applyReactInVue(AddContainer),
    SearchContainer: applyReactInVue(SearchContainer),
    ContactListContainer: applyReactInVue(ContactListContainer),
    ContactInfoContainer: applyReactInVue(ContactInfoContainer),
    MyAvatarContainer: applyReactInVue(MyAvatarContainer),
  },
  data: function () {
    return {
      // imuikit配置
      initOptions: {
        appkey: '', //传入您的App Key
        account: '', // 传入您的云信IM账号
        token: '', // 传入您的Token
        debugLevel: 'debug',
      },
      // callkit配置
      neCallConfig: {
        nim: null,
        appKey: '',//传入您的App Key
        currentUserInfo: { accId: '' }, // 传入您的云信IM账号
        debug: true
      },
      chatInputActions: [
        {
          action: 'audio',
          visible: true,
          render: (data) => {
            const call = () => {
              this.$refs.callViewProviderRef.reactRef.call(
                {
                  accId: data.to,
                  callType: '1',
                }
              )
            }
            // 群呼功能不区分语音跟视频，所以隐藏语音按钮
            if (data.scene === 'team') {
              return null
            }
            // 方案1
            const icon = React.createElement(AudioOutlined, { style: { fontSize: '20px', color: '#656a72' } })
            return React.createElement(Button, { type: 'default', onClick: call }, icon)
          }
        },
        {
          action: 'video',
          visible: true,
          render: (data) => {
            const call = () => {
              if (data.scene === 'team') {
                // demo中全量拨打，实际使用中，可以根据业务需求，写个弹窗，在弹窗中勾选对应呼叫的成员后，再调用groupCall方法
                this.nimKitCore.getTeamMembers({ teamId: data.to }).then((res) => {
                  // 通过接口获取群成员列表，并过滤自己
                  const calleeList = res.map((item) => item.account).filter((item) => item !== this.neCallConfig.currentUserInfo.accId)
                  this.$refs.groupCallViewProviderRef.reactRef.groupCall({ calleeList })
                })
              } else if (data.scene === 'p2p') {
                this.$refs.callViewProviderRef.reactRef.call(
                  {
                    accId: data.to,
                    callType: '2',
                  }
                )
              }
            }
            // 方案2
            return compile(`(
              (Button, VideoCameraOutlined, call) => {
              return <Button type="default" onClick={call}><VideoCameraOutlined style={{ fontSize: '20px', color: '#656a72' }} /></Button>
            }
            )(...arguments)`, null, Button, VideoCameraOutlined, call)
            // const icon = React.createElement(VideoCameraOutlined, { style: { fontSize: '20px', color: '#656a72' } })
            // return React.createElement(Button, { type: 'default', onClick: call }, icon)
          }
        },
        {
          action: 'emoji',
          visible: true,
        },
        {
          action: 'sendImg',
          visible: true,
        },
        {
          action: 'sendFile',
          visible: true,
        }
      ],
      nimKitCore: null,
      model: 'chat'
    }
  },
  methods: {

  },
  mounted: function () {
    const NIM = NimKitCoreFactory(1)
    const nimKitCore = new NIM({
      initOptions: {
        ...this.initOptions,
        // 群呼在这接收邀请
        oncustomsysmsg: (msg) => {
          if (msg.type === 'custom') {
            this.$refs.groupCallViewProviderRef.reactRef.onMsg(msg)
          }
        },
      }
    })
    this.nimKitCore = nimKitCore
    nimKitCore.on('logined', () => {
      this.neCallConfig.nim = nimKitCore.nim
    })
  }
};
</script>

<style module>
.container {
  width: 900px;
  height: 600px;
  box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.header {
  width: 100%;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-bottom: 1px solid #e8e8e8;
}

.search {
  width: 50%;
}

.add {
  margin-left: 20px;
}

.content {
  width: 100%;
  height: 540px;
  display: flex;

}

.left {
  width: 60px;
  border-right: 1px solid #e8e8e8;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.avatar-icon {
  margin: 20px 0 0 0;
  border-radius: 50%;
  border: 1px solid #e8e8e8;
}

.chat-icon {
  margin: 20px 0 0 0;
  font-size: 22px;
  /* color: #1890ff; */
  color: rgba(0, 0, 0, 0.6);
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;

}

.contact-icon {
  margin: 20px 0 0 0;
  font-size: 22px;
  color: rgba(0, 0, 0, 0.6);
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
}

.active {
  color: #1890ff;
}

.logout-icon {
  position: absolute;
  bottom: 10px;
  font-size: 22px;
  color: rgba(0, 0, 0, 0.6);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.icon-label {
  font-size: 12px;
  text-align: center;
}

.right {
  flex: 1;
  display: flex;
}

.right-list {
  width: 200px;
}

.right-content {
  flex: 1;
}
</style>
