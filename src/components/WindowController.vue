<template>
  <div></div>
</template>

<script>
import { MessageType, WinOpt } from "../tools/enums";
import { ipcRenderer as ipc, webFrame } from "electron";

export default {
  name: "WindowController",
  data: function() {
    return {
      routeName: undefined,
      animationId: undefined
    };
  },
  methods: {
    windowOpt(type, args = null) {
      ipc.send(MessageType.WindowOpt.toString(), {
        type: type,
        args: args
      });
    },
    changeMode(routerName) {
      this.$controller.win.routeTo(routerName);
    },
    changeModeNoSave(routerName) {
      this.$router.push({ name: routerName });
    },
    minify(event) {
      this.windowOpt(WinOpt.Minify);
    },
    startDrag(event) {
      if (event.button === 0) {
        requestAnimationFrame(this.dragging);
        this.windowOpt(WinOpt.StartDrag);
      }
    },
    endDrag(e) {
      if (this.animationId) {
        cancelAnimationFrame(this.animationId);
        this.animationId = undefined;
      } else return;
    },
    dragging() {
      this.windowOpt(WinOpt.Dragging);
      this.animationId = requestAnimationFrame(this.dragging);
    },
    close(event) {},
    openMenu(id = null) {
      this.$controller.action.popup(id);
    },
    resize(w = null, h = null, x = null, y = null) {
      this.windowOpt(WinOpt.Resize, {
        h: h,
        w: w,
        x: x,
        y: y
      });
    },
    storeWindow() {
      if (this.routeName) {
        this.$controller.saveWindow(
          this.routeName,
          this.$controller.win.getBound(),
          this.size
        );
      }
    }
  },
  mounted: function() {
    ipc.on(MessageType.Router.toString(), (event, arg) => {
      this.changeModeNoSave(arg);
    });
    ipc.on(MessageType.WindowOpt.toString(), (event, arg) => {
      if (arg.type == WinOpt.SaveMode) {
        this.storeWindow();
      }
    });
    if (this.routeName) this.$controller.restoreWindow(this.routeName);
  },
  destroyed: function() {
    this.storeWindow();
  }
};
</script>
