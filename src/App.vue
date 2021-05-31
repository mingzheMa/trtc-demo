<template>
  <div id="app">
    <label>
      userSig
      <input v-model="form.userSig" type="text" />
    </label>
    <br />
    <label>
      userID
      <input v-model="form.userId" type="text" />
    </label>
    <br />
    <label>
      roomID
      <input v-model="form.roomId" type="text" />
    </label>
    <br />
    <label>
      isAudio
      <input v-model="form.isAudio" type="checkbox" />
    </label>
    <br />
    <label>
      isVideo
      <input v-model="form.isVideo" type="checkbox" />
    </label>
    <br />
    <button @click="gogogo">GO!GO!GO!!!</button>

    <div class="stream-box">
      <div id="stream-box-self"></div>
      <div id="stream-box-user"></div>

      <div class="stream-box-btn" v-if="trtcLocalStream">
        <div class="stream-box-btn__close" @click="backbackback">
          <div />
          <div>挂断</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import TRTC from "trtc-js-sdk";
  export default {
    data() {
      return {
        form: {
          userId: "",
          roomId: "",
          userSig: "",
          isAudio: true,
          isVideo: true
        },

        trtcClient: null,
        trtcLocalStream: null,
        trtcRemoteStream: null
      };
    },
    methods: {
      async gogogo() {
        // 创建客户端
        this.trtcClient = TRTC.createClient({
          mode: "rtc",
          sdkAppId: "1400524884",
          userId: this.form.userId,
          userSig: this.form.userSig,
          useStringRoomId: true // 是否使用str为房间id
        });

        // 监听远端视频流增加
        this.trtcClient.on("stream-added", event => {
          console.log("远端流增加: " + event.stream.getId());
          //订阅远端流
          this.trtcClient.subscribe(event.stream);
        });

        // 监听远端视频流订阅
        this.trtcClient.on("stream-subscribed", event => {
          console.log("远端流订阅成功：" + event.stream.getId());
          // 播放远端流
          // event.stream.play("stream-box" + event.stream.getId());
          event.stream.play("stream-box-user");
          this.trtcRemoteStream = event.stream;
        });

        // 进入房间
        await this.trtcClient
          .join({ roomId: this.form.roomId })
          .catch(error => {
            console.error("进房失败 " + error);
          });
        console.log("进房成功");

        // 创建本地视频流
        this.trtcLocalStream = TRTC.createStream({
          userId: this.form.userId,
          audio: this.form.isAudio,
          video: this.form.isVideo
        });

        // 初始化本地视频流
        await this.trtcLocalStream.initialize().catch(error => {
          console.error("初始化本地流失败 " + error);
        });
        this.trtcLocalStream.play("stream-box-self");
        console.log("初始化本地流成功");

        // 发布本地视频流
        await this.trtcClient.publish(this.trtcLocalStream).catch(error => {
          console.error("本地流发布失败 " + error);
        });
        console.log("本地流发布成功");
      },

      async backbackback() {
        await this.trtcClient.leave().catch(error => {
          console.error("退房失败 " + error);
          // 错误不可恢复，需要刷新页面。
        });

        const videoTrack = this.trtcLocalStream.getVideoTrack();
        if (videoTrack) {
          await this.trtcLocalStream.removeTrack(videoTrack);
          videoTrack.stop();
          this.trtcLocalStream.close();
          this.trtcRemoteStream && this.trtcRemoteStream.close();
          this.trtcRemoteStream = null;
          this.trtcLocalStream = null;
        }
      }
    }
  };
</script>

<style scoped>
  .stream-box {
    position: relative;
    width: 375px;
    height: 812px;
    border: 1px solid #000;
  }

  #stream-box-user {
    position: absolute;
    width: 100%;
    height: 100%;
  }

  #stream-box-self {
    position: absolute;
    width: calc(375px / 4);
    height: calc(812px / 4);
    top: 0;
    right: 0;
  }

  .stream-box-btn {
    position: absolute;
    bottom: 4%;

    width: 100%;
    height: 140px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .stream-box-btn__close {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }

  .stream-box-btn__close > div:first-child {
    width: 80px;
    height: 80px;
    background: crimson;
    border-radius: 50%;
    cursor: pointer;
  }
</style>
