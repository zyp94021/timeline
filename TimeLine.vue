<template>
  <canvas ref="canvas" :width="width" :height="height"></canvas>
</template>

<script>
import { throttle } from "lodash";
import moment from "moment";

// this.ctx.save();
// this.ctx.font = "14px sans-serif";
// this.ctx.textAlign = "center";

// const textArr = this.text.split("");
// let lineArr = [];
// let line = "";
// for (let i = 0; i < textArr.length; i++) {
//   let tempLine = line + textArr[i];
//   const measure = this.ctx.measureText(tempLine);
//   if (textArr[i] === "\n") {
//     lineArr.push(line);
//     line = "";
//   } else if (i === textArr.length - 1) {
//     lineArr.push(tempLine);
//   } else if (measure.width > this.width) {
//     lineArr.push(line);
//     line = textArr[i];
//   } else {
//     line = tempLine;
//   }
// }
// this.ctx.fillStyle = "black";
// for (let i = 0; i < lineArr.length; i++) {
//   this.ctx.fillText(
//     lineArr[i],
//     (this.x + this.width + this.x) / 2,
//     this.height - (lineArr.length - i) * 16,
//     end - start
//   );
// }
// this.ctx.restore();

function Line(
  ctx,
  x,
  y,
  width,
  height,
  {
    lineWidth = 2,
    color = "#d9d9d9",
    startTime = "",
    endTime = "",
    text = "",
  } = {}
) {
  this.ctx = ctx;
  this.x = x + 100;
  this.y = y;
  this.width = width;
  this.height = height;
  this.lineWidth = lineWidth;
  this.color = color;
  this.startTime = startTime;
  this.endTime = endTime;
  this.text = text;
}
Line.prototype.draw = function () {
  this.ctx.save();
  this.ctx.lineWidth = this.lineWidth;
  this.ctx.lineJoin = "miter";
  this.ctx.strokeStyle = this.color;
  this.ctx.beginPath();
  this.ctx.moveTo(this.x, this.y - this.height);
  this.ctx.lineTo(this.x, this.y);
  this.ctx.lineTo(this.x + this.width, this.y);
  this.ctx.lineTo(this.x + this.width, this.y - this.height);
  this.ctx.stroke();
  this.ctx.restore();
};
Line.prototype.drawSelected = function () {
  this.ctx.save();
  this.ctx.lineWidth = this.lineWidth;
  this.ctx.lineJoin = "miter";
  this.ctx.shadowBlur = 3;
  this.ctx.shadowColor = this.color;
  this.ctx.shadowOffsetY = 2;
  this.ctx.shadowOffsetX = 2;
  this.ctx.strokeStyle = this.color;
  this.ctx.beginPath();
  this.ctx.moveTo(this.x, this.y - this.height - 2);
  this.ctx.lineTo(this.x, this.y - 2);
  this.ctx.lineTo(this.x + this.width, this.y - 2);
  this.ctx.lineTo(this.x + this.width, this.y - this.height - 2);
  this.ctx.stroke();
  this.ctx.restore();
};
Line.prototype.checkCollision = function (x, y) {
  return (
    x >= this.x &&
    x <= this.x + this.width &&
    y >= this.y - this.height - 20 &&
    y <= this.y + 20
  );
};
Line.prototype.drawText = function () {
  this.ctx.save();
  this.ctx.font = "14px sans-serif";
  this.ctx.textAlign = "center";
  this.ctx.fillStyle = "black";
  const measure = this.ctx.measureText(this.text);
  if (measure.width <= this.width) {
    this.ctx.fillText(
      this.text,
      (this.x + this.x + this.width) / 2,
      this.y - 20
    );
  }
  this.ctx.restore();
};
Line.prototype.drawSelectedText = function () {
  this.ctx.save();
  this.ctx.font = "14px sans-serif";
  this.ctx.textAlign = "center";
  this.ctx.fillStyle = "black";
  this.ctx.fillText(
    `${this.startTime}-${this.endTime}`,
    (this.x + this.x + this.width) / 2,
    this.y - 40
  );
  this.ctx.fillText(this.text, (this.x + this.x + this.width) / 2, this.y - 20);
  this.ctx.restore();
};
Line.prototype.drawTime = function () {
  this.ctx.save();
  this.ctx.font = "14px sans-serif";
  this.ctx.textAlign = "center";
  this.ctx.fillStyle = "black";
  this.ctx.fillText(this.startTime, this.x, this.y + 20);
  this.ctx.fillText(this.endTime, this.x + this.width, this.y + 20);
  this.ctx.restore();
};
const formatTime = (time) => {
  return moment(time).format("YYYY-MM-DD HH:mm:ss");
};
export default {
  data() {
    return {
      lineHeight: 10,
      linewidth: 2,
      lineArr: [],
      selectedLine: -1,
      offset: 24 * 60 * 60 * 1000 * 5,
    };
  },
  props: ["options", "sourceData"],
  computed: {
    width() {
      return this.options.width;
    },
    height() {
      return this.options.height;
    },
    timeData() {
      return this.sourceData
        .map((data) => ({
          start: new Date(data.start).getTime(),
          end: new Date(data.end).getTime(),
          text: data.text,
        }))
        .sort((a, b) => a.start - b.start);
    },

    startTime() {
      return this.timeData[0].start - this.offset;
    },
    endTime() {
      const length = this.sourceData.length;
      return this.timeData[length - 1].end + this.offset;
    },
    _sourceData() {
      const max = this.endTime - this.startTime;
      const cell = (this.width - 200) / max;

      return this.timeData.map((item) => ({
        startTime: item.start,
        endTime: item.end,
        start: (item.start - this.startTime) * cell,
        end: (item.end - this.startTime) * cell,
        text: item.text,
      }));
    },
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext("2d");
    this.data = { ...this.data, ...this.props };
    this.initData();
    console.log(this._sourceData);
    this.$refs.canvas.addEventListener(
      "mousemove",
      throttle((e) => {
        this.selectedLine = -1;
        for (let i = 1; i < this.lineArr.length; i++) {
          const line = this.lineArr[i];
          if (line.checkCollision(e.offsetX, e.offsetY)) {
            this.selectedLine = i;
            break;
          }
        }
      }, 100)
    );
    this.renderUI();
  },
  methods: {
    initData() {
      const base = new Line(this.ctx, 10, this.height / 2, this.width - 220, 7, {
        startTime: formatTime(this.startTime),
        endTime: formatTime(this.endTime),
      });
      this.lineArr.push(base);
      this._sourceData.forEach((item) => {
        this.lineArr.push(
          new Line(
            this.ctx,
            item.start,
            this.height / 2,
            item.end - item.start,
            8,
            {
              color: "#1890ff",
              startTime: formatTime(item.startTime),
              endTime: formatTime(item.endTime),
              text: item.text,
            }
          )
        );
      });
    },
    renderUI() {
      // const canvas = document.createElement('canvas')
      // const ctx = canvas.getContext('2d')
      // ctx.clearRect()
      this.ctx.clearRect(0, 0, this.width, this.height);
      this.lineArr.forEach((line, index) => {
        if (index === 0) {
          line.drawTime();
        }
        if (index === this.selectedLine) {
          line.drawSelected();
          line.drawSelectedText();
        } else {
          line.draw();
          line.drawText();
        }
      });
      requestAnimationFrame(this.renderUI);
    },
  },
};
</script>

<style></style>
