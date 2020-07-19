<template>
  <canvas ref="canvas" :width="width" :height="height"></canvas>
</template>

<script>
const getColor = () => {
  const colorElements = "0,1,2,3,4,5,6,7,8,9,a,b,c,d,e,f";
  const colorArray = colorElements.split(",");
  let color = "#";
  for (var i = 0; i < 6; i++) {
    color += colorArray[Math.floor(Math.random() * 16)];
  }
  return color;
};
function Line(
  ctx,
  start,
  end,
  { y = 20, lineWidth = 2, lineHeight = 10, color = "#000000" }
) {
  this.ctx = ctx;
  this.start = start;
  this.y = y;
  this.end = end;
  this.lineWidth = lineWidth;
  this.lineHeight = lineHeight;
  this.color = color;
}
Line.prototype.draw = function () {
  this.ctx.save();
  this.ctx.lineWidth = this.lineWidth;
  this.ctx.lineJoin = "miter";
  this.ctx.strokeStyle = this.color;
  this.ctx.beginPath();
  this.ctx.moveTo(this.start, this.y - this.lineHeight);
  this.ctx.lineTo(this.start, this.y);
  this.ctx.lineTo(this.end, this.y);
  this.ctx.lineTo(this.end, this.y - this.lineHeight);
  this.ctx.stroke();
  this.ctx.restore();
};
export default {
  data() {
    return {
      lineHeight: 10,
      linewidth: 2,
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
    lineOffset() {
      return this.lineHeight - this.linewidth;
    },
    _sourceData() {
      const sourceData = this.sourceData
        .map((data) => ({
          start: new Date(data.start).getTime(),
          end: new Date(data.end).getTime(),
          text: data.text,
        }))
        .sort((a, b) => a.start - b.start);
      const offset = 24 * 60 * 60 * 1000 * 5;
      const length = sourceData.length;
      const start = sourceData[0].start - offset;
      const end = sourceData[length - 1].end + offset;
      const max = end - start;
      const cell = this.width / max;

      return sourceData.map((item) => ({
        start: (item.start - start) * cell,
        end: (item.end - start) * cell,
        text: item.text,
      }));
    },
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext("2d");
    this.renderUI();
    this.data = { ...this.data, ...this.props };
  },
  methods: {
    renderUI() {
      // const canvas = document.createElement('canvas')
      // canvas.getContext()
      console.log(this._sourceData);

      // this.drawLine(0, this.width);
      const base = new Line(this.ctx, 10, this.width - 10, {
        lineHeight: 7,
      });
      base.draw();
      let lineArr = [];
      this._sourceData.forEach((element) => {
        lineArr.push(
          new Line(this.ctx, element.start, element.end, {
            lineHeight: 10,
            color: getColor(),
          })
        );
      });
      lineArr.forEach((element) => {
        element.draw();
      });
      // this._sourceData.forEach((item) => {
      //   this.drawTimeLine(item.start, item.end, item.text);
      // });
    },
    drawTimeLine(start, end) {
      this.drawLine(start, end);
      // this.drawText(start, end, text);
    },
    drawLine(start, end) {
      this.ctx.fillStyle = getColor();
      this.ctx.fillRect(
        start,
        this.height - this.lineHeight,
        end - start,
        this.lineHeight
      );
      this.ctx.fillStyle = "white";
      this.ctx.fillRect(
        start + this.linewidth,
        this.height - this.lineHeight,
        end - start - 2 * this.linewidth,
        this.lineOffset
      );
    },
    drawText(start, end, text) {
      this.ctx.font = "14px sans-serif";
      this.ctx.textAlign = "center";

      const textArr = text.split("");
      let lineArr = [];
      let line = "";
      for (let i = 0; i < textArr.length; i++) {
        let tempLine = line + textArr[i];
        const measure = this.ctx.measureText(tempLine);
        if (textArr[i] === "\n") {
          lineArr.push(line);
          line = "";
        } else if (i === textArr.length - 1) {
          lineArr.push(tempLine);
        } else if (measure.width > end - start) {
          lineArr.push(line);
          line = textArr[i];
        } else {
          line = tempLine;
        }
      }
      this.ctx.fillStyle = "black";
      for (let i = 0; i < lineArr.length; i++) {
        this.ctx.fillText(
          lineArr[i],
          (start + end) / 2,
          this.height - (lineArr.length - i) * 16,
          end - start
        );
      }
    },
  },
};
</script>

<style></style>
