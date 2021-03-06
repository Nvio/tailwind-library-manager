<template>
  <div class="relative pr-4 bg-gray-500">
    <iframe
      ref="preview"
      @load="handleLoadedPreview"
      :srcdoc="html"
      class="w-full h-auto bg-white pointer-events-none"
      :class="{
        'pointer-events-none': isResizing,
        'pointer-events-auto': !isResizing,
      }"
      :style="{ width, height }"
    />

    <ResizeHandle
      @resize="resize"
      @end="endResize"
      :style="{ right: `${-change}px` }"
    />
  </div>
</template>

<script>
import ResizeHandle from "@/components/ResizeHandle";

export default {
  props: ["code"],
  data: () => ({
    widthChange: 0,
    startingWidthChange: 0,
    height: "150px",
    isResizing: false,
  }),
  computed: {
    width() {
      return `calc(100% + ${this.change}px)`;
    },
    html() {
      return `
      <!DOCTYPE html>
        <html lang="en">
          <head>
            <meta charset="utf-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width,initial-scale=1.0">
            <link href="http://localhost:9991/styles.css" rel="stylesheet">
          </head>
          <body style="padding-top: 1rem; padding-bottom: 1rem; padding-right: 0.5rem; padding-left: 0.5rem;">
            ${this.code}
          </body>
        </html>
      `;
    },
    change() {
      const change = this.startingWidthChange + this.widthChange;
      return change <= 0 ? change : 0;
    },
  },
  methods: {
    updateHeight() {
      this.height =
        this.$refs.preview.contentWindow.document.body.scrollHeight + "px";
    },
    resize(change) {
      this.widthChange = change;
      this.isResizing = true;
      this.updateHeight();
    },
    endResize() {
      this.startingWidthChange += this.widthChange;
      this.widthChange = 0;
      this.isResizing = false;
    },
    handleLoadedPreview() {
      this.updateHeight();
      // disable iframe links
      this.$refs.preview.contentWindow.document.body
        .querySelectorAll("a")
        .forEach((element) => {
          element.addEventListener("click", (e) => {
            e.preventDefault();
          });
        });
    },
  },
  components: { ResizeHandle },
};
</script>
