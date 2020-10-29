<template>
  <div id="VideoDiv">
    <video
      id="videoElement"
      width="1920"
      height="1200"
      autoplay
      muted
      tabindex="-1"
      @mouseenter="mouseEnter()"
      @mouseleave="mouseLeave()"
      @mousemove="mouseMove($event)"
      @mousedown="mouseDown($event)"
      @mouseup="mouseUp($event)"
      @keydown="keyboardDown($event)"
      @keypress="keyboardPress($event)"
      @keyup="keyboardUp($event)"
    ></video>
    <p>
      room id: <input id="roomid" type="text" value="movie" />
      <button @click="onClick()">start</button>
    </p>
  </div>
</template>

<script>
import flvjs from "flv.js";
import * as myWorker from "./myWorker.js";

const judgeMouseButton = (button) => {
  if (button == 0) return 10002;
  else if (button == 2) return 516;
  else return 519;
};

export default {
  name: "Video",
  data() {
    return {
      calculator: false,
      offsetTop: 0,
      offsetLeft: 0,
      scrollTop: 0,
      scrollLeft: 0,
      myWorker: null,
      mySocket: null,
      clientHeight: 0,
      clientWidth: 0,
    };
  },
  mounted() {
    // let videoElement = document.getElementById("videoElement");
    // videoElement.addEventListener("keypress", this.keyboardPress);
    // videoElement.addEventListener("keyup", this.keyboardUp);
    // videoElement.addEventListener("keydown", this.keyboardDown);
    // this.myWorker = this.$worker.create([
    //   {
    //     message: "onMouseMove",
    //     func: myWorker.onMouseMove,
    //   },
    //   {
    //     message: "onMouseDown",
    //     func: myWorker.onMouseDown,
    //   },
    //   {
    //     message: "onMouseUp",
    //     func: myWorker.onMouseUp,
    //   },
    // ]);
  },
  methods: {
    onClick() {
      let roomId = document.getElementById("roomid").value;
      if (flvjs.isSupported()) {
        console.log("support flvjs");
        let videoElement = document.getElementById("videoElement");
        let url = "http://192.168.252.39:8080/live/" + roomId + ".flv";
        let flvPlayer = flvjs.createPlayer(
          {
            type: "flv",
            isLive: true,
            hasVideo: true,
            hasAudio: false,
            url, //: "http://192.168.252.39:7001/live/movie.flv",
          },
          {
            enableWorker: true,
            enableStashBuffer: false,
            fixAudioTimestampGap: false,
          }
        );
        this.offsetLeft = videoElement.offsetLeft;
        this.offsetTop = videoElement.offsetTop;
        console.log(flvPlayer, "flv对象");
        flvPlayer.attachMediaElement(videoElement);
        flvPlayer.load();
        // flvPlayer.play();
        setTimeout(() => {
          flvPlayer.play();
        }, 2000);

        setInterval(() => {
          if (!videoElement.buffered.length) {
            return;
          }
          let end = videoElement.buffered.end(0);
          let diff = end - videoElement.currentTime;
          console.log(diff, "延时");
          if (diff >= 0.3) {
            videoElement.currentTime = end;
          }
        }, 3000);
      }
      let wsurl = "ws://192.168.252.39:11288/ws?roomid=" + roomId + "&uid=test";
      this.mySocket = myWorker.createSocket(wsurl);
    },
    mouseEnter() {
      this.calculator = true;
    },
    mouseLeave() {
      this.calculator = false;
    },
    mouseMove(e) {
      let videoElement = document.getElementById("videoElement");
      this.offsetLeft = videoElement.offsetLeft;
      this.offsetTop = videoElement.offsetTop;
      this.scrollTop = document.documentElement.scrollTop;
      this.scrollLeft = document.documentElement.scrollLeft;
      this.clientWidth = videoElement.clientWidth;
      this.clientHeight = videoElement.clientHeight;
      if (this.calculator)
        myWorker.sendWSPush({
          event: "mouse",
          msg: "worker run on mouse move",
          type: 10001,
          x: (e.clientX - this.offsetLeft + this.scrollLeft) / this.clientWidth,
          y: (e.clientY - this.offsetTop + this.scrollTop) / this.clientHeight,
        });
      // myWorker.sendWSPush([10001,(e.clientX - this.offsetLeft + this.scrollLeft)/1280,(e.clientY - this.offsetTop + this.scrollTop)/800])
      // this.myWorker.postMessage("onMouseMove", [
      //   {
      //     x: e.clientX - this.offsetLeft + this.scrollLeft,
      //     y: e.clientY - this.offsetTop + this.scrollTop,
      //   },
      // ]);
    },
    mouseDown(e) {
      let type = judgeMouseButton(e.button);
      myWorker.sendWSPush({
        event: "mouse",
        msg: "worker run on mouse down",
        type,
        x: (e.clientX - this.offsetLeft + this.scrollLeft) / this.clientWidth,
        y: (e.clientY - this.offsetTop + this.scrollTop) / this.clientHeight,
      });
      // this.myWorker.postMessage("onMouseDown", [
      //   {
      //     x: e.clientX - this.offsetLeft + this.scrollLeft,
      //     y: e.clientY - this.offsetTop + this.scrollTop,
      //   },
      // ]);
    },
    mouseUp(e) {
      let type = judgeMouseButton(e.button) + 1;
      myWorker.sendWSPush({
        event: "mouse",
        msg: "worker run on mouse up",
        type,
        x: (e.clientX - this.offsetLeft + this.scrollLeft) / this.clientWidth,
        y: (e.clientY - this.offsetTop + this.scrollTop) / this.clientHeight,
      });
      e.returnValue = false;
      e.returnvalue = false;
      // this.myWorker.postMessage("onMouseUp", [
      //   {
      //     x: e.clientX - this.offsetLeft + this.scrollLeft,
      //     y: e.clientY - this.offsetTop + this.scrollTop,
      //   },
      // ]);
    },
    keyboardDown(e) {
      console.log(e);
      // console.log(e.code);
      myWorker.sendWSPush({
        event: "keyboard",
        msg: "worker run on keyboard down",
        key: e.code,
        key_down: true,
      });
      e.returnValue = false;
      e.returnvalue = false;
    },
    keyboardPress(e) {
      console.log(e);
      e.returnvalue = false;
      e.returnValue = false;
    },
    keyboardUp(e) {
      console.log(e);
      // console.log(e.code);
      myWorker.sendWSPush({
        event: "keyboard",
        msg: "worker run on keyboard down",
        key: e.code,
        key_down: false,
      });
      e.returnvalue = false;
      e.returnValue = false;
    },
  },
};
</script>